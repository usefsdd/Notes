# Ceph RGW Interfaces

## 1. S3-Compatible REST API

you can use this API via:
* aws-cli
* SDKs
    * boto3 (Python)
    * AWS SDK for Java
* Tools
    * s3cmd
    * rclone
    * minio-client
* raw HTTP
    * PUT
    * GET
    * POST
    * DELETE

---

### 1. Bucket Operations
```
PUT /bucket create bucket
```
```
GET /bucket list contents
```
```
DELETE /bucket delete if empty
```
```
GET /bucket?location get region
```
```
PUT /bucket?acl / GET /bucket?acl manage ACLs
```

### 2. Object Operations
```
PUT /bucket/key	Upload an object
```
```
GET /bucket/key	Download an object
```
```
DELETE /bucket/key Delete an object
```
```
HEAD /bucket/key Get metadata (size, ETag, etc.)
```
```
PUT with x-amz-copy-source Copy an object
```
```
PUT with ?tagging Add tags
```
```
GET with ?versionId=<> Get specific version (if versioning is on)
```

### 3. Multipart Upload

Useful for uploading large files in parts.
Steps:
```
POST /bucket/key?uploads initiate
```
```
PUT /bucket/key?uploadId=<...>&partNumber=1 upload parts
```
```
POST /bucket/key?uploadId=<...> complete
```

### 4. Signed URLs (Pre-signed)
You can pre-sign URLs so clients can upload/download without having AWS credentials.

**Give a browser/user permission to upload for 10 mins.**

* s3.generate_presigned_url('put_object', {...}) ***(in boto3)***
* Internally generates a URL with X-Amz-Signature, X-Amz-Date, etc.

### 5. Authentication
Ceph RGW supports AWS Signature V2 and Signature V4. Signature V4 is now standard and required for most secure clients

Authentication involves:
* Authorization header
* Date and Host headers
* HMAC-SHA hashing of request and secret key

### 6. Access Control
Supported through:

```
Bucket Policies (PUT /bucket?policy)
```
```
ACLs (PUT /bucket?acl, x-amz-acl: public-read)
```
```
STS (optional): short-term token-based access
```

### 7. Tags and Metadata
```
PUT /bucket/key?tagging Add_tags
```
```
Custom metadata: headers like x-amz-meta-mykey: myvalue
```
```
GET /bucket/key?tagging Get_tags
```

---
---

## S3 Object Operations in Ceph RGW

This part provides examples of how to interact with a Ceph RGW bucket using different clients and protocols: `aws-cli`, `s3cmd`, `boto3`, and raw HTTP.

---

### Prerequisites

Set the following environment variables or use these values in your tools:

```bash
ENDPOINT=http://10.43.163.242:32004
ACCESS_KEY=TZ3VKX9LUB2K3PKSYVKO
SECRET_KEY=JgHAtoQ0H2qmiGElxRLMhd8DhXpgZSkrGpPCZwYD
BUCKET=test-bucket
```
Ensure you have a file called `text.txt` for upload tests.

---

### 1. `aws-cli`

#### Configure:

```bash
aws configure --profile myceph
# Or export env variables:
export AWS_ACCESS_KEY_ID=$ACCESS_KEY
export AWS_SECRET_ACCESS_KEY=$SECRET_KEY
```

#### Upload (PUT):

```bash
aws --endpoint-url $ENDPOINT s3 cp text.txt s3://$BUCKET/
```

#### Download (GET):

```bash
aws --endpoint-url $ENDPOINT s3 cp s3://$BUCKET/text.txt downloaded.txt
```

#### Delete:

```bash
aws --endpoint-url $ENDPOINT s3 rm s3://$BUCKET/text.txt
```

---

### 2. `s3cmd`

#### Configure:

```bash
s3cmd --configure
# Use ACCESS_KEY, SECRET_KEY, and ENDPOINT during config
```

#### Upload (PUT):

```bash
s3cmd put text.txt s3://$BUCKET/
```

#### Download (GET):

```bash
s3cmd get s3://$BUCKET/text.txt downloaded.txt
```

#### Delete:

```bash
s3cmd del s3://$BUCKET/text.txt
```

---

### 3. `boto3` (Python)

#### Install:

```bash
pip install boto3
```

#### Script:

```python
import boto3

access_key = 'TZ3VKX9LUB2K3PKSYVKO'
secret_key = 'JgHAtoQ0H2qmiGElxRLMhd8DhXpgZSkrGpPCZwYD'
endpoint = 'http://10.43.163.242:32004'
bucket = 'test-bucket'
key = 'text.txt'

s3 = boto3.client(
    's3',
    endpoint_url=endpoint,
    aws_access_key_id=access_key,
    aws_secret_access_key=secret_key,
    region_name='us-east-1'
)

# Upload
s3.upload_file('text.txt', bucket, key)

# Download
s3.download_file(bucket, key, 'downloaded.txt')

# Delete
s3.delete_object(Bucket=bucket, Key=key)
```

---

### 4. Raw HTTP (Presigned URL)

#### Generate presigned URL:

```python
url = s3.generate_presigned_url(
    'put_object',
    Params={'Bucket': bucket, 'Key': key},
    ExpiresIn=3600
)

print(f"curl -T text.txt '{url}'")
```

#### Upload via curl:

```bash
curl -T text.txt "<presigned-url>"
```

---

### Bonus: Raw HTTP Upload (Public Bucket Only)

If bucket is public:

```bash
curl -X PUT -T text.txt $ENDPOINT/$BUCKET/text.txt
```

---
---
---




### 2. SWIFT-Compatible REST API
### 3. Native Client SDK & CLI
### 4. radosgw-admin
### 5. Ceph Dashboard
