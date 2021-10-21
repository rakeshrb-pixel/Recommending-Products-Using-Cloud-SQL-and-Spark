# Recommending-Products-Using-Cloud-SQL-and-Spark
In this lab, I populate rentals data in Cloud SQL for the rentals recommendation engine to use. The recommendations engine itself will run on Dataproc using Spark ML.
#  Objectives
In this lab, I learn how to perform the following tasks:

1) Create a Cloud SQL instance

2) Create database tables by importing .sql files from Cloud Storage

3) Populate the tables by importing .csv files from Cloud Storage

4) Allow access to Cloud SQL

5) Explore the rentals data using SQL statements from Cloud Shell

# Check project permissions

In the Google Cloud console, on the Navigation menu (nav-menu.png), click IAM & Admin > IAM.

Confirm that the default compute Service Account {project-number}-compute@developer.gserviceaccount.com is present and has the editor role assigned. The account prefix is the project number, which you can find on Navigation menu > Home.

![image](https://user-images.githubusercontent.com/60198979/138269291-4d2c4977-9f99-4bce-826d-d9a9ae110584.png)

