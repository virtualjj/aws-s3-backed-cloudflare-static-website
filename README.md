# AWS S3 Backed Static Website With CloudFlare

I've seen a lot of S3 static website projects that use [AWS CloudFront](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) but not [CloudFlare](https://www.cloudflare.com/). CloudFront is a terrific service but I think CloudFlare&mdash;especially the free version&mdash;has A LOT to offer and is probably more accessible for folks new to CDN (Content Delivery Network) and WAF (Web Application Firewall).

## Prerequisites

Since this this project is supposed to work with CloudFlare, you will of course need a CloudFlare account. If you don't have a free account yet create one [here](https://www.cloudflare.com/a/sign-up).

CloudFlare does a great explaining how to get started but make sure the domain name for the site you plan on hosting with AWS S3 has it's name servers set to CloudFlare's&mdash;here is an example:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/prereq-000-confirm-cloudflare-dns-setup.jpg" alt="Example DNS configured with CloudFlare." height="75%" width="75%">
</p>

## Caveats

Combining S3 static web hosting buckets with CloudFlare will allow you to enable HTTPS but it is not true end-to-end TLS. Actually, you wouldn't have it with CloudFront + AWS generated TLS certificates (ACM) either because static web hosting enabled S3 buckets do not work with HTTPS as an ***origin*** anyway:

> If your Amazon S3 bucket is configured as a website endpoint, you can't configure CloudFront to use HTTPS to communicate with your origin because Amazon S3 doesn't support HTTPS connections in that configuration.

Source: http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-https-cloudfront-to-s3-origin.html

## CONFIRM S3 BUCKET NAME AVAILABILITY

Before launching the stack confirm whether or not your domain name is available. S3 is a global service so if ***example.com***&mdash;for example&mdash;is already taken, it's taken in all regions and ***all*** AWS accounts. This is important to note as the stack will fail if the S3 bucket already exists but not in your region and/or AWS account.

Check by either creating a bucket in the AWS console or the AWS CLI.

**AWS Console Example:**

This is the error you'll get if the bucket is already claimed:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/confirms3-000-s3-bucket-available-console.jpg" alt="Example DNS configured with CloudFlare." height="75%" width="75%">
</p>

If you were able to create the bucket, don't delete per AWS's advice:

> If you want to continue to use the same bucket name, don't delete the bucket. We recommend that you empty the bucket and keep it. After a bucket is deleted, the name becomes available to reuse, but the name might not be available for you to reuse for various reasons. For example, it might take some time before the name can be reused and some other account could create a bucket with that name before you do.

Source: http://docs.aws.amazon.com/AmazonS3/latest/user-guide/delete-bucket.html

**AWS CLI Example:**

Here is one example of what an already claimed bucket message will look like:

```
> aws s3 ls s3://example.com
An error occurred (AccessDenied) when calling the ListObjects operation: Access Denied
```

If it doesn't exist, you'll get this message:

```
>aws s3 ls s3://tutorialstuff.xyz

An error occurred (NoSuchBucket) when calling the ListObjects operation: The specified bucket does not exist
```
If your bucket name is available let the stack create it for you to save you some clicking.


## STACK DEPLOYMENT

1. Login to your AWS account and select the region that you want to deploy your S3 static website bucket. This is very important as its easy to accidentally open tabs in other regions.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-000-login-region-check.jpg" alt="Make sure you are in the intended AWS region." height="75%" width="75%">
</p>


2. Click the **Launch Stack** button below to go directly to the CloudFormation service in the selected region of your AWS account.

[![Launch CloudFormation Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png
)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=s3-cloudflare-static-website&templateURL=https://s3-us-west-2.amazonaws.com/github-aws-s3-backed-cloudflare-static-website/aws-s3-backed-cloudflare-static-website.yml)

3. You will now see the **Create Stack** section of CloudFormation. The most important thing to confirm on this screen is the region&mdash;again. The CloudFormation template is stored and hosted publicly on ***my*** AWS account. Click **Next**.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-003-region-check-again.jpg" alt="Double-check the intended AWS region." height="75%" width="75%">
</p>

4. Enter your __ROOT DOMAIN__ name __without__ the *www* prefix. You also need to enter an email address that has a __different__ domain name than the one you will use for the static website S3 bucket. Leave __Repo Name__ and __CodeCommit User__ blank to follow along with this tutorial.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-004-enter-fqdn.jpg" alt="Enter the FQDN of the bucket name." height="75%" width="75%">
</p>

5. If this is the first time using this template you will typically leave all these fields blank. However, if you already have __Website Bucket Name__, __Redirect Bucket Name__, and __Logs for Bucket Name__ already created then enter them here. The [DeletionPolicy](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-deletionpolicy.html) attribute is set to **retain** for S3 buckets created by this template. Click **Next**.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-005-leave-blank-or-enter-names.jpg" alt="Leave fields blank unless the S3 buckets names you want to use are already created." height="75%" width="75%">
</p>

6. There isn't anything to do **Create Stack Options** screen so click **Next**.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-006-create-stack-options.jpg" alt="Example Create Stack Options - Click Next." height="75%" width="75%">
</p>

7. This is your last chance to make sure you've checked whether or not your bucket names have already been taken or already created. Check the checkbox for the message about IAM resources - this stack creates an IAM group, user, and policy so that is what this message appears. Click **Next**.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-007-confirm-settings-checkbox.jpg" alt="Confirm your settings and check the IAM resources acknowledgement checkbox." height="75%" width="75%">
</p>

8. The stack should take about 2 ~ 3 minutes to complete but make sure to check your email and subscribe to the SNS subscription notification that you received otherwise the stack will get stuck.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-008-check-email-sns-subscribe.jpg" alt="Check email for SNS subscription." height="75%" width="75%">
</p>

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-008-sns-subscribe-success.jpg" alt="When you click on the subscription link you should see a success message." height="75%" width="75%">
</p>

9. You should now have a green __CREATE_COMPLETE__ status for the CloudFormation stack.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-009-stack-launch-success.jpg" alt="Stack successfully created." height="75%" width="75%">
</p>

10. Click on the ***Options*** drop down to view details of the created resources. We will reference these throughout this tutorial.



## Acknowledgments

Thanks to Eric Hammond for inspiring me to work on this project. Much of his [alestic/aws-git-backed-static-website](https://github.com/alestic/aws-git-backed-static-website) template was used as the basis for this project.
