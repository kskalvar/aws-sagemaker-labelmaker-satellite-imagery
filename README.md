AWS SageMaker Labelmaker Satellite Imagery  
===========================================

This solution shows how to process map imagery using AWS SageMaker and Labelmaker to build an AI Model to predict buildings. This readme updates an article "Use Label Maker and Amazon SageMaker to automatically map buildings in Vietnam" by ZHUANGFANG NANA YI referenced below and provides a more basic step by step process.

First we'll build an EC2 Instance for downloading and preprocessing map images using labelmaker.  We'll then transfer the map data to S3.  Once on S3 we'll start a Jupyter Notebook using AWS SageMaker to build and deploy a model.  We can then use the Jupyter Notebook to predict buildings in the satellite imagery.

## Configure AWS EC2 Instance

Use the AWS Console to configure the EC2 Instance for processing map data.  This is a step by step process.

### AWS EC2 Dashboard
Select Instances

#### Instances
Click on "Launch Instance"

Choose AMI
```
Ubuntu Server 16.04 LTS
```  
Click on "Select"

Choose Instance Type
```
m5.4xlarge
```
Click on "Next"

Configure Instance  
Click on "Advanced Details
```
User data
Select "As file"
Click on "Choose File" and Select "cloud-init" from project cloud-deployment directory 
```  
Next

Add Storage  
Next

Add Tags  
Next

Configure Security Group  
Select "Select an existing security group"  
Select "default"

Review  
Click on "Launch"

## Obtain MapBox Access Token

You need to create an account on MapBox and obtain your access token from the Account section.

## Preprocess Satellite Imagery on AWS EC2 Instance

You will need to ssh into the AWS EC2 Instance you created above.  This is a step by step process.

### Check to insure cloud-init has completed

See contents of "/tmp/install-label-maker" it should say "installation complete".

### Download the labelmaker json configuration file

Download vietname.json and replace ```<ACCESS TOKEN>``` with API Key you created in your Mapbox Account.  
wget https://raw.githubusercontent.com/kskalvar/aws-sagemaker-labelmaker-satellite-imagery/master/labelmaker-config/vietnam.json  

### Preprocess satellite imagery using labelmaker

Run the following label-maker commands:
```
label-maker download --dest vietnam_building --config vietnam.json
label-maker labels --dest vietnam_building --config vietnam.json
label-maker preview -n 10 --dest vietnam_building --config vietnam.json
label-maker images --dest vietnam_building --config vietnam.json
label-maker package --dest vietnam_building --config vietnam.json

Note: You will likely see a couple of errors similar to the following:
OSError: cannot identify image file 'vietnam_building/tiles/104171-57805-17.jpg'
rm vietnam_building/tiles/104171-57805-17.jpg 
rerun package command
```

### Configure AWS CLI and copy results to S3
aws configure
```
AWS Access Key ID []:
AWS Secret Access Key []:
```
aws s3api create-bucket --bucket mybucket-labelmaker --region us-east-1  
aws s3 cp vietnam_building  s3://mybucket-labelmaker --recursive

## Configure AWS SageMaker
Use the AWS Console to configure a SageMaker Instance for processing map data.  This is a step by step process.

### AWS SageMaker Dashboard
#### Click on "Create notebook instance"
Notebook instance name: labelmaker  
Notebook instance type: ml.t2.medium  
IAM Role: Create a new role  
```
S3 buckets you specify:
Select Specific S3 buckets
Enter: mybucket-labelmaker
Click on "Create role"
```
Click on "Create notebook instance"

#### Display Notebook instances using the SageMaker Dashboard
Notebook instances  
Name: labelmaker  
Action: Open  

This should open the Jupyter Notebook in a new tab on your browser.

#### Upload SageMaker_mx-lenet.ipynb Jupyter Notebook
Once the notebook is uploaded, click on the notebook to open it.  
Run each cell Step by Step


## References
Use Label Maker and Amazon SageMaker to automatically map buildings in Vietnam
https://developmentseed.org/blog/2018/01/19/sagemaker-label-maker-case/

Mapbox API Key  
http://www.mapbox.com
