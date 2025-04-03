# HW2 - Markdown Answers Below

## ML Modeling within Redshift

**Amazon Redshift ML**

Redshift has the ability to perform machine learning within a cluster. This eliminates the need to build custom pipelines to move data in and out of the cluster for processing in a data model. Redshift supports a variety of models, and you can even bring your own. You can learn more about Redshift's ML features on [its product page](https://aws.amazon.com/redshift/features/redshift-ml/).

**Prerequisites**

Before starting, consider the following prerequisites:

- Plan your time: the model training portion of the assignment may take up to one hour.
- Resources: This walkthrough will consume about $12.00 of your lab budget.
- Ensure you are always working in the `us-east-1` region.
- On the Amazon S3 console, create an S3 bucket that Redshift ML uses for uploading the training data that Forecast uses to train the model.
  - Bucket names need to be globally unique, so use your UW Net ID to help enforce uniqueness. For example, my bucket name is: `uw-kazzazmk-mlredshift`.
- You should be running a Redshift cluster similar to what you did during the hands-on portion in class. If you need a refresher, [check out our notes from class](https://gitlab.cs.washington.edu/mmkazzaz/csed516-2024au/-/blob/main/admin/redshift/redshift_cluster_parameters.md?ref_type=heads).
- Use the `LabRole` IAM role whenever an IAM role is required.
- Remember to only run your Redshift cluster when you are using it. 
  - Pause or terminate it when it's not in use. 
  - If not, you risk running out of lab credits for our class, and you will need to pay for your own AWS usage.

**Submission Requirements**
- Use a Redshift Notebook via the Query Editor V2 to paste, update, and run all of the below commands.
- Use a final markdown cell to answer the questions at the end of these instructions.
- Upload your executed Redshift Notebook to complete the assignment.

---
---

**Predicting customer churn via classification model**

For this assignment, we will perform classification prediction. We will use data to label whether or not customers have churned. Customer churn refers to the loss of customers over a period of time. It happens when customers stop using a company's product or service, and it is often used as a key measure to understand customer loyalty and retention.

We will use a version of the Iranian Churn data set from the UCI Machine Learning Repository. The original source is available via [UC Irvine's Machine Learning Repository](https://archive.ics.uci.edu/dataset/563/iranian+churn+dataset). For our purpose, we will use a modified copy of the data hosted by AWS:
- Via S3: `s3://redshift-downloads/redshift-ml/customer_activity/`

Let's get this table loaded into Redshift. Complete the below DDL statements to create and load the data.

```sql
CREATE SCHEMA DEMO_ML
;

CREATE TABLE demo_ml.customer_activity (
state varchar(2), 
account_length int, 
area_code int,
phone varchar(8), 
intl_plan varchar(3), 
vMail_plan varchar(3),
vMail_message int, 
day_mins float, 
day_calls int, 
day_charge float,
total_charge float,
eve_mins float, 
eve_calls int, 
eve_charge float, 
night_mins float,
night_calls int, 
night_charge float, 
intl_mins float, 
intl_calls int,
intl_charge float, 
cust_serv_calls int, 
churn varchar(6),
record_date date)
;

COPY DEMO_ML.customer_activity 
FROM 's3://redshift-downloads/redshift-ml/customer_activity/' 
IAM_ROLE '${IAM_ROLE}'
delimiter ',' 
IGNOREHEADER 1  
region 'us-east-1'
;
```

**Create Forecast Model**

For Redshift ML forecasting model, you need to ensure that when you issue a CREATE MODEL statement, you specify `MODEL_TYPE` as `FORECAST`. When Redshift ML trains a model or predictor on Amazon Forecast, it has a fixed forecast, meaning there is not a physical model to compile and execute.

**Heads up! The next command may take up to one hour to complete.**

```sql
CREATE MODEL demo_ml.customer_churn_model
FROM (SELECT state,
             area_code,
             total_charge/account_length AS average_daily_spend, 
             cust_serv_calls/account_length AS average_daily_cases,
             churn 
      FROM demo_ml.customer_activity
         WHERE record_date < '2020-01-01' 

     )
TARGET churn
FUNCTION predict_customer_churn
IAM_ROLE '${IAM_ROLE}'
SETTINGS (
  S3_BUCKET '${BUCKET_NAME}'
)
;
```

The SELECT query in the FROM clause specifies the training data. The TARGET clause specifies which column is the label that the CREATE MODEL builds a model to predict. The other columns in the training query are the features (input) used for the prediction. In this example, the training data provides features regarding state, area code, average daily spend, and average daily cases for the customers that have been active earlier than January 1, 2020. The target column churn indicates whether the customer still has an active membership or has suspended their membership. For more information about CREATE MODEL syntax, see the [Amazon Redshift Database Developer Guide](https://docs.aws.amazon.com/redshift/latest/dg/machine_learning-overview.html).

Creating a model in Redshift takes some time. Before we can move on, we will need to wait until our model is ready. Run the following query every few minutes until you see Model State transition from **TRAINING** to **READY**.

Redshift is working with SageMaker to run training, transform, and hypertuning jobs. You can track the progress by navigating to the SageMaker Dashboard: https://us-east-1.console.aws.amazon.com/sagemaker/home?region=us-east-1#/dashboard. The long pole of the process is SageMaker hypertuning the parameters. Once the Hypertuning starts, it will execute 100 training jobs. Once those jobs complete, our model will be ready for use.

For reference, when creating this assignment, it took just under an hour for my model to be ready.

```sql
SHOW MODEL demo_ml.customer_churn_model
;
```

**Reviewing our model**

Wth our model ready, we can now evaluate how well it performed.

```sql
WITH infer_data AS (
  SELECT area_code ||phone  accountid, churn,
    demo_ml.predict_customer_churn( 
          state,
          area_code, 
          total_charge/account_length , 
          cust_serv_calls/account_length ) AS predicted
  FROM demo_ml.customer_activity
WHERE record_date <  '2020-01-01'

)
SELECT * FROM infer_data
;
```

Let's look further at the instances where our model was wrong. This function will present the label probabilities alongside the predicted label.

```sql
WITH infer_data AS (
  SELECT area_code ||phone  accountid, churn,
    demo_ml.predict_customer_churn_prob( 
          state,
          area_code, 
          total_charge/account_length , 
          cust_serv_calls/account_length ) AS predicted
  FROM demo_ml.customer_activity
WHERE record_date <  '2020-01-01'

)
SELECT *  FROM infer_data where churn!=predicted
;
```

**Use our ML model for inference**

The value of an model comes from using it to inference results on non-training data. In our case, we trained our data on activity prior to 2020. Let's run our model against records starting in 2020 for area code 408.

```sql
SELECT area_code ||phone  accountid, 
       demo_ml.predict_customer_churn( 
          state,
          area_code, 
          total_charge/account_length , 
          cust_serv_calls/account_length )
          AS "predictedActive"
FROM demo_ml.customer_activity
WHERE area_code='408' and record_date > '2020-01-01'
;
```

---
---
## Notes

-  The model had 100 rows with churn = False (among the 3,000 records in the dataset): Or maybe 100 is the maximum number of rows that the query editor can display?
- All of the probabilities are over 0.50, meaning the model was sure it was likely correct
- 

## Homework Answers
Answers to the following questions:
- What does each of the parameters of the COPY statement we used above do?


- What is the difference between the `predict_customer_churn` and `predict_customer_churn_prob` functions used in the SQL code? When would you use each one?


- Explain the purpose of the two WITH clauses used in the SQL code to review the model's performance. What are these WITH clauses doing? Why are they useful?


- When would you want to use ML within Reshift? When would you not?


- Review the features available in the source data. Write an alternative CREATE MODEL statement with a different set of features. **You do not need to execute the statement.** Why did you select these features?



---
---
Remember to pause/terminate your instance. Upload your executed notebook to Canvas to gain credit for this assignment.