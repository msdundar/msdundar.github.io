---
title: "Canary AWS Lambda Deployments"
description: "Gradual/Canary deployments with AWS Lambda explained."
date: 2021-06-14T22:50:45+02:00
categories:
- tech
tags:
- aws
- lambda
cover:
    image: "posts/canary-aws-lambda-deployments/assets/la-campagne-nivernaise-1873-johan-barthold-jongkind.jpg"
    alt: "La Campagne Nivernaise (1873) - Johan Barthold Jongkind"
    relative: false
images: ["assets/la-campagne-nivernaise-1873-johan-barthold-jongkind.jpg"]
---

Lambdas aren't easy. This isn't just a provocative start, but instead my overall
experience planning, creating, and deploying them.

Let's be honest, making something up and running requires plenty of AWS
knowledge. One might get lost easily even inside IAM alone.

IAM groups, IAM users, IAM roles, IAM group policy attachments,  IAM policy
documents, IAM role policies - and how they connect is confusing enough
considering what Lambda promises, simplicity. And this is just IAM.

Nowadays no infrastructure is simple. If you're old enough to remember days
when developers used to drag and drop files in an FTP agent to deploy something,
things have changed a long time ago.

The rationale behind the change is not tools and it's not Lambda to blame, it's
the scale. Even the simplest solutions, such as Lambda and S3, have designed to
scale, and when you start working on things that can scale, they get complicated
quickly.

Think about the most basic Lambda + API Gateway + DynamoDB setup you can
imagine. You will still need to learn a lot about AWS. Most likely you're going
to need an average understanding of six different services, at minimum. Lambda,
API Gateway, S3, IAM, DynamoDB, CloudWatch and so on.

The number of development practices we follow has also evolved rapidly with
the growing scale. Nowadays almost any software project has at least two
environments (production, staging etc.), projects often deployed on multiple
geographical locations, continuous integration and continuous deployment are
everyday tools and deployment choices are rich, starting from manual
deployments to semi-automated and fully-automated. Logging, monitoring and
incident management are essential stages that need to be considered for each
service. If you think about all the pieces one needs to combine to get a simple
setup up and running and to reflect all these pieces on Terraform, or in any
other IAC tool, the complexity can't be underestimated.

## Use Case

