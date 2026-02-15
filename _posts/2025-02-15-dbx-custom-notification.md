---
layout: post
title: "Databricks - Custom Notification Mechanism"
date: 2025-02-15
categories: [technology, notes]
---

## The Idea

Databricks already offers notification mechanisms that can be configured at either Job level or Task level. This is great as it provides a starting point to inform someone that something went wrong or things succeeded. The issue I found is the limitation in terms of information you can provide via a channel. You will know that your job succeeded (or not), but what if you need more than that?

As I needed for my own use to send more rich information alerts, such as informing the total number of quarantined files and some reasons why files got quarantined, I reached the Databricks limitation in terms of customizability. After some research, the solution was obvious: use my own alert system using SMTP.

## Solution

### Setting Up Secrets

I will be using my own personal Gmail as an SMTP server. I have already created an application token, so to follow best practices, I will set up a Secret in Databricks.

![Create Secret Scope](/assets/imgs/create_secret_scope.png)

Now, I will go to my terminal and using `databricks-cli` I will set up the value for my scope.

![Set up secret value](/assets/imgs/Pasted_image_20260215134621.png)

Next, I will verify that my secret is accessible.

```python
try:
    gmail_password = dbutils.secrets.get(
        scope="gmail-application-secret",
        key="gmail-app-password"
    )
    print("Secret retrieved successfully!")
except Exception as e:
    print("Failed to retrieve secret:", e)
    
# Secret retrieved successfully!
```

### Data Ingestion and Validation

Now let's proceed with the ingestion. I have defined a simple SMTP class that will be used to send emails and also a simple Notebook that reads CSV files from the inbound volume and applies a few rules on some columns. The dataset used is a sample from [NYC Yellow Taxi Trip Data](https://www.kaggle.com/datasets/elemento/nyc-yellow-taxi-trip-data?resource=download).

The logic is as simple as possible. We read the data, and based on this filter we separate into two different dataframes, flagging valid and invalid data. If the invalid data count is greater than 0 at the end of the notebook, this should trigger sending the information via email.

Basically, anything that won't pass this validation should trigger the quarantine and move the rows there.

```python
df_validated = df.withColumn(
    "is_valid",
    when(
        (col("fare_amount") > 0) &
        (col("extra") >= 0) &
        (col("mta_tax") >= 0) &
        (col("tip_amount") >= 0),
        True
    ).otherwise(False)
).withColumn(
    "ValidationError",
    when(col("fare_amount") < 0, "fare_amount < 0")
    .when(col("extra") < 0, "extra < 0")
    .when(col("mta_tax") < 0, "mta_tax < 0")
    .when(col("tip_amount") < 0, "tip_amount < 0")
    .otherwise(None)
)
```

### Passing Task Values

Now, as this will be part of a Job execution, I need to know what was the exact number of invalid values during my ingestion. For this purpose, I will use [task values to pass information between tasks](https://docs.databricks.com/aws/en/jobs/task-values).

```python
dbutils.jobs.taskValues.set(
    key="has_invalid_data",
    value=True if df_invalid.count() > 0 else False
)
```

> **Note:** One subtle issue: `dbutils.jobs.taskValues.set` expects the value parameter to be a string.

### Setting Up the Job Pipeline

I will now set up the Job pipeline. It might be redundant to have three tasks, but this is for showcasing purposes. We first define the `ingest_data` task which will trigger the ingestion notebook that will set the variable value to either `True` or `False`. I will then use a Conditional Task to fetch this variable value, and based on this, I will trigger (or not) a new notebook that will send me the custom notification.

![Job pipeline configuration](/assets/imgs/Pasted_image_20260215142928.png)

![Conditional task setup](/assets/imgs/Pasted_image_20260215143020.png)

### Notification Logic

Now, the Notification notebook will use simple logic. I will use a new table named `notification_metadata` where I will store the emails of users who should be notified. I won't make it more advanced, but obviously I could add more information to have more control over what email addresses can do what, sort of a simple user-based access control system. For this purpose, the list of emails should be enough.

![Notification metadata table](/assets/imgs/Pasted_image_20260215145112.png)

I will use AI to create a nice email HTML body, then read the `INVALID_TABLE` and create some statistics.

Ingestion timestamp, filename, validation error, and count of errors should be enough for a short yet informative message.

![Email statistics](/assets/imgs/Pasted_image_20260215145506.png)

I have also put my email as a secret so I can share my code without exposing my email address. This is why you will see REDACTED.

![Email configuration](/assets/imgs/Pasted_image_20260215150316.png)

### Enabling AutoLoader

Great! One more step. Let's turn on AutoLoader so the pipeline will execute automatically when new files are uploaded.

![AutoLoader configuration](/assets/imgs/Pasted_image_20260215150448.png)

### Testing the Pipeline

Now let's upload data that has some errors on purpose.

![Upload test data](/assets/imgs/Pasted_image_20260215150753.png)

Great! The job executed successfully.

![Job execution output](/assets/imgs/Pasted_image_20260215154329.png)

And here is my nicely formatted output sent to my email:

![Email notification output](/assets/imgs/Pasted_image_20260215154358.png)