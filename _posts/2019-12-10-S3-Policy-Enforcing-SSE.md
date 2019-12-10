---
layout: post
title: "S3 Policy - Enforcing default SSE-S3 encryption"
date: '2019-12-10'
categories: aws s3 encryption security s3-policy
author: brasskazoo
---

So you've setup default encryption on your bucket using the standard SSS-S3:

![S3 Default Encryption](/assets/images/2019/12/AWS S3 Default Encryption.png)

But take note - to properly ensure that files being uploaded are encypted with the intended encryption method, you need to take measures in the S3 policy to deny attempts to override the default settings.

This example policy denies attempts to:
* Upload an unencrypted object (when the `x-amz-server-side-encryption` header is false), 
* Use an alternate encryption method (e.g. attempting to use a KMS key instead like `aws s3 cp ./mytextfile.txt s3://mytestbucket/ --sse aws:kms --sse-kms-key-id testkey`).

{% gist 2fd184b543f7b7c9bde5c6a29495778c %}
