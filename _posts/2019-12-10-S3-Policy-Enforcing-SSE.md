---
layout: post
title: "S3 Policy - Enforcing default SSE-S3 encryption"
date: '2019-12-10'
categories: aws s3 encryption security s3-policy
author: brasskazoo
---

So you've setup default encryption on your bucket using the standard SSS-S3:

<img src="/assets/images/2019/12/AWS%20S3%20Default%20Encryption.png"  width="480">

But take note - to properly ensure that files being uploaded are encypted with the intended encryption method, you need to take measures in the S3 policy to deny attempts to override the default settings.

This example policy denies attempts to:
* Upload an unencrypted object (when the `x-amz-server-side-encryption` header is false), 
* Use an alternate encryption method (e.g. attempting to use a KMS key instead like `aws s3 cp ./mytextfile.txt s3://mytestbucket/ --sse aws:kms --sse-kms-key-id testkey`).

```
#####
# Deny attempts to override default server-side encryption using header x-amz-server-side-encryption
#
# Header val  | !=AES256  | !(is Null)  | Result  |
#  <empty>    | true      | false       | OK      |
#  AES256     | false     | true        | OK      |
#  false      | true      | true        | DENY    |
#  xyz        | true      | true        | DENY    |
#

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyIncorrectEncryptionHeader",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::mine-unit-inbound/*",
            "Condition": {
                "Null": {
                    "s3:x-amz-server-side-encryption": "false"
                },
                "StringNotEquals": {
                    "s3:x-amz-server-side-encryption": "AES256"
                }
            }
        }
    ]
}
```
