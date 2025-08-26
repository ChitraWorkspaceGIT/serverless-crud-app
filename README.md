#  Serverless CRUD App (S3 + Lambda + API Gateway + DynamoDB)

A simple **serverless CRUD (Create, Read, Update, Delete)** application built on AWS using **S3, Lambda, API Gateway, and DynamoDB**.

---

##  Project Demonstrates

-  Upload data files to **S3** → automatically store in **DynamoDB** via **Lambda**.  
-  Manage items via a **REST API** built with API Gateway + Lambda.  
-  Perform CRUD operations (**GET, POST, DELETE**) without managing servers.  

---

##  Architecture

- **Amazon S3** → Stores uploaded JSON files, triggers a Lambda on file upload.  
- **AWS Lambda** → Processes S3 uploads & API requests (GET, POST, DELETE).  
- **Amazon DynamoDB** → NoSQL database to store items.  
- **Amazon API Gateway** → Exposes REST API endpoints:  
  - `GET /items` → list all items  
  - `POST /items` → add an item  
  - `DELETE /items/{id}` → delete item by ID  

---

##  Technologies Used

- AWS Lambda (**Python 3.11 & 3.12 runtime**)  
- Amazon S3 (storage + event trigger)  
- Amazon DynamoDB (NoSQL database)  
- Amazon API Gateway (REST API)  
- IAM Roles & Policies (permissions)  
- CloudWatch Logs (monitoring & error logging)  

---

##  Step-by-Step Setup

###  Step 1: Create an S3 Bucket
- This bucket where you can upload JSON files.
- Open AWS Console → Go to **S3**  
- Click **Create bucket** → Name: `my-serverlesschitra-bucket`  
- Choose region (e.g., `us-east-1`) → **Create**  

---

###  Step 2: Create DynamoDB Table
- DynamoDB is where all your items will be stored like a database.
- Open **DynamoDB** → Create Table  
- Table name: `ItemsTable`  
- Partition key: `id (String)`  
- Leave defaults → **Create**  

---

###  Step 3: Create IAM Role for Lambda
- This role gives permission for Lambda functions to read from S3, write to DynamoDB, and log errors to CloudWatch
- Go to **IAM → Roles → Create Role**  
- Trusted entity: **Lambda**  
- Attach policies:  
  - `AmazonS3FullAccess`  
  - `AmazonDynamoDBFullAccess`  
  - `CloudWatchLogsFullAccess`  
- Name: `lambda-s3-dynamo-role`  

---

###  Step 4: Create Lambda (S3 → DynamoDB)
- This Lambda is triggered when a file is uploaded to S3. It reads the file and saves the data into DynamoDB.
- Go to **Lambda → Create Function**  
- Name: `s3-to-dynamodb-func`  
- Runtime: `Python 3.11`  
- Role: `lambda-s3-dynamo-role`  
- Paste code from `s3-to-dynamodb-func.py.txt`  

---

###  Step 5: Connect S3 to Lambda
- Every time you upload a JSON file → it will be stored in DynamoDB automatically.
- In Lambda, add a **Trigger → S3**  
- Select bucket: `my-serverlesschitra-bucket`  
- Event type: **PUT (new file created)**  

---

###  Step 6: Test the Flow
Upload sample file `sample-data/file.txt`:

```json
{
  "id": "1",
  "name": "Book",
  "category": "Stationery"
}

Check DynamoDB → Table Explorer → Item appears (id=1, name=Book).

**Step 7: Create Another Lambda (for API CRUD)**

This Lambda handles API requests to view, add, or delete items.

Go to Lambda → Create Function.

Function name: items-api-func.

Runtime: Python 3.12.

Attach the same role (lambda-s3-dynamo-role).

GO & Paste code from: items-api-func.py.txt

**Step 8: Setup API Gateway**

Go to API Gateway → Create API → HTTP API

Name: items-api

Integrate with items-api-func

Add routes:

GET /items → items-api-func

POST /items → items-api-func

DELETE /items/{id} → items-api-func

Deploy → Stage: prod

👉 After deployment, you get an Invoke URL like:

https://abc123.execute-api.ap-south-1.amazonaws.com/prod

**Step 9: Test the API**

GET all items

GET https://abc123.execute-api.ap-south-1.amazonaws.com/prod/items

POST add new item

POST https://abc123.execute-api.ap-south-1.amazonaws.com/prod/items

Body:
{
  "id": "2",
  "name": "Laptop",
  "category": "Electronics"
}

DELETE an item

DELETE https://abc123.execute-api.ap-south-1.amazonaws.com/prod/2


Now you have a fully working serverless CRUD app!

**_Achieved_**

Learned how S3 can trigger Lambda.

Used DynamoDB as a serverless database.

Created REST APIs with API Gateway + Lambda.

Tested end-to-end CRUD operations without any servers.

Logging & error tracking available in CloudWatch.

---

### Step 10: Enable Monitoring with CloudWatch

AWS CloudWatch helps you monitor Lambda execution, errors, and performance.

#### 1. View Lambda Logs
- Go to **Lambda → Monitor tab → View logs in CloudWatch**  
- Each function execution creates a log stream.  
- You can check:  
  -  `START` and `END` of execution  
  -  Execution duration (ms)  
  -  Errors or exceptions (`KeyError`, `NoSuchKey`, etc.)

#### 2. Create CloudWatch Log Group (Optional)
- Open **CloudWatch → Logs → Log groups**  
- By default, Lambda creates a log group like:  

