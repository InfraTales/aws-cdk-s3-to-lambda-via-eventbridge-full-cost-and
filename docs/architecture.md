# Architecture Notes

## Overview

The stack wires an S3 bucket as the event source and routes object-level activity through CloudTrail data events into EventBridge, then triggers a VPC-placed Lambda function — meaning the function has private network access without relying on the legacy S3-to-Lambda direct notification path [from-code]. KMS encrypts the bucket and Lambda environment, Secrets Manager and SSM Parameter Store handle runtime credentials, and a CloudWatch LogGroup with explicit retention is attached to the Lambda so logs don't accumulate indefinitely [from-code]. The non-obvious design choice is using CloudTrail plus EventBridge as the event pipeline instead of a direct S3 event notification, which trades lower latency for richer filtering, a full audit record of every object operation, and the ability to fan out to multiple targets without touching the bucket configuration [editorial]. VPC endpoint provisioning for S3, Secrets Manager, and SSM is required for the Lambda to reach AWS services privately, and each endpoint carries an ~$7/month/AZ cost that teams consistently undercount when sizing this architecture [inferred].

## Key Decisions

- CloudTrail data events for S3 cost $0.10 per 100k events — a bucket receiving millions of PUTs per day can generate a CloudTrail bill that dwarfs the Lambda execution cost itself [inferred]
- Routing through EventBridge adds ~1-3 seconds of end-to-end latency compared to a direct S3-to-Lambda notification trigger — acceptable for audit pipelines, fatal for real-time processing requirements [inferred]
- Placing the Lambda inside a VPC adds cold start overhead (ENI attachment) and requires NAT Gateway or VPC endpoints for the Lambda to reach S3, Secrets Manager, and SSM — each VPC endpoint costs ~$7/month/AZ [inferred]
- KMS CMK usage across S3, Lambda, and CloudWatch Logs means every encryption/decryption operation is a KMS API call at $0.03 per 10k requests — high-throughput workloads need key caching or per-service key strategies to keep this bounded [inferred]
- LocalStack detection baked into the stack via environment variable checks is a pragmatic dev experience decision, but it introduces a conditional code path that is never exercised in CI unless LocalStack is explicitly wired into the pipeline [from-code]