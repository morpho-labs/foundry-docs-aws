# 📖🛠️ Foundry Docs Generator

- Automatically update your AWS-hosted smart contract documentation, by running this action in a CI on each of your Pull Requests!

## Getting started

### Automatically generate docs & upload them on every PR

Add a workflow (`.github/workflows/foundry-docs-aws.yml`):

```yaml
name: Generate docs

on:
  push:
    branches:
      - main
  pull_request:
    # Optionally configure to run only for changes in specific files. For example:
    # paths:
    # - src/**
    # - foundry.toml
    # - remappings.txt
    # - .github/workflows/foundry-docs-aws.yml

jobs:
  forge-docs:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
        with:
          submodules: recursive

      - name: Generate & upload forge docs
        uses: morpho-labs/foundry-docs-aws@v1
        with:
          aws-s3-bucket: forge-docs
          aws-cloudfront-distribution-id: ${{ secrets.AWS_CLOUDFRONT_DISTRIBUTION_ID }} # optionally invalidate the Cloudfront cache on each upload
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
          aws-region: ${{ secrets.AWS_REGION }}
```

---

## How it works

Everytime somebody opens a Pull Request, the action runs [Foundry](https://github.com/foundry-rs/foundry) `forge` to generate automated documentation based on the NATSPECs of your contracts.


## AWS IAM Credentials

Your credentials must have s3 sync autorization attached. The minimum policies to have are setted in the following `policy.json` file:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "S3SyncPermission",
            "Effect": "Allow",
            "Action": [
                "s3:DeleteObject",
                "s3:GetBucketLocation",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:ListObjectsV2"
            ],
            "Resource": [
                "arn:aws:s3:::<your-bucket>",
                "arn:aws:s3:::<your-bucket>/*"
            ]
        },
        {
            "Sid": "ListBucketsPermission",
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets"
            ],
            "Resource": "*"
        }
    ]
}
```
As option, if you are distributing the documentation through CloudFront you can give authorization to invalidate the cache in order to reflect changes directly:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CleanCachePermission",
            "Effect": "Allow",
            "Action": [
                "cloudfront:GetDistribution",
                "cloudfront:ListInvalidations",
                "cloudfront:GetInvalidation",
                "cloudfront:CreateInvalidation"
            ],
            "Resource": "<distribution-arn>"
        }
    ]
}
```
