# Production-Ready Static Website on AWS with Cloudflare

A production-style static website deployment demonstrating how to combine **Amazon S3**, **Amazon CloudFront**, **AWS Certificate Manager (ACM)**, **Cloudflare**, and **GitHub Actions** to deliver a secure, highly available, and globally cached website.

> **Note:** Cloudflare was used for DNS and domain management instead of Amazon Route 53.

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
* Automated deployments using GitHub Actions
* CloudFront cache invalidation after deployments

## AWS Services Used

| Service                     | Purpose                                                |
| --------------------------- | ------------------------------------------------------ |
| **Amazon S3**               | Stores and serves the static website files             |
| **Amazon CloudFront**       | Global CDN, caching, compression, and HTTPS delivery   |
| **AWS Certificate Manager** | Provides the SSL/TLS certificate for the custom domain |
| **Cloudflare**              | DNS and domain management                              |
| **GitHub Actions**          | Automates website deployments to S3                    |

## Deployment Architecture

### 1. Amazon S3

An S3 bucket was created to host the static website.

The bucket contains:

```text
index.html
error.html
```

Static website hosting was enabled, allowing S3 to serve the website content.

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

CloudFront provides global edge caching so users can retrieve content from an edge location closer to them rather than directly from the S3 origin.

### 3. AWS Certificate Manager

An ACM certificate was created for the custom domain and associated with the CloudFront distribution.

This enables HTTPS access to the website.

For CloudFront, the ACM certificate must be created in the **US East (N. Virginia) / `us-east-1`** region.

### 4. Cloudflare DNS

Instead of Route 53, **Cloudflare** was used to manage the domain's DNS records.

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

## CI/CD with GitHub Actions

A GitHub Actions workflow was added to automate website deployments.

The deployment process follows:

```text
Developer
    │
    ▼
GitHub Repository
    │
    ▼
GitHub Actions
    │
    ├── Sync website files
    │
    ▼
Amazon S3
    │
    ▼
CloudFront Invalidation
    │
    ▼
Updated Website
```

This removes the need to manually upload website files to S3 after every change.

## Deployment Workflow

A typical deployment consists of:

1. Developer pushes changes to GitHub.
2. GitHub Actions starts the deployment workflow.
3. Website files are synchronized to the S3 bucket.
4. CloudFront cache is invalidated.
5. CloudFront retrieves the updated files from S3.
6. Users receive the latest version of the website.

## Security Considerations

The project also demonstrates several production-oriented security concepts:

* HTTPS for encrypted client connections
* TLS certificate management through ACM
* CloudFront used as the public delivery layer
* DNS managed through Cloudflare
* AWS credentials stored as GitHub Actions secrets rather than committed to the repository
* Minimal deployment permissions for the CI/CD identity
* CDN caching to reduce direct load on the S3 origin

## Key DevOps Concepts Demonstrated

### Infrastructure and Hosting

* AWS S3 static website hosting
* CDN architecture
* CloudFront distributions
* Origin configuration
* DNS management
* SSL/TLS certificates

### Performance

* Global edge caching
* CloudFront cache policies
* Object compression
* Cache invalidation

### Automation

* GitHub Actions
* Automated S3 deployments
* Automated CloudFront invalidation
* Secrets management

### Troubleshooting

The project also provided practical experience with:

* DNS resolution
* CloudFront distribution configuration
* HTTPS certificate validation
* S3 website hosting
* CDN caching behavior
* Cache invalidation
* CI/CD deployment failures

## What I Learned

This project demonstrated how a relatively simple static website can be deployed using a production-style architecture rather than being served directly from a single server.

The key architecture principle is:

> **S3 provides the storage/origin, CloudFront provides global delivery and HTTPS, and Cloudflare provides DNS management.**

The addition of GitHub Actions turns the infrastructure into an automated deployment workflow, allowing changes pushed to the repository to be automatically deployed and distributed through CloudFront.

## Future Improvements

Potential improvements include:

* Add security headers using CloudFront Functions
* Implement stricter S3 bucket access controls
* Add automated testing before deployment
* Add cache-control headers for different asset types
* Add monitoring and alerting
* Add Infrastructure as Code using Terraform
* Implement separate development and production environments
* Add Cloudflare security and caching features where appropriate

## Project Outcome

The completed project demonstrates an end-to-end DevOps workflow for deploying a static website with:

**GitHub → GitHub Actions → S3 → CloudFront → Cloudflare DNS → End Users**

It combines cloud hosting, CDN delivery, DNS, HTTPS, caching, and CI/CD automation into a single production-oriented deployment.
