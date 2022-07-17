# Data-and-AI-at-Scale
Some experience, thoughts, code snippets related to data and AI at scale 

# Principles: To avoid scalability by Archtecting

## Infrastructure
### Severless First
Serverless service hides the scalability.

### Managed First
Managed service hides the detail of scalability.

### Cloud First
Cloud service reduce the difficulty of scalability.

## Application
### Built-in First
### Native First


# Situations: To provide scalability by Creativity 

## Situation 1: Different data vendors have different data license requirements. How to link them together and following these requirements?

## Situation 2: Vendors provide data at a daily frequence, how to keep the history but also to keep the management cost low?
Table with Partition or Wildcard tables?

## Situation 3: Vendors provide all kinds of data. Should we use ETL or ELT? Or EtLT (try to transform after loading as much as possible)
In some situations (parquet files with unsual column names), we even use ELT at first, then ETL when the former fails.

## Situation 4: A vendor called R provides data as many time series of data, how to pivot it and unpivot it?

## Situation 5: A vendor called B provides 200+ files as a batch. How to link them together?

## Situation 6: A company has thousands of staff, how to create contextual views for each one at run time?

## Situation 7: Thousands of data columns have been ingested as string, how to decide a most suitable column type?

## Situation 8: A vendor called C provides an unbound pivot, which exceeds the BigQuery's column number limit (10k).

## Situation 9: A BigQuery loading function ignores hundred of rows of CSV for some special characters within them while Park can recognize it in a partial correct way.

## Situation 10: How to operate data pipelines automatically and manually?

## Situation 11: How to make sure data pipeline runs when the whole batch of files are ready?



