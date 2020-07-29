# aws-s3-security
Controls to secure S3 data

- White list IP ranges: Deny API actions on an S3 bucket and its objects unless from specified IP addresses in the Condition block of the policy.
```
{
  "Version": "2012-10-17",
  "Id": "Example",
  "Statement": [
    {
      "Sid": "IPWhiteList",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
	       "arn:aws:s3:::mys3bucket",
         "arn:aws:s3:::mys3bucket/*"
      ],
      "Condition": {
	 "NotIpAddress": {"aws:SourceIp": "203.0.113.0/24"}
      }
    }
  ]
}
```

- White list and blacklist IP ranges: Alow API actions on  S3 bucket objects when API action originates from specified IP addresses in  IpAddress entry of the policy, but deny if originated from NotIpAddress IP ranges.
```
{
  "Id":"PolicyId2",
  "Version":"2012-10-17",
  "Statement":[
    {
      "Sid":"AllowIPmix",
      "Effect":"Allow",
      "Principal":"*",
      "Action":"s3:*",
      "Resource":"arn:aws:s3:::mys3bucket/*",
      "Condition": {
        "IpAddress": { "aws:SourceIp": "203.0.113.0/24"},
        "NotIpAddress": { "aws:SourceIp": "54.0.143.128/30"}
      }
    }
  ]
}
```
- Enforce MFA: Deny any operation on an S3 bucket unless authintcated with MFA.
```
{
    "Version": "2012-10-17",
    "Id": "Example",
    "Statement": [
      {
        "Sid": "Example",
        "Effect": "Deny",
        "Principal": "*",
        "Action": "s3:*",
        "Resource": "arn:aws:s3:::mys3bucket/applogs/*",
        "Condition": { "Null": { "aws:MultiFactorAuthAge": true }}
      }
    ]
 }
```
- Bucket owner must have full control of objects uploaded to S3 bucket from an external AWS account. 

Example policy

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "example1",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ExternalAccountID:user/developer01"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::mys3bucket/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-grant-full-control": "id=SourceAaccount-CanonicalUserID"
        }
      }
    }
  ]
}
```
Example of CLI command from external account used to give bucket owner in source account full control of created S3 objects:
```
aws s3api put-object-acl --bucket mys3bucket --key keyname --acl bucket-owner-full-control
```
- Limit S3 bucket creation to a specific region.
```
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Sid":"Example",
         "Effect":"Allow",
         "Action": "s3:CreateBucket",
         "Resource": "arn:aws:s3:::*",
         "Condition": {
             "StringLike": {
                 "s3:LocationConstraint": "us-east-1"
             }
         }
       }
    ]
}
```
