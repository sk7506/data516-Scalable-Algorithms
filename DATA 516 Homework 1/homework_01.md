# Homework 1: Data Validation and Transformation via Polars DataFrames

## Background
You work in Supply Chain for a hyperscale cloud provider. One of your organization's responsibilities is to procure server racks for data centers. Server racks have two major categories of components: server components and rack components. Server components make up each individual server (e.g., CPU, SSD, HDD, etc.). There can be multiple servers in a rack. Rack components hold all the servers in a chassis and provide a top-of-rack (TOR) networking switch for data center connectivity. This photo from [Nlyte Software](https://www.nlyte.com/blog/data-center-basics/) visualizes server and rack components:

![alt text](Server-and-Rack.png)

## Situation
We manage several different server rack programs for various compute products. Each program is quoted by a variety of vendors. The quoting process is challenging. Initially, vendors submitted summary-level quotes that provided the cost of an entire server rack. As your organization evolved, you asked vendors to start providing more detailed quotes where each line represents the cost of a individual component in the server rack. These quote line details do not include server or rack quantities. Unfortunately, internal systems are still tied to the summary-level quote submissions. The current state requires vendors to provide quotes as both summaries and detailed line items. 

Vendors have built automation to continuously supply quotes, as market rates for various components can fluctuate daily. There is little to no validation performed on quotes when they enter our system. As such, we need to perform the following validations on our quote data:

- Quotes for a given month can only be accepted starting on the first Monday of the month and ending on the 25th. These boundary dates are inclusive. Any quotes provided outside of these dates will not be considered.
- The sum of the quote line details must match the provided quote summary total. If the two datasets do not agree, the quotes will not be considered.
- We do not purchase Program C and Program E quotes from Vendor 7. Vendor 7’s systems are configured to quote all available programs, so these quotes need to be discarded.

## Available data
- Server component quantites (Excel)
- Rack component quantities (Excel)
- Vendor quote summary data (csv)
- Vendor quote line detail data (csv)

## Assignment
As part of the data team, you need to transform various datasets to create the following:

- A table detailing the total server rack quantity for each component across all programs. Each record in the table should represent a distinct component. Each attribute of the table should represent a distinct program. The intersection of record and attribute should represent the total extended quantity of parts for that component and program combination.
- You must join the relevant datasets and apply the necessary calculations, transformations, and filters to create a final dataset that complies with all of our data validation requirements.
- Once you have a dataset that meets all the requirements, create tables for each of the following scenarios:
  - If we only consider the latest quote received by each vendor for each program, what is the total server rack cost per program per vendor?
  - If we only consider the first quote received by each vendor for each program, what is the total cost per program per vendor?
  - To determine "best-in-class" pricing, calculate the total server rack cost by determining the lowest price per component and summing the total, regardless of vendor. What is the "best-in-class" pricing for each program?

## Submission Requirements
Submit to Canvas a populated Jupyter notebook where the last four executed cells contain:
 1. The table of extended quantites per component for all programs.
 2. The table of total cost per program per vendor, based on the latest received quote.
 3. The table of total cost per program per vendor, based on the first received quote.
 4. The table of "best-in-class" total cost per program (regardless of vendor).

These four tables must be the final outputs of your notebook. If not, we will not grade your assignment beyond the attempt. Our TAs will not execute your code. The notebook you upload to Canvas must have been executed, with all tables shown as output.
