- ### **Task 1: Creating a lambda function**
In the AWS Management Console, on the Services menu, enter 
``````
Lambda
``````
 From the search results, choose 
``````
Lambda
``````

Choose 
``````
Create Function
``````

Edit the following:
``````
Function name: resize_image
Runtime: Python3.9
``````

Expand the Change default execution role section. Choose 
``````
Use an existing role
``````
 In the dropdown list for Existing Role, 
choose 
``````
ResizeImageLambdaRole
``````

Choose 
``````
Create function
``````

Scroll to the Layers section.

Choose 
``````
Add a layer
``````
Choose 
``````
Custom layers
``````

From the Custom layers dropdown list, choose 
``````
PillowPythonLambdaLayer
``````

From the Version dropdown list, choose 
``````
1 or the option with the highest version number
``````

Choose 
``````
Add
``````

Scroll to the Code source section of the page.

Copy the code below, and replace the default source code in the lambda_function.py file with it.

```
import boto3
import os
import sys
import uuid
from urllib.parse import unquote_plus
from PIL import Image
import PIL.Image

s3_client = boto3.client('s3')

def resize_image(image_path, resized_path):
   with Image.open(image_path) as image:
      image.thumbnail(tuple(x / 2 for x in image.size))
      image.save(resized_path)

def lambda_handler(event, context):
   print('Begin resizing image')
   for record in event['Records']:
      bucket = record['s3']['bucket']['name']
      key = unquote_plus(record['s3']['object']['key'])
      print(bucket)
      print(key)
      tmpkey = key.replace('/', '')
      download_path = '/tmp/{}{}'.format(uuid.uuid4(), tmpkey)
      upload_path = '/tmp/resized-{}'.format(tmpkey)
      s3_client.download_file(bucket, key, download_path)
      resize_image(download_path, upload_path)
      s3_client.upload_file(upload_path, os.environ['RESIZED_BUCKET'], key)
   print('Resizing image complete')

```
- ### To deploy your Lambda function, choose 
``````
Deploy
``````

Choose the 
``````
Configuration tab
``````
, and then choose 
``````
General configuration
`````` 
Choose 
``````
Edit
``````

In the Memory field, 
``````
enter 512 MB
``````
Then chose
``````
Save
``````

At the top of these instructions, choose 
``````
Details
``````
 Next to AWS, choose 
 ``````
 Show
 ``````
  Copy the value next to ResizedBucketName. Use it as the value in step 25.

In the AWS Management Console, 
``````
choose environment variables
``````


Choose 
``````
Edit
``````


Choose 
``````
Add environment variable
``````

Enter the following values:
``````
  Key: RESIZED_BUCKET
  Value: Enter the value that you retrieved in 
  step 21
  ``````

Choose 
``````
Save
``````
You have created a Lambda function.👍😄 
___

### Task 2: Configuring an Amazon S3 trigger to invoke a Lambda function
In this task, you configure an S3 trigger on an existing S3 bucket and your Lambda function. The Lambda function resizes images and places them in another bucket.

In the Function overview section of the Lambda console near the top of the page, choose 

    Add trigger

In the Trigger configuration section, 

    choose S3 from the dropdown list

For Bucket, choose the bucket with 

    original 

in the name.

For Event Type, choose 

    All object create events

Acknowledge the notification for Recursive invocation by selecting the check box.

Choose 

    Add

***You have configured your Lambda function to be initiated when a new object is uploaded to the S3 bucket👍.***