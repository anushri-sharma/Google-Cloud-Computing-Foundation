GSP323

# [Prepare Data for ML APIs on Google Cloud: Challenge Lab](https://www.skills.google/paths/36/course_templates/631/labs/594536)

# Overview
In a challenge lab you’re given a scenario and a set of tasks. Instead of following step-by-step instructions, you will use the skills learned from the labs in the course to figure out how to complete the tasks on your own! An automated scoring system (shown on this page) will provide feedback on whether you have completed your tasks correctly.

When you take a challenge lab, you will not be taught new Google Cloud concepts. You are expected to extend your learned skills, like changing default values and reading and researching error messages to fix your own mistakes.

To score 100% you must successfully complete all tasks within the time period!

This lab is recommended for students who have enrolled in the Prepare Data for ML APIs on Google Cloud skill badge. Are you ready for the challenge?

## In this lab, you learn how to:

- Create a simple Managed Apache Spark job.
- Create a simple DataFlow job.
- Perform two Google machine learning backed API tasks.

## Check project permissions
Before you begin your work on Google Cloud, you need to ensure that your project has the correct permissions within Identity and Access Management (IAM).

> 1. In the Google Cloud console, on the **Navigation menu , select IAM & Admin > IAM.**

> 2. Confirm that the default compute Service Account **{project-number}-compute@developer.gserviceaccount.com** is present and has the editor and storage.admin role assigned. The account prefix is the project number, which you can find on **Navigation menu > Cloud Overview > Dashboard**.

>> Note: If the account is not present in IAM or does not have the storage.admin role, follow the steps below to assign the required role.

> 1. In the Google Cloud console, on the **Navigation menu, click Cloud Overview > Dashboard**.
> 2. Copy the **project number (e.g. 729328892908)**.
> 3. On the Navigation menu, select **IAM & Admin > IAM.**
> 4. At the top of the roles table, below **View by Principals, click Grant Access**.
> 5. For New principals, type:

```
{project-number}-compute@developer.gserviceaccount.com
```

> 6. Replace **{project-number}** with your project number.

> 7. For **Role, select Storage Admin**.

> 8. Click **Save**.

# Challenge scenario
As a junior data engineer in Jooli Inc. and recently trained with Google Cloud and a number of data services you have been asked to demonstrate your newly learned skills. The team has asked you to complete the following tasks.

You are expected to have the skills and knowledge for these tasks so don’t expect step-by-step guides.

# Task 1. Run a simple Dataflow job
In this task, you use the Dataflow batch template Text Files on Cloud Storage to BigQuery under "Process Data in Bulk (batch)" to transfer data from a Cloud Storage bucket (gs://spls/gsp323/lab.csv). The following table has the values you need to correctly configure the Dataflow job.

You will need to make sure to:

> Create a BigQuery dataset called `lab_195`(BigQuery Dataset Name) with a table called `customers_437`(Output Table Name).

> Create a Cloud Storage Bucket called `qwiklabs-gcp-01-063ae53f3d10-marking`(Cloud Storage Bucket Name).

> Expand the hint below for defining the BigQuery table schema

Edit as text:

```
[ {"type":"STRING","name":"guid"},
{"type":"BOOLEAN","name":"isActive"},
{"type":"STRING","name":"firstname"},
{"type":"STRING","name":"surname"},
{"type":"STRING","name":"company"},
{"type":"STRING","name":"email"},
{"type":"STRING","name":"phone"},
{"type":"STRING","name":"address"},
{"type":"STRING","name":"about"},
{"type":"TIMESTAMP","name":"registered"},
{"type":"FLOAT","name":"latitude"},
{"type":"FLOAT","name":"longitude"} ]
```

| Field                                                      | Value                                                                |
| ---------------------------------------------------------- | -------------------------------------------------------------------- |
| Region endpoint for the job                                | **Region**                                                           |
| Cloud Storage input file(s)                                | `gs://spls/gsp323/lab.csv`                                           |
| Cloud Storage location of your BigQuery schema file        | `gs://spls/gsp323/lab.schema`                                        |
| BigQuery output table                                      | **Output Table Name**                                                |
| Temporary directory for BigQuery loading process           | **Temporary BigQuery Directory**                                     |
| Temporary location                                         | **Temporary Location**                                               |
| Optional Parameters > JavaScript UDF path in Cloud Storage | `gs://spls/gsp323/lab.js`                                            |
| Optional Parameters > JavaScript UDF name                  | `transform`                                                          |
| Optional Parameters > Machine Type                         | Deselect **Use default machine type**, then select **e2-standard-2** |


Wait for the job to finish before trying to check your progress.

# Task 2. Run a simple Managed Apache Spark job
In this task, you run an example Spark job using Managed Apache Spark.

>> Note: Before you run your job, log into one of the cluster nodes and copy the `data.txt` file into hdfs using the command `hdfs dfs -cp gs://spls/gsp323/data.txt /data.txt`.

Run a Managed Apache Spark job using the values below:

| Field                 | Value                                                                              |
| --------------------- | ---------------------------------------------------------------------------------- |
| Job type              | **Spark**                                                                          |
| Main class or jar     | `org.apache.spark.examples.SparkPageRank`                                          |
| Jar files             | `file:///usr/lib/spark/examples/jars/spark-examples.jar`                           |
| Arguments             | `/data.txt`                                                                        |
| Max restarts per hour | `1`                                                                                |
| Apache Spark cluster  | **Compute Engine**                                                                 |
| Region                | **Region**                                                                         |
| Machine series        | **N2D**                                                                            |
| Manager node          | Select **Standard Persistent Disk**, then set **Machine Type** to `n2d-standard-2` |
| Worker node           | Select **Standard Persistent Disk**, then set **Machine Type** to `n2d-standard-2` |
| Max worker nodes      | `2`                                                                                |
| Primary disk size     | `100 GB`                                                                           |
| Internal IP only      | Deselect **Configure all instances to have only internal IP addresses**            |


Wait for the job to finish before trying to check your progress.

Example Managed Apache Spark job is shown below:

<img width="597" height="941" alt="image" src="https://github.com/user-attachments/assets/af16a5ce-3ee9-425b-8cd4-c45d1a7c2138" />


# Task 3. Use the Google Cloud Speech-to-Text API

- Use Google Cloud Speech-to-Text API to analyze the audio file gs://spls/gsp323/task3.flac. Once you have analyzed the file, upload the resulting file to: Cloud Speech Location, Ensure the uploaded object has its Content-Type set to application/json.

>> Note: If you are facing issues in this task, you can refer to the respective lab for troubleshooting: Google Cloud Speech-to-Text API: Qwik Start


# Task 4. Use the Cloud Natural Language API

- Use the Cloud Natural Language API to analyze the sentence from text about Odin. The text you need to analyze is "Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." Once you have analyzed the text, upload the resulting file to: Cloud Natural Language Location, Ensure the uploaded object has its Content-Type set to application/json.

>> Note: If you are facing issues in this task, you can refer to the respective lab for troubleshooting: Cloud Natural Language API: Qwik Start
