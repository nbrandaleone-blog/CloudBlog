---
title: "Running Hugo in the Cloud"
date: 2020-12-23T08:50:38-05:00
---

## Jekyll or Hugo
Let's face it, _Hugo_ is where all the cools kids are. My old blog worked just fine, and _Jekyll_ is really a wonderful static site generator. That beind said, _Hugo_ **is** faster.  Also, being written in **GO** allows for cleaner package management and improved portability.

## Running Hugo on the Cloud
My old blog ran out of a S3 bucket as a static web site, because it was inexpensive and easy to set up.  I just did the same thing on [GCP](https://cloud.google.com/storage/docs/hosting-static-website).  However, this might not be my long term architecture since GCP also requires a [Cloud Load Balancer](https://cloud.google.com/load-balancing) to redirect traffic to the bucket.  Load Balancers are expensive, and I do not really want to shell out more than $5/month for this blog.  I am going to look into [Cloud Run](https://cloud.google.com/run) in the future, since as a serverless/container platform it has the potential to be much less expensive.

### Guides to running Hugo on AWS and GCP
- [How to use S3 for static website hosting](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/static-website-hosting.html)
- [Deploying a Hugo Site to AWS Lightsail Containers](https://www.jeremymorgan.com/tutorials/golang/how-to-deploy-hugo-aws-lightsail-containers/)
- [Using Google buckets for static website hosting](https://cloud.google.com/storage/docs/hosting-static-website)
- [Running Hugo from a Google Storage bucket](https://pretired.dazwilkin.com/posts/200930/)
