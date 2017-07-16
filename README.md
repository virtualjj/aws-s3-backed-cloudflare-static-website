# AWS S3 Backed Static Website With CloudFlare

I've seen a lot of S3 static website projects that use [AWS CloudFront](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) but not [CloudFlare](https://www.cloudflare.com/). CloudFront is a terrific service but I think CloudFlare&mdash;especially the free version&mdash;has A LOT to offer and is probably more accessible for folks new to CDN (Content Delivery Network) and WAF (Web Application Firewall).

## Caveats

## Getting Started

### Preparation

Before launching the stack you might want to check if the FQDN of your domain name is available. S3 is a global service so if *microsoft.com* for example is already taken, its taken in all regions. This is important to note as the stack will fail if the S3 bucket already exists. If you already have a bucket created you can simply include it in the stack. If you have the AWS CLI installed you can check whether or not a bucket is available or not with this command:

```
aws s3 ls s3://<FQDN>
```

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/prepstep-000-confirm-bucket-availability.jpg" alt="Check if bucket name already exists." height="75%" width="75%">
</p>



### Stack Deployment

1. Login to your AWS account and select the region that you want to deploy your S3 static website bucket. This is very important as its easy to accidentally open tabs in other regions.

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-000-login-region-check.jpg" alt="Make sure you are in the intended AWS region." height="75%" width="75%">
</p>


2. Click the **Launch Stack** button below to go directly to the CloudFormation service in the selected region of your AWS account.

[![Launch CloudFormation Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png
)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=s3-cloudflare-static-website&templateURL=https://s3-us-west-2.amazonaws.com/github.aws-s3-backed-cloudflare-static-website/aws-s3-backed-cloudflare-static-website.yml)

3. You will now see the **Create Stack** section of CloudFormation. The most important thing to confirm on this screen is the region. Click **Next**.

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-003-region-check-again.jpg" alt="Double-check the intended AWS region." height="75%" width="75%">
</p>

4. Enter your target domain name WITHOUT the www prefix. 

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-004-enter-fqdn.jpg" alt="Enter the FQDN of the bucket name." height="75%" width="75%">
</p>

5. To have the CodeCommit repo created for you using your domain name, leave it blank. The stack's deletion policy for this resource is set to *Retain* so if you delete the stack, the CodeCommit repository will not be deleted. Next enter an email address using a different domain to receive notifications for changes. Finally, I recommend setting up a group with limited access for users to push changes to CodeCommit for your static website instead of using an AWS user that has administrator access.

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-005-codecommit-setup.jpg" alt="Enter the FQDN of the bucket name." height="75%" width="75%">
</p>

6. You don't need to enter anything for the following parameters if this is the first time you launched this stack AND you don't already have the buckets created. Be careful deleting and re-creating buckets because sometimes it takes AWS a long time to propagate the changes so if you already have the buckets, I recommend for you to leave them.

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-006-site-and-logging-buckets.jpg" alt="Enter the FQDN of the bucket name." height="75%" width="75%">
</p>

7. There isn't much to do at the **Options** screen. Feel free to add some custom tags (good practice by the way) otherwise click **Next**.

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-007-options-tags.jpg" alt="Options screen, not much to do here." height="75%" width="75%">
</p>

8. This is the final confirmation screen before launching the stack. You can confirm the parameter settings that you chose. Also note that the stack will rollback on failure so you won't have any orphaned resources if something goes whacky. Make sure to check the acknowledgement check box and click **Create**.

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-008-confirm-before-launch.jpg" alt="Review stack, acknowledge, and click create." height="75%" width="75%">
</p>

9. Depending on how AWS is feeling it will take only a few minutes for the launch to complete.

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-009-stack-launch-success.jpg" alt="Wait for launch completion." height="75%" width="75%">
</p>

10. Click on the ***Options*** dropdown to view details of the created resources.

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-010-stack-options.jpg" alt="Click the options dropdown to see details of created resources." height="75%" width="75%">
</p>

11. Confirm that all of your buckets were created in S3.

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-011-confirm-created-s3-buckets.jpg" alt="Confirm S3 buckets were created." height="75%" width="75%">
</p>

12. Confirm that a CodeCommit repository was created.

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-012-confirm-created-codecommit-repo.jpg" alt="Confirm CodeCommit repo was created." height="75%" width="75%">
</p>

13. Confirm an IAM group was created and review the policy.

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/deploystep-013-confirm-created-iam-group.jpg" alt="Confirm an IAM group was created." height="75%" width="75%">
</p>





