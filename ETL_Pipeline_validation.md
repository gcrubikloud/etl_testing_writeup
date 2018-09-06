# ETL Testing Interview Question

[copied from Rubikloud screening test](https://github.com/grahamcrowell/rubikloud)

Explain your thoughts on how you would ensure that ETL Pipelines are tested in an automated fashion, please identify all the steps/prerequisites required along with the assumptions you make. Recommend tools / procedures, assume we do all testing manually. How do we get a to a number which shows our test coverage is measurable and improving over time.

# Discussion

Assume we are talking about a batch style ETL pipeline. It's reads in input dataset as a snapshot of the source data and generates an output dataset in some data model/format optimized for client queries.
The pipeline's output is entirely determined by the
- state of source data at extract time and
- state of ETL code being executed.
 
None, one or both of these will change between a "run" of the ETL pipeline which could cause a regression.

## Strategy: Test these two sources of change separately

> When debugging the first step is usually to determine if the root cause is source data or ETL logic.

### Source Data Validation

ETL pipeline should generate and log a report summarizing pass/fail of following unit tests. Each unit test validates the shape/profile of the source data. Each unit test includes a "severity" setting (breaks load or hurts load) that determines if load is allowed to continue or should fail.

For each file/table define a unit tests that check 
- Column count
- Delimiter (if file)

For each source column define unit tests that check
- Column name
- Data type 
- Nullness
- Uniqueness

For each lookup and 1..m relationship validate referential integrity.
- Every foreign key value exists as primary key value

Other domain specific checks are also possible
- promotion date < sale date
- sale date < shipment date
- every order needs >= 1 item

> You can quantify coverage by computing the proportion of source tables and columns that have these unit tests defined.

### ETL Pipeline Testing

#### Unit Testing

##### The Test

ETL pipelines perform tranformations from the source's schema/model to the expected target schema/model.
- create tool to generate valid baseline mock raw source data (similar TPC-H) (**very small dataset** - an ETL run must be fast when ran locally in dev environment)
    - in the context of ETL unit testing each "data set" is a "test" (ie. "test data set")
    - persist the data of each tests in some globally accessible (blob) data store
        - deploy data store as stand alone micro service
        - document each data set automatically by enforcing convention that url/path of the micro service end point (eg. www.rubikloud.com/testdata/DP-123 where DP-123 is a Jira issue)
    - each test data set must include both mock source data and the expected output data

##### The Unit

For each transformation:
- manually edit mocked data so that it represents the shape/characteristics of transformation's input
    - in the context of ETL unit testing a "transformation" is a "unit"

ETL pipelines make assumptions about data they ingest.  But as more and more *raw real world* source data is ingested *data idiosyncracies* are discovered that violate these assumptions.

For each real world data idiosyncracy:
- document in Jira as a task or story (or whatever)
- manually/magically create mock data sets that represent a specific/isolated idiosyncracy found in real world raw source data
    - in the context of ETL unit testing "transformation" is a "unit"

##### ETL Unit Test Types  

|unit|test|
|:--|:---|
|transformation|baseline data set|
|real world data idiosyncracy|modified baseline data set|

> TODO: Find/develop a DataFrame comparison library ([option: spark-fast-tests](https://github.com/MrPowers/spark-fast-tests))

> Principle: Design transformations so they *do one thing*.

#### Regression Testing

Extend analytical engine so that it can be use for detecting regressions between loads.
Define metrics that quantify data quality. For example, diff metric values based current dataset with previous output.  This allows you to define acceptable thresholds (eg "predicted total monthly sales shouldn't charges by more than 20% between loads"). 
 
> An automation tool like Jenkins can be used automate the execution of these metrics.