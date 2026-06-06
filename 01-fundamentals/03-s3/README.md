# S3 - Simple Storage Service

## What is S3?

Amazon S3 (Simple Storage Service) is object storage for the internet. Think of it as **an infinite hard drive in the cloud** where you can store any file and access it from anywhere.

**S3 is the most popular AWS service** - used by millions of applications for everything from backups to hosting websites to machine learning datasets.

---

## Core Concepts

### 1. **Bucket**
A container that holds objects (files). Like a folder, but globally unique.

```
Bucket name: my-company-photos
├── photos/
│   ├── vacation-2023.jpg
│   └── family-pic.jpg
├── videos/
│   └── holiday-2023.mp4
└── documents/
    └── annual-report.pdf
```

**Rules:**
- Name must be globally unique (no one else can have it)
- Name must be 3-63 characters, lowercase
- Must follow DNS naming rules
- Once deleted, name can be reused after 30 minutes

### 2. **Object**
A file stored in S3 (up to 5TB).

```
Object Key (path): photos/vacation-2023.jpg
Size: 2.5 MB
Last Modified: 2023-12-15 10:30:00
ETag: abc123def456 (unique identifier)
```

### 3. **Object Key (Path)**
The full path to the object, including "folders".

```
Bucket: my-photos
Object key: vacation/2023/photo-001.jpg

Full path: https://my-photos.s3.amazonaws.com/vacation/2023/photo-001.jpg
```

**Note:** S3 doesn't have real folders. It just looks like folders because of the "/" in the key.

---

## S3 Access Levels

### 1. **Private (Default)**
Only the AWS account owner can access.

```
Bucket: my-company-secrets
Access: Only my AWS account can read/write
Others: Cannot even see it exists
```

### 2. **Public Read**
Anyone can read, only owner can write.

```
Bucket: my-blog-images
Access: Anyone can view images
Security: Images must be safe for public

Use case: Hosting public images, documentation
```

### 3. **Public Read/Write**
Anyone can read and write (rarely used, risky!).

```
CAUTION: This is dangerous!
Anyone can upload files (storage costs)
Anyone can modify/delete files
Could be used for malware distribution
```

### 4. **Restricted (CloudFront)**
Access only through CloudFront (CDN).

```
Users:
1. Try to access S3 bucket directly ❌
2. CloudFront is the only gateway ✓
3. CloudFront serves from cache
4. CloudFront fetches from S3 if not cached
```

---

## S3 Storage Classes

Different storage options for different use cases.

### 1. **S3 Standard**
Default storage class. Fast access, frequently accessed data.

```
Cost: $0.023 per GB
Retrieval: Immediate
Availability: 99.99%
Durability: 99.999999999% (11 nines!)

Use case:
- Website content
- Mobile app data
- Active databases
- Frequently accessed files
```

### 2. **S3 Standard-IA (Infrequent Access)**
Lower cost for infrequent access (min 30 days).

```
Cost: $0.0125 per GB (45% cheaper)
Retrieval: Immediate
Minimum: 30 days
Retrieval fee: $0.01 per GB

Use case:
- Database backups (accessed occasionally)
- Log files (occasional analysis)
- Disaster recovery
- Earlier database versions
```

### 3. **S3 One Zone-IA**
Even cheaper, but only in one AZ (not replicated).

```
Cost: $0.01 per GB (57% cheaper)
Retrieval: Immediate
Minimum: 30 days
Risk: If AZ fails, data is lost

Use case:
- Non-critical data
- Data you have backup of elsewhere
- Development/test environments
```

### 4. **S3 Glacier Instant Retrieval**
Long-term archival, fast retrieval (min 90 days).

```
Cost: $0.004 per GB (83% cheaper!)
Retrieval: Minutes (instant)
Minimum: 90 days
Retrieval fee: $0.03 per GB

Use case:
- Year-long data retention
- Quarterly access patterns
- Compliance requirements
```

### 5. **S3 Glacier Deep Archive**
Ultra-cheap archival (min 180 days).

```
Cost: $0.0018 per GB (92% cheaper!)
Retrieval: 12 hours (bulk), 48 hours (standard)
Minimum: 180 days
Retrieval fee: $0.10 per GB

Use case:
- Annual compliance archives
- 7-year retention requirements
- Rarely accessed historical data
```

### Cost Comparison Example:

```
Store 100GB for 1 year:

S3 Standard:
- Monthly: 100 × $0.023 = $2.30
- Yearly: $2.30 × 12 = $27.60

S3 Standard-IA:
- Monthly: 100 × $0.0125 = $1.25
- Retrieval fee: $0.01 × 100 = $1.00
- Yearly: ($1.25 + $1.00) × 12 = $26.40

S3 Glacier Instant Retrieval:
- Monthly: 100 × $0.004 = $0.40
- Retrieval fee: $0.03 × 100 = $3.00
- Yearly: ($0.40 + $3.00) × 12 = $40.80

S3 Glacier Deep Archive:
- Monthly: 100 × $0.0018 = $0.18
- Retrieval fee: $0.10 × 100 = $10.00
- Yearly: ($0.18 + $10.00) × 12 = $122.16
```

