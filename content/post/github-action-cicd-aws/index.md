+++
author = "Jeff Chang"
title = "Automate your deployment with Github Action CI/CD and AWS"
date = "2022-06-18"
description = "In this article, we will be going through step by step on how to automate the deployment on AWS S3 static site hosting with Github action"
tags = [
    "devops"
]
categories = [
	"DevOps"
]
+++

## Table of contents

- [Create AWS S3 bucket for website hosting](#aws-s3-static-website-hosting)
- [Create IAM role in AWS](#aws-iam)
- [Attach credentials in Github repository](#github-credentials)
- [Configure Github workflows](#github-workflows)

## Create AWS S3 bucket for website hosting<a name="aws-s3-static-website-hosting"></a>

Before everything started, we need to ensure our AWS S3 bucket is created so that our files could be uploaded via Github workflow later.

### 1. To create new S3 bucket

Search and navigate to AWS Management Console S3 and click on the **Create bucket** button.

<div style="max-width:80%; margin:0 auto">
    <img src="create_s3_bucket.png" alt="Create AWS S3 bucket" />
</div>

### 2. Create bucket and enable public access

Enter our bucket name and select AWS region.

<div style="max-width:80%; margin:0 auto">
    <img src="s3_bucket.png" alt="Create AWS S3 bucket" />
</div>

<!-- prettier-ignore -->
In order to allow everyone access our website in public, we would need to uncheck the *Block all public access* rule.

<div style="max-width:80%; margin:0 auto">
    <img src="s3_bucket_rules.png" alt="Create AWS S3 bucket" />
</div>

Also, we would need to attach our bucket policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME_HERE/*"
        }
    ]
}
```

### 3. Enable Static website hosting

The next step will be enable the static website hosting property for the bucket. To do this, we can go to the _Properties_ tab of the bucket and **scroll to the most bottom** to click on the **Edit** button

<div style="max-width:80%; margin:0 auto">
    <img src="bucket_properties.png" alt="S3 bucket properties" />
</div>

<div style="max-width:80%; margin:0 auto">
    <img src="enable_static_hosting.png" alt="Enable Static website hosting" />
</div>

We can now enable our static website hosting option and we would need to specify the filename refers the website home page. This is normally **index.html** file. We could also specified the error webpage and it will automatically shows up when something goes wrong in our website.

<div style="max-width:80%; margin:0 auto">
    <img src="configure_static_website.png" alt="Configure Static website hosting" />
</div>

And our website will now able to access by anyone with the given url

<div style="max-width:100%; margin:0 auto">
    <img src="website_url.png" alt="Website url" />
</div>

## Create IAM role in AWS<a name="aws-iam"></a>

Since we are going automate upload files to S3 bucket process in github action later. We would need to generate a AWS secret and access key so that it will authorize our github repository to perform the upload action

### 1. To create new AWS IAM user

Search and navigate to AWS Management Console S3 and click on the **Create bucket** button.

<div style="max-width:100%; margin:0 auto">
    <img src="create_new_user.png" alt="Create new AWS IAM user" />
</div>

### 2. Create AWS IAM user with Programmatic Access

Select **Programmatic Access** for the credential type

<div style="max-width:80%; margin:0 auto">
    <img src="aws_iam_user.png" alt="Create new AWS IAM user" />
</div>

### 3. Attach S3 bucket policy

Select attach existing policies directly and select **AmazonS3FullAccess** option.

_Note: You may also select the 'Add user to group' option if you already have a group policy, as long as it has **AmazonS3FullAccess** policy, then you are good to go_

<div style="max-width:80%; margin:0 auto">
    <img src="iam_attach_policy.png" alt="Attach S3 policy" />
</div>

### 4. Get credentials

We will skip the **"Add Tags"** step here but feel free to add a proper tag. After we done the previous steps and hit on the **Create User** button, please **download** the given CSV it will no longer available on the site once we close the webpage.

And when you open the CSV file, it will shows you the **Access key ID** and **Secret access key** value

<div style="max-width:80%; margin:0 auto">
    <img src="download_credentials.png" alt="Download credentials" />
</div>

## Attach credentials in Github repository<a name="github-credentials"></a>

And now we could:

1. Go to our github repository
2. Navigate to the Setting tab
3. Select **Secrets** -> **Actions** on the left panel
4. Create our custom repository secrets such as **AWS_ACCESS_KEY_ID, AWS_REGION** and **AWS_SECRET_ACCESS_KEY** by referring the value from the csv file we downloaded in earlier.
5. Delete the CSV file as it no longer needed

<div style="max-width:80%; margin:0 auto">
    <img src="github_credentials.png" alt="Github credentials" />
</div>
