# Production-Ready Static Website on AWS with Cloudflare

A production-style static website deployment demonstrating how to combine **Amazon S3**, **Amazon CloudFront**, **AWS Certificate Manager (ACM)**, and **Cloudflare** to deliver a secure, highly available, and globally cached website.

## Architecture

```text
                         ┌──────────────────┐
                         │     Internet     │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │    Cloudflare    │
                         │       DNS        │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │  Amazon          │
                         │  CloudFront      │
                         │  CDN + HTTPS     │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │   Amazon S3      │
                         │ Static Website   │
                         │     Hosting      │
                         └──────────────────┘
```

## Project Objectives

The goal of this project was to deploy a static website using AWS services while applying production-oriented DevOps practices.

The deployment provides:

* Static website hosting with Amazon S3
* Global content delivery through CloudFront
* HTTPS using AWS Certificate Manager
* DNS management through Cloudflare
* CloudFront caching and compression
* CloudFront cache invalidation after deployments

## AWS Services Used

| Service                     | Purpose                                                |
| --------------------------- | ------------------------------------------------------ |
| **Amazon S3**               | Stores and serves the static website files             |
| **Amazon CloudFront**       | Global CDN, caching, compression, and HTTPS delivery   |
| **AWS Certificate Manager** | Provides the SSL/TLS certificate for the custom domain |
| **Cloudflare**              | DNS and domain management                              |

## Deployment Architecture

### 1. Amazon S3

An S3 bucket was created to host the static website.

The bucket contains:

```text
index.html
error.html
```

<p align="left"> <img width="1000" src="./Images/files uploaded s3.png"> </p>

Static website hosting was enabled, allowing S3 to serve the website content.

<p align="left"> <img width="1000" src="./Images/s3 static web hosting enabled.png"> </p>

The website's static assets are stored in S3 while CloudFront acts as the public distribution layer.

### 2. Amazon CloudFront

CloudFront was configured as the CDN in front of the S3 website.

Configuration included:

* S3 website endpoint as the origin
* HTTPS enabled
* Object compression enabled
* Managed **CachingOptimized** cache policy
* `GET` and `HEAD` methods allowed
* Custom domain configured
* ACM SSL/TLS certificate attached

<p align="left"> <img width="1000" src="./Images/cloudfront distribution created.png"> </p>

<p align="left"> <img width="1000" src="./Images/cloudfront distribution behaviour settings.png"> </p>

<p align="left"> <img width="1000" src="./Images/cloudfront distribution settings updated with cert.png"> </p>


CloudFront provides global edge caching so users can retrieve content from an edge location closer to them rather than directly from the S3 origin.

### 3. AWS Certificate Manager

An ACM certificate was created for the custom domain and associated with the CloudFront distribution.

This enables HTTPS access to the website.

For CloudFront, the ACM certificate must be created in the **US East (N. Virginia) / `us-east-1`** region.

### 4. Cloudflare DNS

Instead of Route 53, **Cloudflare** was used to manage the domain's DNS records owing to the fact it was already managed.

The domain was configured to resolve to the CloudFront distribution.

The request flow is therefore:

```text
User
  │
  ▼
Cloudflare DNS
  │
  ▼
CloudFront
  │
  ▼
S3
```

Cloudflare handles DNS resolution while CloudFront handles CDN delivery, caching, and HTTPS.

## Caching Demonstration

CloudFront caching was tested by modifying `index.html`.

Because CloudFront caches objects at its edge locations, changes to the S3 object may not immediately appear to users.

To immediately distribute an updated version, a CloudFront invalidation was created.

Example:

```text
Update index.html
       ↓
Upload to S3
       ↓
Create CloudFront invalidation
       ↓
CloudFront removes cached object
       ↓
User receives updated content
```

This demonstrates an important consideration when deploying static websites behind a CDN: **the origin can be updated without the cached version immediately changing at the edge.**


## Security Considerations

The project also demonstrates several production-oriented security concepts:

* HTTPS for encrypted client connections
* TLS certificate management through ACM
* CloudFront used as the public delivery layer
* DNS managed through Cloudflare
* AWS credentials stored as GitHub Actions secrets rather than committed to the repository
* CDN caching to reduce direct load on the S3 origin

## What I Learned

This project demonstrated how a relatively simple static website can be deployed using a production-style architecture rather than being served directly from a single server.

The key architecture principle is:

> **S3 provides the storage/origin, CloudFront provides global delivery and HTTPS, and Cloudflare provides DNS management.**

The addition of GitHub Actions turns the infrastructure into an automated deployment workflow, allowing changes pushed to the repository to be automatically deployed and distributed through CloudFront.

## Future Improvements

Potential improvements include:

* Add security headers using CloudFront Functions
* Implement stricter S3 bucket access controls
* Add cache-control headers for different asset types
* Add Infrastructure as Code using Terraform
* Add a CI/CD pipeline using GitHub Actions → automatic deploy to S3
* Implement separate development and production environments
* Add monitoring and alerting
* Add Lambda@Edge to rewrite URLs
* Add Cloudflare security and caching features where appropriate

