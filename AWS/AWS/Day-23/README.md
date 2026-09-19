# Day 23 — S3 Data Migration Using AWS CLI

## Objective

Migrate the complete contents of an existing S3 bucket to a new private S3 bucket and verify that the destination contains the
same data.

### Source

`devops-s3-778310932`

### Destination

`devops-sync-778310932`

### Region

`us-east-1`

---

## The Problem

The existing S3 bucket contained a substantial amount of application data that needed to be moved to a new bucket.

Manually copying individual files would be inefficient and could result in missing objects. We therefore used the AWS CLI to automate the migration and then verified the result.

---

## Solution

The migration followed three main steps:

**Create -> Sync -> Verify**

We used:

- Amazon S3 for object storage
- AWS CLI for bucket creation and data migration
- `aws s3 sync` to copy the data
- Recursive listing and a dry run to verify consistency

---

## Implementation

### 1. Verify the AWS CLI identity

```bash
aws sts get-caller-identity : This allowed us to confirm that the source bucket contained the expected application data.

### 2. Inspect the existing bucket:
 aws s3 ls s3://devops-s3-778310932

This allowed us to confirm that the source bucket contained the expected application data.

3. Create the new private bucket
aws s3 mb s3://devops-sync-778310932 --region us-east-1

mb means make bucket.

The --region option ensures the bucket is created in the
required us-east-1 region.

4. Migrate the data
aws s3 sync \
  s3://devops-s3-778310932 \
  s3://devops-sync-778310932 \
  --region us-east-1

sync compares the source and destination and transfers the
objects that need to be synchronized.

This is more practical than copying files individually,
especially when a bucket contains many objects.

Verification

After the migration, we compared the contents of both buckets.

Source
aws s3 ls s3://devops-s3-778310932 --recursive --summarize
Destination
aws s3 ls s3://devops-sync-778310932 --recursive --summarize

--recursive checks objects throughout the bucket, while
--summarize provides the total object count and total size.

The source and destination should have matching totals.

We also used a dry run:

aws s3 sync \
  s3://devops-s3-778310932 \
  s3://devops-sync-778310932 \
  --dryrun \
  --region us-east-1

--dryrun shows what AWS CLI would synchronize without actually
making changes. An empty result indicates that no further
synchronization is required.

Real-World Relevance

S3-to-S3 synchronization is useful for:

Data migrations
Backups
Disaster recovery
Moving application assets
Synchronizing environments
Cloud storage reorganizations

Key Takeaway

The important lesson was that data migration does not end when
the copy command finishes.

A successful migration should be followed by verification to
confirm that the destination contains the expected data.

aws s3 sync provides an efficient way to automate S3 data
migration while reducing the risk of manually missing objects.
