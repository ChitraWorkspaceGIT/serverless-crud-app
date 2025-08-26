## Serverless CRUD App(s3+Lambda+API Gateway+DynamoDB)

A simple serverless CRUD (Create, Read, Update, Delete) application built on AWS using S3, Lambda, API Gateway, and DynamoDB.

This project demonstrates how to:

-->Upload data files to S3 → automatically store in DynamoDB via Lambda.

-->Manage items via a REST API built with API Gateway + Lambda.

-->Perform CRUD operations (GET, POST, DELETE) without managing servers.

**_Architecture_**

1. Amazon S3: Stores uploaded JSON files. Triggers a Lambda when a file is uploaded.

2. AWS Lambda: Processes S3 file uploads and saves data into DynamoDB. Provides business logic for API requests (GET, POST, DELETE).

3. Amazon DynamoDB: NoSQL database to store items.

4. Amazon API Gateway: Exposes REST API endpoints: GET /items → list all items,  POST /items → add an item,  DELETE /items/{id} → delete item by ID

**_Technologies Used_**

AWS Lambda (Python 3.11 runtime)

Amazon S3 (storage + event trigger)

Amazon DynamoDB (NoSQL database)

Amazon API Gateway (REST API)

IAM Roles & Policies (permissions)

CloudWatch Logs (monitoring & error logging)

**Step 1: Create an S3 Bucket**
(This bucket where you can upload JSON files.)

Open the AWS Console → Go to S3.

Click Create bucket.

Enter a unique name (example: my-serverlesschitra-bucket).

Choose a region (e.g., us-east-1) → Create.

Example Json file:

**Step 2: Create a DynamoDB Table**
(DynamoDB is where all your items will be stored like a database.)

Open DynamoDB in AWS Console.

Go to Tables → Create table.

Table name: ItemsTable.

Partition key: id (String).

Leave defaults → Create table.

**Step 3: Create an IAM Role for Lambda**

This role gives permission for Lambda functions to read from S3, write to DynamoDB, and log errors to CloudWatch.

Open IAM → Roles → Create Role.

Trusted entity: Lambda.

Attach permissions: AmazonS3FullAccess, AmazonDynamoDBFullAccess, CloudWatchLogsFullAccess

Name it: lambda-s3-dynamo-role.

**Step 4: Create Lambda Function (S3 → DynamoDB)**
This Lambda is triggered when a file is uploaded to S3. It reads the file and saves the data into DynamoDB.

Go to Lambda → Create Function.

Function name: s3-to-dynamodb-func.

Runtime: Python 3.11.

Choose the IAM role lambda-s3-dynamo-role.

Go & Paste this code: s3-to-dynamodb-func.py.txt

**Step 5: Connect S3 to Lambda**
Every time you upload a JSON file → it will be stored in DynamoDB automatically.
Open your Lambda → Add Trigger.

Choose S3.

Select your bucket (my-serverless-items-bucket).

Event type: PUT (when a new file is created).

Every time you upload a JSON file → it will be stored in DynamoDB automatically.

🟢 Step 6: Test the Flow

Upload item1.json into S3:

{
  "id": "1",
  "name": "Book",
  "category": "Stationery"
}


Go to DynamoDB → Explore Table.

✅ You’ll see the item saved (id=1, name=Book).

🟢 Step 7: Create Another Lambda (for API CRUD)

Go to Lambda → Create Function.

Function name: items-api-func.

Runtime: Python 3.9.

Attach the same role (lambda-s3-dynamo-role).

Paste this code:

import json
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('ItemsTable')

def lambda_handler(event, context):
    method = event['requestContext']['http']['method']

    if method == "GET":
        items = table.scan()
        return {"statusCode": 200, "body": json.dumps(items['Items'])}

    elif method == "POST":
        body = json.loads(event['body'])
        table.put_item(Item=body)
        return {"statusCode": 200, "body": "Item added"}

    elif method == "DELETE":
        item_id = event['pathParameters']['id']
        table.delete_item(Key={'id': item_id})
        return {"statusCode": 200, "body": "Item deleted"}

    return {"statusCode": 400, "body": "Unsupported method"}


👉 Why?
This Lambda handles API requests to view, add, or delete items.

🟢 Step 8: Set Up API Gateway

Go to API Gateway → Create API → HTTP API.

Name: items-api.

Integrate it with items-api-func.

Add routes:

GET /items → items-api-func

POST /items → items-api-func

DELETE /items/{id} → items-api-func

Deploy → Stage name: prod.

👉 After deployment, you get an Invoke URL like:

https://abc123.execute-api.ap-south-1.amazonaws.com/prod

🟢 Step 9: Test the API

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


✅ Now you have a fully working serverless CRUD app!

🎯 What You Achieved

Learned how S3 can trigger Lambda.

Used DynamoDB as a serverless database.

Created REST APIs with API Gateway + Lambda.

Tested end-to-end CRUD operations without any servers.

Logging & error tracking available in CloudWatch.
