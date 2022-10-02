# Data-and-AI-at-Scale
Some experience, thoughts, code snippets related to data and AI at scale 

# Principles: To avoid scalability issues by Archtecting

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


# Experience and Thoughts on Data Pipeline Development in Airflow

I have led a team to build 20+ ETL/ELT data pipelines in Airflow. As a practitioner, I am to share my experience and thoughts.

## How to build a robust ETL/ELT data pipeline in Airflow

Most challenges come from the data pipeline part instead of the ETL/ELT part. While the ETL/ELT part could vary case by case, the data pipeline part can be more general and critical to make ETL/ELT idempotent and robust.

The general challenges
ETL/ELT work could be very heavy. 
Data could be late or not available as a whole batch.
Data could be dirty.
Schema could change batch by batch.
The schedule is not regular.

### Some important Airflow features we will use in our design
#### Deferrable Operators/Sensors (Airflow 2.2+, Python 3.7+)
These (deferrable) operators leverage Python’s asyncio library to efficiently run tasks waiting for an external resource to finish. This frees up your workers, allowing you to utilize those resources more effectively. (Digested from a good summary at https://www.astronomer.io/guides/deferrable-operators/) .
Deferrable operators and sensors could improve pipeline efficiency greatly. So when we design, we don’t need to pay much attention to performance.
#### Timetable (Airflow 2.2+)
Timetables allow users to create their own custom schedules using Python, effectively eliminating the limitations of cron. With timetables, you can now schedule DAGs to run at any time for any use case. (Digested from a good summary at https://www.astronomer.io/guides/scheduling-in-airflow/) .
#### LatestOnlyOperator (Airflow 2+) (https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/operators/latest_only/index.html?highlight=latest_only+operator#airflow.operators.latest_only.LatestOnlyOperator )
This operator is very important but not well-known. It allows a workflow to skip tasks that are not running during the most recent schedule interval. In situations where you care about the latest batch only, it is very useful to make sure to skip some tasks (such as publishing) of other batches.

### A general data pipeline process
#### Deferrable Sensor to detect the existence of Data. 
If data is late, without existence checking, the following job, for example, a Spark job could fail with a technical issue. It is not convenient for data operation. By using a deferrable sensor, the following jobs only run until the data is available.

#### Offload ETL/ELT task. 
Although some simple ETL/ELT can be run on Airflow server, it is better to offload them to another server. For example, the Apache Spark job should be run on a Spark cluster. BTW, you should use deferrable operators if possible.
Output Data Quality Check. The generated batch output could be of low quality. Without a data quality check, dirty data could be published.

#### Latest only data release. 
Most likely, data pipelines are not for one-off. So how deal with different batches of data is very critical. In most situations, only the latest data is really needed for publishing. Please note, in some situations, the different batches of data should be union-ed together instead of replaced by the latest version.

#### Keep the history. Usually, the generated batch data should be kept. In most situations, a batch of data can be treated as a partition of a table. But for some schema change situations, it is not good to use partitions. A general way is to use tables with a wildcard, such as table_YYYYMMDD. 

### Schedule
After we discussed a general data pipeline process, then let us focus on schedule, which is critical to non-one-off pipelines.
At first, we may use a new feature of Airflow, timetable, to deal with some irregular schedules. For example, some data is only available for stock market trading days.
Then, most likely we need to assign a time part. For example, when data is supposed to come between 8 pm and 10 pm, which time should be used to schedule the data pipeline? Because we use deferrable sensors to check the existence of data, so we can use 8 pm instead of 10 pm. So that data will be processed as soon as possible with a little checking overhead.

In conclusion, ETL/ELT data pipelines are not only ETL/ELT but also data pipelines. A good design could mitigate risks and make the data pipeline more robust. Thereafter the following data operation will be easy.


## Use Jupyter as a common tool for both data analysts and data engineers

Jupyter Notebook (Jupyter in short) is a very popular tool for data analysts and data scientists. But usually, data engineers use IDE to develop. Without a common tool, there is not a common code base, thereafter it is hard for them to work together. But actually, Jupyter can be a common tool for both of them.

Jupyter provides an interactive way to check data and run code, which is very friendly to data analysts and data scientists. But so far, Jupyter is not well supported by Github. So if Jupyter cannot be well versioned and so that the code management is not effective, which is a basic requirement of data engineers.

### Basic Usage: To use Jupyter along with a Python code file
In a data team, usually, data analysts cannot share Python code easily with data engineers for they are using different file formats. Data analysts usually use .ipynb file, technically a Json file, while data engineers usually use .py file, technically a plain text file.

Luckily, there is a well-known Jupyter extension called Jupytext (https://github.com/mwouts/jupytext). Basically, Jupytext will save a corresponding .py file along with the original .ipynb file. If one of .py and .ipynb is changed, another one will be changed automatically.
 
So that data engineers can work with data analysts together by using Jupyter, even in their favorite IDEs. And the .py files can be easily managed by Github.

### Further Usage: To use Jupyeter with conditional coding

Data Analysts are always doing some data exploring in Jupyter. When sharing code with data engineers, the data exploring part may not be needed for production. 
Technically .ipynb file is a Json file. Tags on cells are metadata of a cell. If we can tag each cell clearly, we can use simple code to digest the needed cells.



In conclusion, by using Jupyter as a common tool for both data analysts and data engineers, they can work together on the same code base using their favorite environments. 


## Teamwork in Data Pipeline Development

Data pipelines use a few technologies and a few of person with different backgrounds may be involved. How to make everyone work efficiently is critical. Here only technologies-related teamwork topics will be discussed.

### Align technologies.
Different people may have their favorite technologies. In order to improve the whole efficiency of the team, it is better to align technologies. For example, if Apache Airflow is used as the orchestration platform, Python should be the primary language for development for Python is the native language for Airflow.
Versions of different technologies should be paid attention to. For Python, 3.6 may be not supported by some technologies and some new features may need Python 3.7 or 3.8, but not 3.9. Deciding a version will mitigate the version conflicts.

Just as mentioned above, Jupyter can be used as a common tool for both data analysts and data engineers.

### Split data pipeline development.
It is a natural idea to assign each data pipeline to a person. But if we can split development into more parts, it will be more convenient for project management.
Besides the dimension of data pipelines, we can split development into at least 2 parts, ETL/ELT part and DAG part, just as mentioned above.

So we have at least 2 dimensions to split, one is by data source, while another is by tasks in DAG.

### Know different skillsets
Different people have different skill sets and different preferences. Some may be good at ETL/ELT, while others may good at general DAG design.

### Adapt schedule to changes
Data pipelines have some dependencies, such as live feeding of data. If the environment changes, we may need to change our schedule accordingly.


In a conclusion, teamwork is very important to data pipeline development, we need technologies along with management to make teamwork efficient.


As a partitioner, I wish my experience and thoughts can bring you some real hints on solving practical issues in data pipeline development in Airflow.

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
