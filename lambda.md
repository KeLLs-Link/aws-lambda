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
___

### Task 3: Uploading an image to the Amazon S3 bucket
In this task, you upload an image file to your bucket.

Open the context (right-click) menu for the following link, and download the file to your computer:

    large-image.jpg


In the AWS Management Console, on the Services menu, enter 

    S3 

From the search results, choose 

    S3

Choose the link for the bucket that has 

    original 

in the name.

Choose 

    Upload

Choose 

    Add files

Choose 

    the file that you downloaded

Choose 

    Upload.
  
  Your file is uploaded to the bucket.🌍
  
  chose 
  
    close 

- ***If you are working on Task 4, you can now return to step 55.***

When your Lambda function runs correctly, the image file that you uploaded is reduced in size and placed in the S3 bucket that you specified when you set the environment variable for RESIZED_BUCKET.

Return to the Buckets section in the S3 Console.

Choose the link for the bucket that has resized in the name.

   Notice the file size. It's significantly reduced from the original size of 4.9 MB.

When you uploaded the image file, S3 initiated the resize_image Lambda function. Review the logs to see how your function performed.

    Navigate back to the Lambda console

Choose the 

    link for the resize_image function

Choose the 

    Monitor tab

Choose 

    Logs

In the Recent Invocations table, choose a row to expand the details.
Notice the metrics that are recorded for each function invocation. You might have to wait for up to one minute for the data to be updated.

The DurationInMS column tells you how long your function ran for this invocation.  

The first time that your Lambda function is invoked, the Lambda execution environment has to download your code and start a new execution environment. This process is called a cold start. The @initDuration metric in the Recent invocation details signifies the cold start time.

Note the MemorySetInMB column. The amount of memory that's available to your Lambda function can be adjusted to affect the performance of your Lambda function.

The amount of memory also determines the amount of virtual CPU available to a function. Adding more memory proportionally increases the amount of CPU, which increases the overall computational power available. If a function is CPU-, network- or memory-bound, then changing the memory setting can dramatically improve its performance.

Notice how long it took your function to run. It could be faster. Adjust your Lambda function so that it runs faster.

### Task 4: Optimizing Lambda function memory for performance
Choose the 

    Configuration tab

Choose 

    General Configuration

Choose 

    Edit

Adjust Memory to 

    1024 MB

Choose 

    Save

Upload your image to the original S3 bucket again. If you need help, review steps 35-41.


Repeat steps 49-54 using 2048 MB and 3008 MB as the value for Memory.

***Your function now runs in approximately 500 milliseconds with 3008 MB of memory and an image that is 5 MB. 👍***

# Summary
In this project, I created a Lambda function and configured an S3 trigger on that Lambda function. I uploaded an image to an S3 bucket which initiated the Lambda function to resize the image and place it in another bucket. After reviewing the logs to see performance metrics, you optimized the Lambda function by increasing the available memory for the function.

**Excellent work 👍**