I started to investigate available Lambda deployment strategies while working
on a hobby project. I've been using Lambdas for more than two years, but
deployment internals has always been a mystery to me, as I was lucky to join a
company with rich internal tooling is already available. However, of course,
I knew about my options, such as [SAM](https://aws.amazon.com/serverless/sam/),
[Serverless Framework](https://www.serverless.com/),
[Terraform](https://www.terraform.io/) and
[AWS SDKs](https://aws.amazon.com/getting-started/tools-sdks/).

At Babbel, the deployment of a Lambda is hassle-free, thanks to the internal
tooling available. One can easily set up continuous deployment or choose
to follow a semi-automated deployment strategy within a couple of hours with
tools already available for this purpose.

However, without tools to deal with deployment, it's like Wild West out there.
In this article, I will summarize some common deployment practices that I came
across during my research. But first of all, here is the infrastructure setup
I have:

- Github repositories managed by Terraform
- An AWS account fully managed by Terraform
- Two identical (resource-wise) environments for each AWS services, `production`
  and `staging`.
- IAM setup is near perfect, no services can do more than they supposed to.
  Service accesses, roles, policies and permissions are finely tuned.
- Log groups are organized clearly. Alarms and budgets set up for each service.
- A simple DynamoDB database to store names, such as "John Doe"
- An S3 bucket with versioning enabled to store lambda packages as .zip files.
- An API Gateway with a simple GET endpoint that is used as an event source for
  the lambda.
- Github Actions (CI) is enabled and it takes care of packing lambda functions
  and uploading them to the S3 bucket.
- The lambda function has an extensive test suite and tested both locally and
  on CI.

Here is how my demo system works:

- An HTTP GET request together with a query string parameter is sent to the API
  Gateway, i.e.: https://foo.com/names?name=John
- API Gateway triggers the lambda function. Lambda function queries the name
  in DynamoDB, and if a matching record has found in the database it returns
  `200` and a simple JSON in the body, otherwise, it returns `404` and an empty
  body.

## The Next Step

Briefly, I have a setup that I can test on AWS or locally. My Github Actions
setup can pack the lambda and upload it to the S3 bucket. Now it's time to
decide on continuous deployment!

These are my acceptance criteria for Lambda deployments:

- CI should take care of lambda aliases and versioning.
- An IAM user dedicated to CI should only have minimum possible permissions, and
  should only access resources it actually manages.
- Changes reflected in `main` should be deployed to `production` lambda.
- Changes reflected in `develop` should be deployed to `staging` lambda.
- Deployments should be _gradual_ on `production`.
- Deployments should be _all at once_ on `staging`.
- CI should auto-rollback the deployment if a failure happens during the gradual
  deployment.
- CI should implement a health-check mechanism to see if the gradual deployment
  is successful.
- CI should take care of full roll-out when gradual deployment steps are
  successful.
- All CI actions should be runnable on local development environment without
  hassle.

With these criteria in mind, I started researching my options and best-practices
out there.

## Deployments with Terraform

I've listed 3 different approaches here. However, deployments with Terraform
violates some criteria I've listed above. First of all, resources should be
initialized and configured with Terraform, but Terraform shouldn't take care
of internal changes of them, according to the ideal infrastructure in my mind.
For example, Terraform should bring a lambda to life, configure its resources
such as memory and timeout, but shouldn't care about the code changes of a
lambda.

It makes sense, right? If not, think about S3. Terraform creates buckets,
defined policies and lifecycles for them, but doesn't care about what you put
into those buckets. With a similar point of view, I didn't prefer Terraform
deployments for the following reasons:

- It just doesn't feel right.
- Running a full `terraform apply` on CI requires an IAM user with extensive
  permissions.
- When multiple developers modify resources, keeping terraform `tfstate` in sync
  is a pain.

However, I will still list approaches I came across while exploring this option:

### Approach 1: Keep a .version file in your github repository

This option is about keeping a `.version` file in the code repository located on
Github to track changes on the code. Deployments are going to be possible after
running `terraform apply` either locally or on CI.

Initialize the file with Terraform:

```hcl
resource "github_repository_file" "name_searcher_lambda_version" {
  repository          = github_repository.name_searcher_lambda.name
  branch              = "main"
  file                = ".version"
  content             = "v0.0.1"
  commit_message      = "Initialize the version file"
  commit_author       = "Someone"
  commit_email        = "something@gmail.com"
}
```

Add `source_code_hash` key to your lambda:

```hcl
resource "aws_lambda_function" "name_searcher" {
  function_name = "name-searcher"

  handler     = "main"
  runtime     = "go1.x"
  memory_size = 128
  timeout     = 5

  s3_bucket = aws_s3_bucket.lambda_sources[each.key].bucket
  s3_key    = "${github_repository.name_searcher_lambda.name}/index.zip"

  source_code_hash = github_repository_file.name_searcher_lambda_version.commit_sha
}
```

In this scenario, the lambda function is going to use the `commit_sha` of the
`.version` file as `source_code_hash`. `.version` file needs to be updated
whenever you would like to publish the lambda again. After the `commit_sha` has
modified, you also need to run `terraform apply` to re-deploy the lambda from
the S3 bucket.

I didn't like this approach at all.

### Approach 2: Store lambda packages in version-aware subfolders on S3

This approach is the one used in the official
[Terraform documentation](https://learn.hashicorp.com/tutorials/terraform/lambda-api-gateway)
for lambdas.

In this approach, you need to upload your lambda packages under versioned
subfolders as follows:

```
s3://lambda-sources/name-searcher/1.1.0/index.zip
```

Then add the following to the `variables.tf` to get version number as a
variable:

```hcl
variable "lambda_version" {}
```

Finally, add the `lambda_version` to the `s3_key`:

```hcl
s3_key = "v${var.lambda_version}/example.zip"
```

Run `terraform_apply` with the `lambda_version` variable to deploy changes:

```sh
terraform apply -var lambda_version="1.0.1"
```

For rolling back the deployment with an older version, follow the same logic:

```sh
terraform apply -var lambda_version="0.0.5"
```

In this approach, you still need to take care of version numbers either locally
or in your CI. Again, this approach requires too much manual intervention and
violates my acceptance criteria.

### Approach 3: Upload .zip SHA as plaintext to S3

I've seen this approach in [John Roach's blog](https://johnroach.io/2020/09/04/deploying-lambda-functions-with-terraform-just-dont/).
It's similar to the first two approaches but this one takes actual source code
changes into consideration by getting the SHA of the whole deployment package for
change tracking purposes.

In this approach, you need to upload a plaintext file including the SHA of the
deployment package to the S3 bucket together with your .zip file. In the end,
there will be two files in your S3 bucket for a lambda:

```
lambda_package.zip # lambda deployment package
lambda_package.zip.sha256 # plaintext file including SHA of the lambda_package.zip
```

Then use this SHA as `source_code_hash` in your lambda:

```hcl
data "aws_s3_bucket_object" "name_searcher_hash" {
  bucket = "lambda-sources"
  key    = "path/name_searcher_payload.zip.base64sha256"
}

resource "aws_lambda_function" "name_searcher" {
  source_code_hash = data.aws_s3_bucket_object.name_searcher_hash.body
  s3_bucket        = "lambda-sources"
  s3_key           = "path/lambda_function_payload.zip"
}
```

The author of this approach prefers to ignore `source_code_hash` changes in
lifecycle rules (which makes sense) and re-deploys the lambda function as
follows:

```sh
aws lambda update-function-code --function-name name_searcher \
                                --s3-bucket lambda-sources \
                                --s3-key path/lambda_function_payload.zip
```

This approach can also be combined with the first two approaches. You can treat
commit SHAs similar to versions, and vice versa, it all depends on your choices.
I only provided a modified brief version of this approach, so visit the original
post for more details.

## Deployments with SAM

SAM specification is great to make things up and running quickly, especially if
your infrastructure isn't complex. For the use case I mentioned earlier SAM
could be an option, however, there are things I don't like about SAM:

- It just doesn't feel right when used together with Terraform.
- SAM doesn't support some AWS services such as IAM Role, IAM Policy, KMS etc.
  This forces infrastructure to split between Terraform and SAM, and I want a
  single source of truth, Terraform.
- SAM makes more sense and provides convenience especially when used together
  with other services such as API Gateway, however, I prefer to keep them in
  Terraform.
- It's a little bit magical! I don't feel comfortable SAM creating IAM roles
  and permission automatically for me.
- Programmatic access and structuring in SAM templates are ugly.
- Plenty of IAM permissions needs to be granted to run `sam deploy` on CI that
  again I prefer to avoid.

Because of the listed reasons, I decided not to evaluate SAM as an option.

## Deployments with Serverless

Serverless and SAM are somehow similar, both are CloudFormation abstractions,
and both generate CloudFormation templates. For the use case I mentioned,
Serverless again didn't feel right for the same reasons I've listed for SAM.

## Deployments with AWS SDK

AWS SDK is at the core of AWS services. The amount of functionality they provide
enables developers to build their custom solutions on top of what is already
available. Therefire, I've adopted AWS SDK and built my own deployment workflow
satisfying all acceptance criteria I've defined. Here are the details:

An IAM user for CI usage with policies (`aws_iam_policy_document`) including
minimum possible permissions:

```tf
# Only for the bucket that CI can upload
"s3:Get*"
"s3:List*"
"s3:PutObject"
"s3:PutObjectAcl"

# Only for the lambda that CI can deploy
"lambda:GetFunction"
"lambda:GetFunctionConfiguration"
"lambda:InvokeFunction"
"lambda:UpdateFunctionCode"
"lambda:PublishVersion"
"lambda:UpdateAlias"
```

Credentials of this IAM user are being added to the Github Repository of lambda
as `secrets` by Terraform. The IAM user of CI can only upload files to the S3
`lambda_sources` S3 bucket, and can't access others. Similarly, it can only
deploy the related lambda, not others.

There is an alias that is going to be exposed for client usage, named `live`.
The ARN of the alias I've created is the one to be triggered from API Gateway:

```tf
resource "aws_lambda_function" "name_searcher" {
  for_each = var.environments

  function_name = "name-searcher-${each.key}"
  description   = "Queries the harmful-domains DynamoDB table"

  handler     = "main"
  runtime     = "go1.x"
  memory_size = 128
  timeout     = 5

  s3_bucket = aws_s3_bucket.lambda_sources[each.key].bucket
  s3_key    = "${github_repository.name_searcher_lambda.name}/v1.0.0/index.zip"

  role = aws_iam_role.lambda_name_searcher.arn
}

resource "aws_lambda_alias" "name_searcher" {
  for_each = var.environments

  name             = "live"
  description      = "Live version of the lambda. Gradual traffic shifting in place."
  function_name    = aws_lambda_function.name_searcher[each.key].arn
  function_version = "$LATEST"

  lifecycle {
    ignore_changes = [function_version]
  }
}
```

`ignore_changes` part here is important because I want CI to manage releases.

The rest is pretty standard. An S3 bucket, some Terraform config for managing
the Github repository and so on.

After creating the initial infrastructure I've decided to create a Go module
to take care of canary Lambda deployments. Therefore, I've created a module
around AWS SDK and integrated into the CI setup of all my personal lambda
projects.

Here are you can have a look:

{{< figure src="kanarya-sm.png" link="https://github.com/msdundar/kanarya" attr= "https://github.com/msdundar/kanarya" attrlink= "https://github.com/msdundar/kanarya" >}}


## References

- https://learn.hashicorp.com/tutorials/terraform/lambda-api-gateway
- https://johnroach.io/2020/09/04/deploying-lambda-functions-with-terraform-just-dont/
- https://medium.com/galvanize/aws-lambda-deployment-with-terraform-24d36cc86533
- https://docs.aws.amazon.com/sdk-for-go/api/service/lambda/
