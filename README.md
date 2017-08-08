# AWS S3 Backed Static Website With Cloudflare

  - [PREREQUISITES](#prerequisites)
  - [CAVEATS](#caveats)
  - [CONFIRM S3 BUCKET NAME AVAILABILITY](#confirm-s3-bucket-name-availability)
  - [STACK DEPLOYMENT](#stack-deployment)
  - [CONFIRM STATIC HOSTING WORKS](#confirm-static-hosting-works)
  - [ADD CNAME TO CLOUDFLARE](#add-cname-to-cloudflare)
  - [SETUP CLOUDFLARE HTTPS REDIRECT](#setup-cloudflare-https-redirect)
  - [SETUP HUGO WEBSITE EXAMPLE](#setup-hugo-website-example)
  - [CONFIGURE CODECOMMIT USER](#configure-codecommit-user)
  - [ACKNOWLEDGMENTS](#acknowledgments)

I've seen a lot of S3 static website projects that use [AWS CloudFront](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html) but not [Cloudflare](https://www.Cloudflare.com/). CloudFront is a terrific service but I think Cloudflare&mdash;especially the free version&mdash;has A LOT to offer and is probably more accessible for folks new to CDN (Content Delivery Network) and WAF (Web Application Firewall).

## PREREQUISITES

Since this this project is supposed to work with Cloudflare, you will of course need a Cloudflare account. If you don't have a free account yet create one [here](https://www.Cloudflare.com/a/sign-up).

Cloudflare does a great job explaining how to get started but make sure the domain name for the site you plan on hosting with AWS S3 has it's name servers set to Cloudflare's&mdash;here is an example of the one that I will use for this tutorial:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/prereq-000-confirm-cloudflare-dns-setup.jpg" alt="Example DNS configured with Cloudflare." height="75%" width="75%">
</p>

Notice the red arrow pointing to __SSL:Flexible__&mdash;that brings us to a small caveat&hellip;

## CAVEATS

Combining S3 static web hosting buckets with Cloudflare will allow you to enable HTTPS but it is not true end-to-end TLS. Actually, you wouldn't have it with CloudFront + AWS generated TLS certificates (ACM) either because static web hosting enabled S3 buckets do not work with HTTPS as an ***origin*** anyway:

> If your Amazon S3 bucket is configured as a website endpoint, you can't configure CloudFront to use HTTPS to communicate with your origin because Amazon S3 doesn't support HTTPS connections in that configuration.

Source: http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-https-cloudfront-to-s3-origin.html

You will need to set Cloudflare's SSL option to **Flexible** which means that your visitors will connect to your web site over TLS to Cloudflare's CDN and then Cloudflare will connect to your AWS S3 origin (i.e. static website bucket) *unencrypted*. This shouldn't be an issue for personal blogs or small websites but I wanted to clarify this caveat as it could be a showstopper for some. Details about Cloudflare's SSL options can be referenced [here](https://support.Cloudflare.com/hc/en-us/articles/204144518-SSL-FAQ).

## CONFIRM S3 BUCKET NAME AVAILABILITY

Before launching the stack confirm whether or not your domain name is available. S3 is a global service so if ***example.com***&mdash;for example&mdash;is already taken, it's taken in all regions and ***all*** AWS accounts. This is important to note as the stack will fail if the S3 bucket already exists but not in your region and/or AWS account.

Check by either creating a bucket in the AWS console or trying to list the contents of the bucket using the AWS CLI.

**AWS Console Example:**

This is the error you'll get if the bucket is already claimed:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/confirms3-000-s3-bucket-available-console.jpg" alt="Example DNS configured with Cloudflare." height="75%" width="75%">
</p>

If you were able to create the bucket, don't delete it per AWS's advice:

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
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/stackdeployment-001-login-region-check.jpg" alt="Make sure you are in the intended AWS region." height="75%" width="75%">
</p>


2. Click the **Launch Stack** button below to go directly to the CloudFormation service in the selected region of your AWS account.

[![Launch CloudFormation Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png
)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=s3-cloudflare-static-website&templateURL=https://s3-us-west-2.amazonaws.com/github-aws-s3-backed-cloudflare-static-website/aws-s3-backed-cloudflare-static-website.yml)

3. You will now see the **Create Stack** section of CloudFormation. The most important thing to confirm on this screen is the region&mdash;again. The CloudFormation template is stored and hosted publicly on ***my*** AWS account. Click **Next**.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/stackdeployment-003-region-check-again.jpg" alt="Double-check the intended AWS region." height="75%" width="75%">
</p>

4. Enter your __ROOT DOMAIN__ name __without__ the *www* prefix. You also need to enter an email address that has a __different__ domain name than the one you will use for the static website S3 bucket. Leave __Repo Name__ and __CodeCommit User__ blank to follow along with this tutorial. Scroll down.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/stackdeployment-004-enter-root-domain-name.jpg" alt="Enter the FQDN of the bucket name." height="75%" width="75%">
</p>

5. If this is the first time using this template you will typically leave all these fields blank. However, if you already have __Website Bucket Name__, __Redirect Bucket Name__, and __Logs for Bucket Name__ already created then enter them here. The [DeletionPolicy](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-deletionpolicy.html) attribute is set to **retain** for S3 buckets created by this template. Click **Next**.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/stackdeployment-005-leave-blank-or-enter-names.jpg" alt="Leave fields blank unless the S3 buckets names you want to use are already created." height="75%" width="75%">
</p>

6. There isn't anything to do at the **Create Stack Options** screen so click **Next**.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/stackdeployment-006-create-stack-options.jpg" alt="Example Create Stack Options - Click Next." height="75%" width="75%">
</p>

7. This is your last chance to make sure you've checked whether or not your bucket names have already been taken or already created. Check the ***I acknowledge...*** check box for the message about IAM resources. This stack creates an IAM group, user, and policy so that is why this message appears. Click **Next**.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/stackdeployment-007-confirm-settings-checkbox.jpg" alt="Confirm your settings and check the IAM resources acknowledgement checkbox." height="75%" width="75%">
</p>

8. The stack should take about 2 ~ 3 minutes to complete but make sure to check your email and subscribe to the SNS subscription notification that you received otherwise the stack will get stuck.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/stackdeployment-008-check-email-sns-subscribe.jpg" alt="Check email for SNS subscription." height="75%" width="75%">
</p>

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/stackdeployment-008-sns-subscribe-success.jpg" alt="When you click on the subscription link you should see a success message." height="75%" width="75%">
</p>

9. You should now have a green __CREATE_COMPLETE__ status for the CloudFormation stack.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/stackdeployment-009-stack-launch-success.jpg" alt="Stack successfully created." height="75%" width="75%">
</p>

10. Click on the ***Outputs*** drop down to view details of the created resources. We will reference these throughout this tutorial.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/stackdeployment-010-confirm-stack-outputs.jpg" alt="View stack Outputs for details of created resources." height="75%" width="75%">
</p>

## CONFIRM STATIC HOSTING WORKS

Basically all you have to do is upload an *index.html* document to your static web hosting enabled S3 bucket. However, this can be tricky if you are new to S3 so I'll provide a more "involved" example.

1. In the **Outputs** section of the CloudFormation stack that you launched you should see the S3 endpoints for the site bucket and the redirect bucket:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/confirmhosting-001-find-s3-endpoints.jpg" alt="Find the S3 endpoint URLs in the Outputs section of the launched CloudFormation stack." height="75%" width="75%">
</p>

2. Click on the URL for **SiteBucketEndpoint**&mdash;you should see the following error. This is because the objects (i.e. keys) index.html and error.html do not exist:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/confirmhosting-002-no-key-exists.jpg" alt="Cannot open the S3 endpoint because no key (i.e. object) exists for index.html or error.html." height="75%" width="75%">
</p>

3. Upload a simple *index.html* file. I will use one that has only the following line:

```
<h1>WORKS</h1>
```

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/confirmhosting-003-drag-in-files.jpg" alt="Drag or upload your website that at least has an index.html." height="75%" width="75%">
</p>

Make sure you at least have an index.html (that isn't empty preferably) object in your bucket.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/confirmhosting-003-make-sure.indexhtml.jpg" alt="Make sure you have an index.html in your site bucket." height="75%" width="75%">
</p>

4. Refresh your browser or click on the endpoint URL again like you did in step 2&mdash;you will get a new error message. The problem is that there is an index.html but it is not publicly accessible. Since you'll want visitors to be able to view your site, the files need to be public. Note that I could have configured the CloudFormation template to make the bucket public by default but I opted not to in order to avoid folks accidentally uploading sensitive files.:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/confirmhosting-004-access-denied.jpg" alt="Access denied when trying to open index.html." height="75%" width="75%">
</p>

5. Open the the __Permissions__ tab and and then click on __Bucket Policy__. From here you can paste in the following policy&mdash;make sure to replace `<YOUR BUCKET NAME>` with your bucket name:

```
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "AllowPublicRead",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::<YOUR BUCKET NAME>/*"
        }
    ]
}
```
<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/confirmhosting-005-add-public-access-policy.jpg" alt="Add a policy to allow public access." height="75%" width="75%">
</p>

6. Now when you refresh the site you should get your index.html file&mdash;in my case, **WORKS**. Here is a screenshot with the developer tools enabled. Notice that the server shows **S3** and the port is 80:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/confirmhosting-006-refresh-site-view-indexhtml.jpg" alt="Successfully load index.html." height="75%" width="75%">
</p>

## ADD CNAME TO CLOUDFLARE

Now that you confirmed that static web hosting is working, it's time to add the root and www S3 endpoints to Cloudflare as CNAME records.

1. Copy the URL of the site endpoint listed next to __SiteBucketEndpoint__ in the __Outputs__ section of the launched CloudFormation template __WITHOUT__ the `http://*` portion:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/addcname-001-add-root-cname-endpoint.jpg" alt="Add the site root CNAME endpoint." height="75%" width="75%">
</p>

2. Next do the same thing for the redirect bucket labelled __RedirectBucketEndpoint__:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/addcname-002-add-www-cname-endpoint.jpg" alt="Add the site www redirect CNAME endpoint." height="75%" width="75%">
</p>

3. You should now have two CNAME entries that reference your static web hosting enabled S3 buckets. Note that the orange clouds to the right can be toggled on and off&mdash;off (i.e. grey cloud) means that Cloudflare is just running DNS show none of the other features (e.g. CDN, WAF, redirects, etc.) will be applied:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/addcname-003-confirm-two-cname.jpg" alt="CLoudflare should now have two CNAME references to your S3 buckets." height="75%" width="75%">
</p>

4. Go to [whatsmydns.net](https://www.whatsmydns.net) to confirm that Cloudflare DNS is resolving your domain name. Note that you have to keep the A record setting because Cloudlfare uses [CNAME Flattening](https://support.cloudflare.com/hc/en-us/articles/200169056-CNAME-Flattening-RFC-compliant-support-for-CNAME-at-the-root) which presents CNAME's as A records.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/addcname-004-confirm-dns-propagation.jpg" alt="Use whatsmydns.net to confirm DNS propagation." height="75%" width="75%">
</p>

5. Your site should now open on it's domain name now instead of the S3 endpoint. Note that the server now shows __cloudflare-nginx__ If [whatsmydns.net](https://www.whatsmydns.net) is showing all green checks but your browser is not resolving the DNS name, clear out your DNS cache.

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/addcname-005-confirm-can-open-site.jpg" alt="Confirm that you can access your S3 static website using your domain name." height="75%" width="75%">
</p>

If the your website times out make sure that __Crypto__ section in Cloudflare is set to `Flexible`:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/addcname-005-confirm-crypto-set-to-flexible.jpg" alt="Make sure Cloudflare Crypto option is set to Flexible." height="75%" width="75%">
</p>

## SETUP CLOUDFLARE HTTPS REDIRECT

One last thing to do on the Cloudflare side is to setup a redirect so that your site always opens on HTTPS. Do this by going to the __Page Rules__ tab and setup a URL match and set it to `Always use HTTPS`. When you are done click the __Save and Deploy__ button:

```
http://example.com/*
```

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/setupredirect-000-configure-cloudlare-redirect.jpg" alt="Set Cloudflare page rule to redirect all HTTP requests to HTTPS." height="75%" width="75%">
</p>

Now when you access the site using `www.example.com` or `http://example.com` your site will redirect to `https://example.com`. Note that you would normally configure this setting on a web server like NGINX or APACHE but since AWS is managing the S3 static website web server this is one way to accomplish a redirect from HTTP to HTTPS:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/setupredirect-000-try-open-with-www.jpg" alt="Try opening your site using www." height="75%" width="75%">
</p>

Note that in my example the site redirects to `https://tutorialstuff.xyz` and there are no [HTTP status codes](http://www.restapitutorial.com/httpstatuscodes.html) such as `301 Moved Permanently` or `307 Temporary Redirect`:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/setupredirect-000-confirm-http-status-code.jpg" alt="Confirm the HTTP status code of 200 instead of 301 or 307." height="75%" width="75%">
</p>

## CONFIGURE CODECOMMIT USER

If you used the default settings when launching the stack, you should have a new IAM group and user. Navigate to your IAM console and confirm the group and user deployed by this stack. If you had any other IAM users that you want to be able to use CodeCommit for this website you could add them to this group:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/codecommitconfig-000-confirm-group-user.jpg" alt="Confirm IAM group and user." height="75%" width="75%">
</p>

Click on the __Permissions__ tab and notice the __Inline Policies__ section. A policy was created by this template and attached to the *group*, not the IAM user. Normally you would click on __Show Policy__ but for whatever reason CloudFormation generated policies display all on one-line. Argh. So click on __Edit Policy__ instead to get a better view of what permissions have been applied to the group:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/codecommitconfig-000-edit-policy-for-better-view.jpg" alt="Click on Edit Policy to get a better view of the applied policy created by the stack." height="75%" width="75%">
</p>

Here is the actual policy (with my AWS account number masked):

```
{
    "Statement": [
        {
            "Action": [
                "s3:DeleteObject",
                "s3:GetBucketAcl",
                "s3:GetBucketWebsite",
                "s3:GetObject",
                "s3:GetObjectAcl",
                "s3:GetObjectVersion",
                "s3:GetObjectVersionAcl",
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::tutorialstuff.xyz/*",
            "Effect": "Allow",
            "Sid": "S3Access"
        },
        {
            "Action": [
                "codecommit:BatchGetRepositories",
                "codecommit:CreateBranch",
                "codecommit:Get*",
                "codecommit:GitPull",
                "codecommit:GitPush",
                "codecommit:List*",
                "codecommit:Put*",
                "codecommit:Test*",
                "codecommit:Update*"
            ],
            "Resource": [
                "arn:aws:codecommit:us-west-2:XXXXXXXXXXXX:tutorialstuff.xyz"
            ],
            "Effect": "Allow",
            "Sid": "CodeCommitAccess"
        }
    ]
}
```

This policy will allow any IAM user in the group have the required permissions to manage the static website's S3 bucket and CodeCommit repository.

Now that you understand what the group does it's time to make the CodeCommmit user configuration. AWS has good documentation on how to configure the IAM user for CodeCommit [here](http://docs.aws.amazon.com/codecommit/latest/userguide/setting-up.html) but I will demonstrate the way I like to do it.

1. Create a key pair using `ssh-keygen`like the OS X command line example below. If you haven't done this before get your path with `pwd` so you know exactly where to save your public and private key pair. I like to use a bit size of `4096` but choose what you are comfortable with. For the name of the key pair use whatever make sense to you but in this example I will use the actual IAM user name of `tutorialstuff.xyz-CodeCommitUser-us-west-2`. Finally, I always use passwords on my SSH keys and I recommend you do the same but make sure you don't store the passphrase with the private key as that will defeat the purpose:

```
> pwd
/Users/virtualjj/Documents/AWS/SSH Keys/tutorialstuff.xyz
> ssh-keygen -b 4096
Generating public/private rsa key pair.
> Enter file in which to save the key (/Users/virtualjj/.ssh/id_rsa): /Users/virtualjj/Documents/AWS/SSH Keys/tutorialstuff.xyz/tutorialstuff.xyz-CodeCommitUser-us-west-2
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```
<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/codecommitconfig-001-create-4096-keypair.jpg" alt="Create a 4096 bit passphrase protected key pair." height="75%" width="75%">
</p>

View the permissions of the generated public (.pub) and private key pair. Change them to __read-only__ using the `chmod` command:

```
Demo: ls -la
total 16
drwxr-xr-x   4 virtualjj  staff   136 Aug  8 11:18 .
drwxr-xr-x  10 virtualjj  staff   340 Aug  8 11:05 ..
-rw-------   1 virtualjj  staff  3326 Aug  8 11:18 tutorialstuff.xyz-CodeCommitUser-us-west-2
-rw-r--r--   1 virtualjj  staff   745 Aug  8 11:18 tutorialstuff.xyz-CodeCommitUser-us-west-2.pub
Demo: chmod 400 *
Demo: ls -la
total 16
drwxr-xr-x   4 virtualjj  staff   136 Aug  8 11:18 .
drwxr-xr-x  10 virtualjj  staff   340 Aug  8 11:05 ..
-r--------   1 virtualjj  staff  3326 Aug  8 11:18 tutorialstuff.xyz-CodeCommitUser-us-west-2
-r--------   1 virtualjj  staff   745 Aug  8 11:18 tutorialstuff.xyz-CodeCommitUser-us-west-2.pub
```

2. Next click on the IAM user created by this stack and click on the __Security Credentials__ tab. Scroll down and select the __Upload SSH public key__ button:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/codecommitconfig-002-upload-codecommit-keypair.jpg" alt="Upload your CodeCommit public key." height="75%" width="75%">
</p>

3. On OS X I like to use a command called `pbcopy` to copy the contents of a file to my clipboard. You can copy the public key to your clipboard like this&mdash;make sure you copy the file with the extension of __.pub__:

```
pbcopy < tutorialstuff.xyz-CodeCommitUser-us-west-2.pub
```
<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/codecommitconfig-003-copy-public-key-to-clipboard.jpg" alt="Use pbcopy to copy public key to clipboard and upload to IAM.="75%" width="75%">
</p>

You should now have an uploaded CodeCommit public SSH key:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/codecommitconfig-003-confirm-uploaded-cc-ssh-key.jpg" alt="Confirm that your CodeCommit SSH public key has been successfully uploaded.="75%" width="75%">
</p>

## SETUP HUGO WEBSITE EXAMPLE

Now that your CodeCommit user has been setup with an SSH key you need to initialize a local repo with `git`. When we performed the steps in [CONFIRM STATIC HOSTING WORKS](#confirm-static-hosting-works) we used a simple index.html file. You will probably be using a static generator like [Hugo](https://gohugo.io/getting-started/) or [Jekyll](https://jekyllrb.com/) which has a lot of files to track.

1. As an example, I will setup a new Hugo site on my local OS X machine. If you don't have Hugo but want to try it you can follow the [Quick Start](https://gohugo.io/getting-started/quick-start/). Here are the commands to check the Hugo version, create a new site, change directory into it, and confirm your working path:

```
Demo: hugo version
Hugo Static Site Generator v0.25.1 darwin/amd64 BuildDate: 2017-07-13T00:40:37+09:00
Demo: hugo new site s3-tutorialstuff.xyz
Congratulations! Your new Hugo site is created in /Users/virtualjj/Documents/WEBSITES/s3-tutorialstuff.xyz.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/, or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
Demo: cd s3-tutorialstuff.xyz/
Demo: pwd
/Users/virtualjj/Documents/WEBSITES/s3-tutorialstuff.xyz
```
2. Next list the contents of the `themes` directory and change directory into it:

```
Demo: ls -la themes/
total 0
drwxr-xr-x  2 virtualjj  staff   68 Aug  8 11:49 .
drwxr-xr-x  9 virtualjj  staff  306 Aug  8 11:49 ..
Demo: cd themes/
Demo: pwd
/Users/virtualjj/Documents/WEBSITES/s3-tutorialstuff.xyz/themes
```
3. I will clone the [Aerial theme](https://themes.gohugo.io/aerial/) by [Seth MacLeod](https://www.sethmacleod.com/) and use it as an example:

```
git clone git clone https://github.com/sethmacleod/aerial.git
```

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/setuphugosite-003-download-theme.jpg" alt="Clone the Hugo Aerial theme to the local themes directory.="75%" width="75%">
</p>

4. Change directory back to the root of the local Hugo site you just created. Copy the Hugo theme's `exampleSite` __config.toml__ to your site's root folder. This file is what configures settings for Hugo and your theme. When you create a new Hugo site locally a config.toml file exists but it won't have the parameters necessary for the theme. The example below shows how I copied and overwrote the default __config.toml__:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/setuphugosite-004-overwrite-config-toml-with-theme-one.jpg" alt="Overwrite the default Hugo new site config.toml with the theme's config.toml configuration file.="75%" width="75%">
</p>

5. Open the config.toml file that you just copied in your favorite text editor. We need to change:

```
languageCode = "en-us"
title = "Aerial"
baseurl = "http://example.org/"
theme = "aerial"
```

To this&mdash;specifically the `baseurl`. If you don't change the `baseurl` to your domain name the theme links will break and not work properly when you upload the generated site to S3:

```
languageCode = "en-us"
title = "Aerial"
baseurl = "https://tutoriastuff.xyz/"
theme = "aerial"
```

6. Use the following command to run the site locally to confirm that the theme works. Open it on `localhost:1313`

```
hugo server --theme=aerial
```
<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/setuphugosite-006-generate-site-locally.jpg" alt="Generate Hugo site locally to confirm that the theme works.="75%" width="75%">
</p>

Open up a new tab and go to `localhost:1313`. The local website and theme should display:

<p align="center">
<img src="https://github.com/virtualjj/aws-s3-backed-cloudflare-static-website/blob/master/images/readme/setuphugosite-006-view-local-hugo-site-and-theme.jpg" alt="View the Hugo site locally and make sure the theme is working.="75%" width="75%">
</p>

7. Press `Ctrl+C` to cancel the static website.

## CONFIGURE CODECOMMIT GIT REPO






## ACKNOWLEDGMENTS

Thanks to Eric Hammond for inspiring me to work on this project. Much of his [alestic/aws-git-backed-static-website](https://github.com/alestic/aws-git-backed-static-website) template was used as the basis for this project.
