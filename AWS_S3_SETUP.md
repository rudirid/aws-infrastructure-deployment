# AWS S3 Setup - File Storage Infrastructure

## Created: 2025-10-28

## Buckets Created

### 1. 99ai-crm-files
**Purpose:** Client files, call recordings, documents for 99.ai CRM
**Region:** us-east-1
**Features:**
- ✅ Versioning enabled (file history)
- ✅ AES-256 encryption (data at rest)
- ✅ Secure bucket policy (Ares user only)
- ✅ Enforced encryption on uploads

**Usage:**
```bash
# Upload file
aws s3 cp file.pdf s3://99ai-crm-files/clients/client-123/ --sse AES256

# List files
aws s3 ls s3://99ai-crm-files/

# Download file
aws s3 cp s3://99ai-crm-files/clients/client-123/file.pdf ./
```

### 2. asx-trading-data
**Purpose:** Trading system data, price history, pattern analysis
**Region:** us-east-1
**Features:**
- ✅ Versioning enabled
- ✅ AES-256 encryption
- ✅ Secure bucket policy
- ✅ Enforced encryption on uploads

**Usage:**
```bash
# Upload price data
aws s3 cp prices.csv s3://asx-trading-data/historical/2025/ --sse AES256

# Backup trades
aws s3 sync ./trades-backup s3://asx-trading-data/backups/
```

### 3. ares-system-backups
**Purpose:** System backups, Docker images, configuration snapshots
**Region:** us-east-1
**Features:**
- ✅ Versioning enabled
- ✅ AES-256 encryption
- ✅ Secure bucket policy

**Usage:**
```bash
# Backup database
aws s3 cp database-backup.sql s3://ares-system-backups/dynamodb/2025-10-28/ --sse AES256

# Backup Docker configs
aws s3 cp docker-compose.yml s3://ares-system-backups/docker/
```

## Security Configuration

**Bucket Policies:**
- Only Ares IAM user can access
- Encryption enforced on all uploads
- AES-256 encryption at rest

**Access Control:**
```json
{
  "Principal": {
    "AWS": "arn:aws:iam::933232224696:user/Ares"
  },
  "Action": [
    "s3:GetObject",
    "s3:PutObject",
    "s3:DeleteObject",
    "s3:ListBucket"
  ]
}
```

## Python Integration

### boto3 Example (for Docker containers):

```python
import boto3

# Initialize S3 client
s3 = boto3.client('s3', region_name='us-east-1')

# Upload file from CRM API
def upload_client_file(client_id, file_path):
    bucket = '99ai-crm-files'
    key = f'clients/{client_id}/{os.path.basename(file_path)}'

    s3.upload_file(
        file_path,
        bucket,
        key,
        ExtraArgs={'ServerSideEncryption': 'AES256'}
    )

    return f'https://{bucket}.s3.amazonaws.com/{key}'

# Download file
def download_client_file(client_id, filename, local_path):
    bucket = '99ai-crm-files'
    key = f'clients/{client_id}/{filename}'

    s3.download_file(bucket, key, local_path)
```

## Bucket Structure

### 99ai-crm-files/
```
clients/
  ├── client-001/
  │   ├── documents/
  │   ├── recordings/
  │   └── invoices/
  ├── client-002/
  └── ...
shared/
  ├── templates/
  └── resources/
test/
  └── test-upload.txt (✅ verified)
```

### asx-trading-data/
```
historical/
  ├── 2025/
  │   ├── prices/
  │   └── patterns/
backups/
  ├── trades/
  └── recommendations/
reports/
  └── analysis/
```

### ares-system-backups/
```
dynamodb/
  └── YYYY-MM-DD/
docker/
  ├── images/
  └── configs/
code/
  └── snapshots/
```

## Cost Optimization

**Storage Pricing (us-east-1):**
- First 50 TB: $0.023/GB/month
- Expected usage: ~10GB initially = $0.23/month

**Transfer Pricing:**
- Upload: FREE
- Download to EC2 (same region): FREE
- Download to internet: $0.09/GB (first 10TB)

**Optimization:**
- Lifecycle policies (move old data to Glacier)
- Intelligent-Tiering for varying access patterns
- CloudFront CDN for frequently accessed files

## Lifecycle Policy Example

```json
{
  "Rules": [
    {
      "Id": "Archive old backups",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ]
    }
  ]
}
```

## Integration with Docker Containers

**Environment Variables (.env):**
```bash
AWS_ACCESS_KEY_ID=****************
AWS_SECRET_ACCESS_KEY=****************
AWS_DEFAULT_REGION=us-east-1
S3_CRM_BUCKET=99ai-crm-files
S3_TRADING_BUCKET=asx-trading-data
S3_BACKUP_BUCKET=ares-system-backups
```

**Docker Compose (already configured):**
```yaml
services:
  crm-api:
    environment:
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
      - S3_CRM_BUCKET=99ai-crm-files
```

## Testing

**✅ Verified:**
- Bucket creation successful
- Versioning enabled on all buckets
- Encryption enabled (AES-256)
- Bucket policies applied
- Test upload successful (99ai-crm-files/test/test-upload.txt)

**Test Commands:**
```bash
# List all buckets
aws s3 ls

# Check bucket encryption
aws s3api get-bucket-encryption --bucket 99ai-crm-files

# Check bucket versioning
aws s3api get-bucket-versioning --bucket 99ai-crm-files

# Upload test file
aws s3 cp test.txt s3://99ai-crm-files/test/ --sse AES256
```

## Next Steps

✅ **S3 Setup Complete**

**Ready for:**
1. EC2 instance launch
2. Docker deployment to EC2
3. Application integration with S3 buckets
4. Automated backups to S3

## Cleanup (if needed)

```bash
# Delete test files
aws s3 rm s3://99ai-crm-files/test/ --recursive

# Empty and delete bucket (CAREFUL!)
aws s3 rb s3://99ai-crm-files --force
```

---

**Status:** ✅ Complete - 3 buckets operational
**Total Setup Time:** ~5 minutes
**Cost:** ~$0.23/month initially
**Region:** us-east-1
