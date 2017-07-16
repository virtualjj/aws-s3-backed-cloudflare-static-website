# AWS S3 Backed Static Website With CloudFlare

I've seen a lot of S3 static website projects that use [AWS CloudFront](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) but not [CloudFlare](https://www.cloudflare.com/). CloudFront is a terrific service but I think CloudFlare&mdash;especially the free version&mdash;has A LOT to offer and is probably more accessible for folks new to CDN (Content Delivery Network) and WAF (Web Application Firewall).

## Caveats

## Getting Started

### Preparation

### Stack Deployment

1. Login to your AWS account and select the region that you want to deploy the OpenVPN AS instance. This is very important as its easy to accidentally open tabs in other regions.

2. Click the **Launch Stack** button below to go directly to the CloudFormation service in the selected region of your AWS account.

[![Launch CloudFormation Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png
)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=s3-cloudflare-static-website&templateURL=https://s3-us-west-2.amazonaws.com/github.aws-s3-backed-cloudflare-static-website/aws-s3-backed-cloudflare-static-website.yml)


<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/caveats-cannot-create-microsoft-com-bucket.jpg" alt="Cannot create static website bucket names that have already been claimed." height="75%" width="75%">
</p>