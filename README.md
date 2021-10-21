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


# If the account is not present in IAM or does not have the editor role, follow the steps below to assign the required role.

In the Google Cloud console, on the Navigation menu, click Home.

Copy the project number (e.g. 729328892908).

On the Navigation menu, click IAM & Admin > IAM.

At the top of the IAM page, click Add.

For New members, type: {project-number}-compute@developer.gserviceaccount.com ( Replace {project-number} with your project number. )

For Role, select Project > Editor. Click Save. 
![image](https://user-images.githubusercontent.com/60198979/138269559-b97e42c8-4db5-4cbf-997a-11802bb71d03.png)

# Task 1. Create a Cloud SQL instance
In the Google Cloud Console, Select Navigation menu > SQL (in the Databases section).

Click Create instance.

Click Choose MySQL.

For Instance ID, type rentals.
![image](https://user-images.githubusercontent.com/60198979/138269642-17a52080-f059-4a3f-8f26-1d8f5f4db75e.png)

Scroll down and specify a Root password. Before you forget, note down the root password.

For Region select us-central1.

Click Create instance to create the instance. It will take a minute or so for your Cloud SQL instance to be provisioned.

# Task 2. Create tables

While you wait for your instance to be created, read the below mySQL script and answer the questions that follow.

CREATE DATABASE IF NOT EXISTS recommendation_spark;
USE recommendation_spark;
DROP TABLE IF EXISTS Recommendation;
DROP TABLE IF EXISTS Rating;
DROP TABLE IF EXISTS Accommodation;
CREATE TABLE IF NOT EXISTS Accommodation
(
  id varchar(255),
  title varchar(255),
  location varchar(255),
  price int,
  rooms int,
  rating float,
  type varchar(255),
  PRIMARY KEY (ID)
);
CREATE TABLE  IF NOT EXISTS Rating
(
  userId varchar(255),
  accoId varchar(255),
  rating int,
  PRIMARY KEY(accoId, userId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);
CREATE TABLE  IF NOT EXISTS Recommendation
(
  userId varchar(255),
  accoId varchar(255),
  prediction float,
  PRIMARY KEY(userId, accoId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);
SHOW DATABASES;

In Cloud SQL, click rentals to view instance information.

# Connect to the database

Find the Connect to this instance box on the page and click on Open Cloud Shell.

If required, click Continue. Wait for Cloud Shell to load.

Once Cloud Shell loads, you will see the below command already typed:

( gcloud sql connect rentals --user=root --quiet )

Press ENTER.

Wait for your IP Address to be whitelisted.

Allowlisting your IP for incoming connection for 5 minutes...⠹

When prompted, enter your password and press ENTER (note: you will not see your password typed in or even ****).
You can now run commands against your database!
![image](https://user-images.githubusercontent.com/60198979/138271841-4992c771-5418-46f1-81e6-04fc36c09bdc.png)

Run the following command:

SHOW DATABASES;

You should see the default system databases:

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+

Copy and paste the below SQL statement you analyzed earlier into the command line.

CREATE DATABASE IF NOT EXISTS recommendation_spark;
USE recommendation_spark;
DROP TABLE IF EXISTS Recommendation;
DROP TABLE IF EXISTS Rating;
DROP TABLE IF EXISTS Accommodation;
CREATE TABLE IF NOT EXISTS Accommodation
(
  id varchar(255),
  title varchar(255),
  location varchar(255),
  price int,
  rooms int,
  rating float,
  type varchar(255),
  PRIMARY KEY (ID)
);
CREATE TABLE  IF NOT EXISTS Rating
(
  userId varchar(255),
  accoId varchar(255),
  rating int,
  PRIMARY KEY(accoId, userId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);
CREATE TABLE  IF NOT EXISTS Recommendation
(
  userId varchar(255),
  accoId varchar(255),
  prediction float,
  PRIMARY KEY(userId, accoId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);
SHOW DATABASES;

Press ENTER

Confirm that you now see recommendation_spark as a database:
+----------------------+
| Database             |
+----------------------+
| information_schema   |
| mysql                |
| performance_schema   |
| recommendation_spark |
| sys                  |
+----------------------+

Run the following command to show the tables:

USE recommendation_spark;
SHOW TABLES;

Press ENTER.

Confirm that you see the three tables:

+--------------------------------+
| Tables_in_recommendation_spark |
+--------------------------------+
| Accommodation                  |
| Rating                         |
| Recommendation                 |
+--------------------------------+

Run the following query:

SELECT * FROM Accommodation;

# Task 3. Stage data in Cloud Storage
Option 1: Use the command line
Open a new Cloud Shell tab (do not use your existing mySQL Cloud Shell tab).

Copy and paste the following command:

echo "Creating bucket: gs://$DEVSHELL_PROJECT_ID"
gsutil mb gs://$DEVSHELL_PROJECT_ID
echo "Copying data to our storage from public dataset"
gsutil cp gs://cloud-training/bdml/v2.0/data/accommodation.csv gs://$DEVSHELL_PROJECT_ID
gsutil cp gs://cloud-training/bdml/v2.0/data/rating.csv gs://$DEVSHELL_PROJECT_ID
echo "Show the files in our bucket"
gsutil ls gs://$DEVSHELL_PROJECT_ID
echo "View some sample data"
gsutil cat gs://$DEVSHELL_PROJECT_ID/accommodation.csv

Press ENTER


# Option 2: Use the Cloud Console UI
Skip these steps if you have already loaded your data using the command line.

Navigate to Storage and select Cloud Storage > Browser.

Click Create Bucket (if one does not already exist).

Specify your project name as the bucket name.

Click Create.

Download the below files locally and then upload them inside of your new bucket:

accommodation.csv

rating.csv

# Task 4. Load data from Cloud Storage into Cloud SQL tables
Navigate back to SQL.

Click on rentals.

Import accommodation data
Click Import (top menu).

Specify the following:

Source: Click Browse > [Your-Bucket-Name] > accommodation.csv

Click Select.

Format of import: CSV

Database: select recommendation_spark from the dropdown list

Table: copy and paste: Accommodation

Click Import.
![image](https://user-images.githubusercontent.com/60198979/138273183-ca494540-eed9-42e9-8717-542c215f52be.png)

You will be redirected back to the Overview page. Wait one minute for the data to load.

Import user rating data
Click Import (top menu).

Specify the following:

Source: Click Browse > [Your-Bucket-Name] > rating.csv

Click Select.

Format of import: CSV

Database: select recommendation_spark from the dropdown list

Table: copy and paste: Rating

Click Import.

You will be redirected back to the Overview page. Wait one minute for the data to load.

# Task 5. Explore Cloud SQL data
If you closed your Cloud Shell connection to mySQL, open it again by finding Connect to this instance and clicking Open Cloud Shell.

Press ENTER when prompted to log in.

Provide your password and press ENTER.

Query the ratings data:

USE recommendation_spark;
SELECT * FROM Rating
LIMIT 15;

Use a SQL aggregation function to count the number of rows in the table.

SELECT COUNT(*) AS num_ratings
FROM Rating;

What is the average review rating of accommodations?

SELECT
    COUNT(userId) AS num_ratings,
    COUNT(DISTINCT userId) AS distinct_user_ratings,
    MIN(rating) AS worst_rating,
    MAX(rating) AS best_rating,
    AVG(rating) AS avg_rating
FROM Rating;


In machine learning, you will need a rich history of user preferences for the model to learn from. Run the below query to see which users have provided the most ratings.

SELECT
    userId,
    COUNT(rating) AS num_ratings
FROM Rating
GROUP BY userId
ORDER BY num_ratings DESC;

Exit the mysql prompt by typing exit.

# Task 6. Launch Dataproc
You use Dataproc to train the recommendations machine learning model based on users' previous ratings. You then apply that model to create a list of recommendations for every user in the database

To launch Dataproc and configure it so that each of the machines in the cluster can access Cloud SQL:

In the Cloud Console, on the Navigation menu (Navigation menu), click SQL and note the region of your Cloud SQL instance:
![image](https://user-images.githubusercontent.com/60198979/138273814-5b62a321-f63e-479d-807f-608ecd748a28.png)

In the snapshot above, the region is us-central1 and zone is us-central1-c.

In the Cloud Console, on the Navigation menu (Navigation menu), click Dataproc and click Enable API if prompted.

Once enabled, click Create cluster and name your cluster rentals.

Leave the Region as it is i.e. us-central1 and change the Zone to us-central1-c (in the same zone as your Cloud SQL instance). This will minimize network latency between the cluster and the database.

Click on Configure nodes.

For Master node, for Machine type, select n1-standard-2 (2 vCPUs, 7.5 GB memory).

For Worker nodes, for Machine type, select n1-standard-2 (2 vCPUs, 7.5 GB memory).

Leave all other values with their default and click Create. It will take 1-3 minutes to provision your cluster.

Note the Name, Zone and Total worker nodes in your cluster.

Copy and paste the below bash script into your Cloud Shell (optionally change CLUSTER, ZONE, NWORKERS if necessary before running)

echo "Authorizing Cloud Dataproc to connect with Cloud SQL"
CLUSTER=rentals
CLOUDSQL=rentals
ZONE=us-central1-c
NWORKERS=2
machines="$CLUSTER-m"
for w in `seq 0 $(($NWORKERS - 1))`; do
   machines="$machines $CLUSTER-w-$w"
done
echo "Machines to authorize: $machines in $ZONE ... finding their IP addresses"
ips=""
for machine in $machines; do
    IP_ADDRESS=$(gcloud compute instances describe $machine --zone=$ZONE --format='value(networkInterfaces.accessConfigs[].natIP)' | sed "s/\['//g" | sed "s/'\]//g" )/32
    echo "IP address of $machine is $IP_ADDRESS"
    if [ -z  $ips ]; then
       ips=$IP_ADDRESS
    else
       ips="$ips,$IP_ADDRESS"
    fi
done
echo "Authorizing [$ips] to access cloudsql=$CLOUDSQL"
gcloud sql instances patch $CLOUDSQL --authorized-networks $ips

Press ENTER. When prompted, type Y, then press ENTER again to continue.

Wait for the patching to complete. You will see the following:

Patching Cloud SQL instance...done.

On the main Cloud SQL page, under Connect to this instance, copy your Public IP Address to your clipboard. (Alternatively, write it down because you're using it next.)
![image](https://user-images.githubusercontent.com/60198979/138274001-2c8c2738-7809-4f48-80d8-e27981a83fcf.png)

# Task 7. Run the ML model
Next, you create a trained model and apply it to all the users in the system. Your data science team has created a recommendation model using Apache Spark and is written in Python. Copy it over into your staging bucket.

Copy over the model code by executing the below commands in Cloud Shell:
gsutil cp gs://cloud-training/bdml/v2.0/model/train_and_apply.py train_and_apply.py
cloudshell edit train_and_apply.py

When prompted, select Open in New Window.

Wait for the Editor UI to load.

Open the train_and_apply.py file, find line 30: CLOUDSQL_INSTANCE_IP, and paste the Cloud SQL IP address you copied earlier.

# MAKE EDITS HERE
{
CLOUDSQL_INSTANCE_IP = '<paste-your-cloud-sql-ip-here>'   # <---- CHANGE (database server IP)
CLOUDSQL_DB_NAME = 'recommendation_spark' # <--- leave as-is
CLOUDSQL_USER = 'root'  # <--- leave as-is
CLOUDSQL_PWD  = '<type-your-cloud-sql-password-here>'  # <---- CHANGE
  
ind line 33: CLOUDSQL_PWD and type in your Cloud SQL password,

The editor will autosave but to be sure, select File > Save.

From the Cloud Shell ribbon, click on the Open Terminal icon and copy this file to your Cloud Storage bucket using this Cloud Shell command:
  
gsutil cp train_and_apply.py gs://$DEVSHELL_PROJECT_ID 
}
  
# Task 8. Run your ML job on Dataproc
  
In the Dataproc console, click rentals cluster.

Click Submit job.

For Job type, select PySpark and for Main python file, specify the location of the Python file you uploaded to your bucket. Your <bucket-name> is likely to be your Project ID, which you can find by clicking on the Project ID dropdown in the top navigation menu.
![image](https://user-images.githubusercontent.com/60198979/138274729-5e9f6fb0-dc1b-406f-a4f8-1440b6a4befd.png)

  gs://<bucket-name>/train_and_apply.py

For Max restarts per hour, enter 1.

Click Submit.

Select Navigation menu > Dataproc > Job tab to see the Job status.
  
# Task 9. Explore inserted rows with SQL
In a new browser tab, open SQL (in the Databases section).

Click rentals to view details related to your Cloud SQL instance.

Under Connect to this instance section, click Open Cloud Shell. This will start a new Cloud Shell tab. In the Cloud Shell tab press ENTER.

It will take a few minutes to allow your IP for the incoming connection.

When prompted, type the root password you configured, then press ENTER.

At the mysql prompt, type:
  
USE recommendation_spark;
SELECT COUNT(*) AS count FROM Recommendation;
  
If you are getting an Empty Set (0) - wait for your Dataproc job to complete. If it's been more than 5 minutes, your job has likely failed and will require troubleshooting.

Tip: You can use the up arrow in Cloud Shell to return your previous command (or query in this case)
  
Find the recommendations for a user:
  
SELECT
    r.userid,
    r.accoid,
    r.prediction,
    a.title,
    a.location,
    a.price,
    a.rooms,
    a.rating,
    a.type
FROM Recommendation as r
JOIN Accommodation as a
ON r.accoid = a.id
WHERE r.userid = 10;
  
Your result should be similar to the below result:
  

| userid | accoid | prediction | title                       |

| 10     | 41     |  1.7748766 | Big Calm Manor              |
| 10     | 21     |  1.7174504 | Big Peaceful Cabin          |
| 10     | 46     |  1.7159091 | Colossal Private Castle     |
| 10     | 31     |  1.5783813 | Colossal Private Castle     |
| 10     | 32     |  1.5584077 | Immense Private Hall        |

  
 These are the five accommodations that you would recommend. Note that the quality of the recommendations is not great because the dataset was so small (note that the predicted ratings are not very high). Still, this lab illustrates the process you'd go through to create product recommendations.
  
# Recap:
In this lab, you:

Created a fully-managed Cloud SQL instance for rentals
Created tables and explored the schema with SQL
Ingested data from CSVs
Edited and ran a Spark ML job on Dataproc
Viewed prediction results


