# AWS S3 Backed Static Website With CloudFlare

I've seen a lot of S3 static website projects that use [AWS CloudFront](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) but not [CloudFlare](https://www.cloudflare.com/). CloudFront is a terrific service but I think CloudFlare&hellip;especially the free version&hellip;has A LOT to offer and is probably more accessible for folks new to CDN (Content Delivery Network) and WAF (Web Application Firewall).

## Caveats

First, you will need to create an static website hosting S3 bucket that matches the FQDN of your domain name. Unfortunately, AWS's S3 space is a global service and there is a slight chance that someone is already using the name. For example if you wanted to try *microsoft.com* you'd get the following error:

<p align="center"> 
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/caveats-cannot-create-microsoft-com-bucket.jpg" alt="Cannot create static website bucket names that have already been claimed." height="75%" width="75%">
</p>