---

## S3 Lifecycle Policies

Automatically move objects between storage classes.

### Example: Archive Old Data

```json
{
  "Rules": [
    {
      "Id": "Archive old backups",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "backups/"
      },
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        },
        {
          "Days": 365,
          "StorageClass": "DEEP_ARCHIVE"
        }
      ],
      "Expiration": {
        "Days": 2555
      }
    }
  ]
}
```

**Timeline:**
```
Day 0: Upload backup → S3 Standard
Day 30: Auto-move → S3 Standard-IA
Day 90: Auto-move → S3 Glacier
Day 365: Auto-move → S3 Deep Archive
Day 2555 (7 years): Auto-delete
```

---

## Versioning

Keep multiple versions of an object.

### Why Use Versioning?

```
Problem:
1. Upload file.txt (v1)
2. Upload file.txt again (v2)
3. Oops, v2 was wrong!
4. v1 is gone 😞

Solution: Enable versioning
1. Upload file.txt → Version ID: 111
2. Upload file.txt → Version ID: 222
3. Can access both versions!
4. Can restore v1 if needed
```

### Enabled Example:

```
Bucket: my-docs
Versioning: Enabled

my-docs/report.pdf
├── Version 1 (2023-12-01) - draft
├── Version 2 (2023-12-15) - final
└── Version 3 (2024-01-10) - revised
```

### Access Specific Version:

```bash
# Get latest version
aws s3 cp s3://my-docs/report.pdf .

# Get specific version
aws s3api get-object \
  --bucket my-docs \
  --key report.pdf \
  --version-id 222 \
  report.pdf
```

---

## S3 Permissions & Access Control

