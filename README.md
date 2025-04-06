# Pulumi_Static_Website_Deployment

Below is a GitHub README for your *Fast Static Website Deployment with Pulumi, S3, and CloudFront* project. It’s concise, professional, and includes setup instructions, usage, and a bit of flair to make it welcoming. Assumes you’ll push this to a repo named `fast-static-site`—adjust as needed.

---

## Fast Static Site 🚀

Deploy a blazing-fast static website to AWS using Pulumi, S3, and CloudFront. This project sets up a scalable, secure site with minimal effort—perfect for portfolios, landing pages, or any static content needing speed and simplicity.

## Features

- **Amazon S3**: Stores your static files (HTML, CSS, JS) with static hosting enabled.
- **CloudFront CDN**: Delivers content globally with low latency and HTTPS by default.
- **Pulumi + TypeScript**: Infrastructure as code for repeatable, one-command deploys.
- **Lightweight**: No servers, no fuss—just fast static goodness.

## Prerequisites

- [Node.js](https://nodejs.org/) (v18+ recommended)
- [Pulumi CLI](https://www.pulumi.com/docs/install/) (`npm install -g @pulumi/cli`)
- [AWS CLI](https://aws.amazon.com/cli/) configured with credentials (`aws configure`)
- An AWS account with IAM permissions for S3 and CloudFront

## Getting Started

### 1. Clone the Repo

First thing we have to do is configure access through IAM
then create a EC2 on AWS
[https://www.pulumi.com/docs/iac/clouds/aws/](https://www.pulumi.com/docs/iac/clouds/aws/)
[https://www.pulumi.com/docs/iac/get-started/aws/](https://www.pulumi.com/docs/iac/get-started/aws/)

```bash
curl -fsSL https://get.pulumi.com | sh
```

Then lets check if its been installed

```bash
pulumi version
```

It threw an error so we have to add "/home/ec2-user/.pulumi/bin:$PATH" to the PATH
by running the command to do so

```bash
export PATH="/home/ec2-user/.pulumi/bin:$PATH"
```

once we do this we can run Pulumi version again

```bash
pulumi version
```

and we will get back the version `v3.160.0` showing it is now installed correctly

then install Node js
[https://nodejs.org/en/download](https://nodejs.org/en/download)

```bash
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.2/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 22

# Verify the Node.js version:
node -v # Should print "v22.14.0".
nvm current # Should print "v22.14.0".

# Verify npm version:
npm -v # Should print "10.9.2".
```

then setup IAM programmatic access
[https://www.pulumi.com/docs/iac/get-started/aws/begin/](https://www.pulumi.com/docs/iac/get-started/aws/begin/)

```bash
export AWS_ACCESS_KEY_ID="<YOUR_ACCESS_KEY_ID>" &&
export AWS_SECRET_ACCESS_KEY="<YOUR_SECRET_ACCESS_KEY>"
```

then create a new project
[https://www.pulumi.com/docs/iac/get-started/aws/create-project/](https://www.pulumi.com/docs/iac/get-started/aws/create-project/)

we will use typescript for this project

```bash
mkdir quickstart && cd quickstart && pulumi new aws-typescript
```

it may ask to login to Pulumi Cloud if this is your first time just make sure you are logged in to your account so you can create the Token then
enter pulumi access token
then it will go through the prompts to create the project
project name => pul-challenge-aws
project description
stack name => org-name/dev
The package manager to use for installing dependencies => npm
The AWS region to deploy into (aws:region) (us-east-1)
Saved config
Installing dependencies...
npm notice To update run: npm install -g npm@11.2.0

```bash
npm install -g npm@11.2.0
```

these files will be created
Pulumi.dev.yaml  
Pulumi.yaml  
index.ts  
node_modules  
package-lock.json  
package.json  
tsconfig.json

this will give you files you need for review
[https://www.pulumi.com/docs/iac/get-started/aws/review-project/](https://www.pulumi.com/docs/iac/get-started/aws/review-project/)

```bash
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";
import * as awsx from "@pulumi/awsx";

// Create an AWS resource (S3 Bucket)
const bucket = new aws.s3.BucketV2("my-bucket");

// Export the name of the bucket
export const bucketName = bucket.id;
```

then we will deploy our stack which will provision it
[https://www.pulumi.com/docs/iac/get-started/aws/deploy-stack/](https://www.pulumi.com/docs/iac/get-started/aws/deploy-stack/)

```bash
pulumi up
```

takes a little while for it to build

then we will run the command that will print out the name of the bucket so we know we re up and running

```bash
pulumi stack output bucketName
```

then we can modify the program as we need
[https://www.pulumi.com/docs/iac/get-started/aws/modify-program/](https://www.pulumi.com/docs/iac/get-started/aws/modify-program/)

then redeploy the changes
[https://www.pulumi.com/docs/iac/get-started/aws/deploy-changes/](https://www.pulumi.com/docs/iac/get-started/aws/deploy-changes/)

once the update is complete we can verify the object was created
[https://www.pulumi.com/docs/iac/get-started/aws/deploy-changes/](https://www.pulumi.com/docs/iac/get-started/aws/deploy-changes/)

```bash
aws s3 ls $(pulumi stack output bucketName)
```

now we can add a new website configuration

```bash
const website = new aws.s3.BucketWebsiteConfigurationV2("website", {
    bucket: bucket.id,
    indexDocument: {
        suffix: "index.html",
    },
});
```

then make a few adjustments to make these resources accessible on the Internet.

```bash
const ownershipControls = new aws.s3.BucketOwnershipControls("ownership-controls", {
    bucket: bucket.id,
    rule: {
        objectOwnership: "ObjectWriter"
    }
});

const publicAccessBlock = new aws.s3.BucketPublicAccessBlock("public-access-block", {
    bucket: bucket.id,
    blockPublicAcls: false,
});

const bucketObject = new aws.s3.BucketObject("index.html", {
    bucket: bucket.id,
    source: new pulumi.asset.FileAsset("./index.html"),
    contentType: "text/html",
    acl: "public-read",
}, { dependsOn: [publicAccessBlock,ownershipControls,website] });
```

at the end of the program, export the resulting bucket’s endpoint URL so you can browse to it easily:

```bash
export const bucketEndpoint = pulumi.interpolate`http://${website.websiteEndpoint}`;
```

then redeploy

```bash
pulumi up
```

Choose `yes` to perform the deployment:

When the deployment completes, you can check out your new website at the URL in the [`Outputs`](https://www.pulumi.com/docs/iac/concepts/inputs-outputs/#outputs) section of your update or make a `curl` request and see the contents of `index.html` in your terminal:

```bash
curl $(pulumi stack output bucketEndpoint)
```

Next, you’ll destroy the resources.
[https://www.pulumi.com/docs/iac/get-started/aws/destroy-stack/](https://www.pulumi.com/docs/iac/get-started/aws/destroy-stack/)

```bash
pulumi destroy
```

and To delete the stack itself, run

```bash
pulumi stack rm
```
