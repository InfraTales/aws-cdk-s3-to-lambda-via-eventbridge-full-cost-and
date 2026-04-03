# Cost Model

## Overview

This is a reference cost model. Actual costs vary by usage, region, and configuration.

## Key Cost Drivers

- CloudTrail data events for S3 cost $0.10 per 100k events — a bucket receiving millions of PUTs per day can generate a CloudTrail bill that dwarfs the Lambda execution cost itself [inferred]
- Placing the Lambda inside a VPC adds cold start overhead (ENI attachment) and requires NAT Gateway or VPC endpoints for the Lambda to reach S3, Secrets Manager, and SSM — each VPC endpoint costs ~$7/month/AZ [inferred]
- KMS CMK usage across S3, Lambda, and CloudWatch Logs means every encryption/decryption operation is a KMS API call at $0.03 per 10k requests — high-throughput workloads need key caching or per-service key strategies to keep this bounded [inferred]

## Estimated Monthly Cost

| Component | Dev (₹) | Staging (₹) | Production (₹) |
|-----------|---------|-------------|-----------------|
| Compute   | ₹2,000–5,000 | ₹8,000–15,000 | ₹25,000–60,000 |
| Database  | ₹1,500–3,000 | ₹5,000–12,000 | ₹15,000–40,000 |
| Networking| ₹500–1,000   | ₹2,000–5,000  | ₹5,000–15,000  |
| Monitoring| ₹200–500     | ₹1,000–2,000  | ₹3,000–8,000   |
| **Total** | **₹4,200–9,500** | **₹16,000–34,000** | **₹48,000–1,23,000** |

> Estimates based on ap-south-1 (Mumbai) pricing. Actual costs depend on traffic, data volume, and reserved capacity.

## Cost Optimization Strategies

- Use Savings Plans or Reserved Instances for predictable workloads
- Enable auto-scaling with conservative scale-in policies
- Use DynamoDB on-demand for dev, provisioned for production
- Leverage S3 Intelligent-Tiering for infrequently accessed data
- Review Cost Explorer weekly for anomalies