### 1. **Bucket Policy**
JSON policy for bucket-level access.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-public-bucket/*"
    }
  ]
}
```
**Meaning:** Anyone can read objects in this bucket

### 2. **ACL (Access Control List)**
Simple allow/deny for specific users/groups.

```
Options:
- Private (default)
- Public Read
- Public Read/Write
- Authenticated Users Read
```

### 3. **Bucket-Level Policies**

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT:root"
      },
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

### 4. **Presigned URLs**
Temporary access to private objects.

```python
import boto3
from datetime import timedelta

s3_client = boto3.client('s3')

# Generate presigned URL (valid for 1 hour)
url = s3_client.generate_presigned_url(
    'get_object',
    Params={
        'Bucket': 'my-private-bucket',
        'Key': 'confidential-doc.pdf'
    },
    ExpiresIn=3600  # 1 hour in seconds
)

print(url)
# Output: https://my-private-bucket.s3.amazonaws.com/...?X-Amz-Signature=...
```

**Use case:**
- Download links that expire
- Sharing files without making bucket public
- Temporary access to sensitive files

---

## S3 Performance & Optimization

### 1. **Multipart Upload**
Upload large files in parallel chunks.

```
Without multipart:
- Single connection
- 5GB file upload

With multipart:
- 4 parallel connections
- Upload 4 × 1.25GB chunks simultaneously
- 4x faster!
```

### 2. **Transfer Acceleration**
Use CloudFront edge locations for faster uploads.

```bash
# Enable Transfer Acceleration
aws s3api put-bucket-accelerate-configuration \
  --bucket my-bucket \
  --accelerate-configuration Status=Enabled

# Upload using accelerated endpoint
aws s3 cp large-file.zip s3://my-bucket/ \
  --region us-west-2
```

### 3. **Partitioning (Key Design)**
Create efficient key structure.

```
AVOID:
s3://my-bucket/data/file-001.parquet
s3://my-bucket/data/file-002.parquet
s3://my-bucket/data/file-003.parquet
(All have same prefix: "data/")

USE:
s3://my-bucket/data/2023/12/15/file-001.parquet
s3://my-bucket/data/2023/12/16/file-002.parquet
s3://my-bucket/data/2024/01/10/file-003.parquet
(Distributed prefixes)
```

---

## S3 Encryption

### 1. **Server-Side Encryption (SSE-S3)**
AWS manages encryption keys automatically.

```
Default: Enabled for all new buckets
How: AES-256 encryption
Key management: AWS handles everything
Cost: No extra charge
```

### 2. **Server-Side Encryption (SSE-KMS)**
You control encryption keys via AWS KMS.

```
Key management: You manage via KMS
Benefits: Key rotation, audit logs, fine-grained access
Cost: ~$1/month per key + per request fee
Compliance: PCI-DSS, HIPAA, etc.
```

### 3. **Client-Side Encryption**
Encrypt before uploading.

```python
from cryptography.fernet import Fernet
import boto3

# Generate key
key = Fernet.generate_key()
cipher = Fernet(key)

# Read file
with open('sensitive.txt', 'rb') as f:
    data = f.read()

# Encrypt
encrypted_data = cipher.encrypt(data)

# Upload encrypted
s3_client = boto3.client('s3')
s3_client.put_object(
    Bucket='my-bucket',
    Key='sensitive.txt',
    Body=encrypted_data
)
```

---

## S3 with CloudFront (CDN)

Distribute content globally with caching.

```
User in London:
1. Request image from CloudFront
2. CloudFront checks edge location in London
3. If cached: Return immediately ✓
4. If not cached: Fetch from S3 (origin)
5. Cache in London edge location
6. Return to user

User in Tokyo:
1. Request same image from CloudFront
2. Check edge location in Tokyo
3. If not cached: Fetch from origin S3
4. Cache in Tokyo edge location
5. Return to user
```

### Setup:

```bash
# Create CloudFront distribution
aws cloudfront create-distribution \
  --origin-domain-name my-bucket.s3.amazonaws.com \
  --default-root-object index.html
```

---

## S3 Event Notifications

Trigger actions when objects are uploaded/deleted.

### Integration with Lambda:

```json
{
  "EventSource": "aws:s3",
  "EventName": "ObjectCreated:Put",
  "Records": [
    {
      "s3": {
        "bucket": {
          "name": "my-bucket"
        },
        "object": {
          "key": "uploads/photo-001.jpg",
          "size": 2500000
        }
      }
    }
  ]
}
```

**Use case:**
```
1. User uploads photo to S3
2. S3 triggers Lambda function
3. Lambda processes image (resize, optimize)
4. Lambda stores result in separate bucket
5. Lambda updates database with metadata
```

---

## Hands-On: Create & Use S3 Bucket

```bash
# 1. Create bucket
aws s3 mb s3://my-unique-bucket-name-12345

# 2. Upload file
aws s3 cp myfile.txt s3://my-unique-bucket-name-12345/

# 3. List contents
aws s3 ls s3://my-unique-bucket-name-12345/

# 4. Download file
aws s3 cp s3://my-unique-bucket-name-12345/myfile.txt .

# 5. Make bucket public
aws s3api put-bucket-policy \
  --bucket my-unique-bucket-name-12345 \
  --policy file://policy.json

# 6. Generate presigned URL (valid 1 hour)
aws s3 presign \
  s3://my-unique-bucket-name-12345/myfile.txt \
  --expires-in 3600

# 7. Delete file
aws s3 rm s3://my-unique-bucket-name-12345/myfile.txt

# 8. Delete bucket (must be empty)
aws s3 rb s3://my-unique-bucket-name-12345/
```

---

## S3 Best Practices

### ✅ DO:

1. **Enable versioning** for important data
2. **Use lifecycle policies** to manage costs
3. **Enable encryption** (at minimum SSE-S3)
4. **Block public access** by default
5. **Use presigned URLs** instead of public buckets
6. **Enable logging** for audit trails
7. **Monitor costs** with CloudWatch metrics

### ❌ DON'T:

1. Make buckets public unnecessarily
2. Leave old objects cluttering the bucket
3. Use bucket names with account info
4. Ignore encryption requirements
5. Grant excessive permissions
6. Rely solely on ACLs (use bucket policies)
7. Store secrets in S3 (use Secrets Manager)

---

## Common S3 Mistakes

### ❌ Mistake 1: All Data in Standard Storage
```
COST: 100GB for 1 year = $27.60

Better:
- Recent data (90 days): Standard
- 90-365 days: Standard-IA
- 365+ days: Glacier

SAVINGS: 60-70% reduction
```

### ❌ Mistake 2: Public Bucket = Data Breach
```
BAD: make it public "temporarily"
GOOD: Use presigned URLs, CloudFront, or VPC endpoints
```

### ❌ Mistake 3: No Versioning = Data Loss
```
BAD: Accidentally overwrite important file
GOOD: Enable versioning, can restore previous version
```

### ❌ Mistake 4: Ignoring Event Notifications
```
BAD: Manual processing after uploads
GOOD: S3 events trigger Lambda for automatic processing
```

---

## Summary

| Topic | Key Point |
|-------|-----------|
| **Bucket** | Global container for objects |
| **Object** | File stored in S3 (up to 5TB) |
| **Storage Class** | Choose based on access patterns |
| **Versioning** | Keep multiple versions of objects |
| **Encryption** | Protect data at rest |
| **Permissions** | Bucket policy, ACL, IAM |
| **Lifecycle** | Auto-transition to cheaper storage |
| **CloudFront** | Global distribution with caching |

---

## Next Steps

1. Create an S3 bucket
2. Upload some files
3. Make one file public
4. Generate presigned URL
5. Move to [Databases - RDS & DynamoDB](../04-databases/README.md)

---

## Resources

- S3 Documentation: https://docs.aws.amazon.com/s3/
- S3 Pricing: https://aws.amazon.com/s3/pricing/
- Storage Classes: https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html
- S3 Best Practices: https://docs.aws.amazon.com/AmazonS3/latest/userguide/BestPractices.html
