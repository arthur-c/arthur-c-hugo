---
date: 2018-06-13T00:00:00+00:00
Categories:
- kubernetes
Description: ""
Tags:
- kubernetes
- aws
- kms
- secrets
menu: main
title: Kubernetes secrets management with AWS KMS
---


A workflow to manage secrets and configuration on Kubernetes.

**PROS:**

  - simple
  - single source of truth
  - securely versionned
  - decrypted on the fly
  - private key managed by AWS

**CONS:**

  - you have to trust AWS
  - kubernetes secrets are stored unencrypted in etcd (https://github.com/kubernetes/features/issues/92)


# Step 1: AWS KMS key creation

```
export AWS_DEFAULT_REGION=eu-west-1
export AWS_PROFILE=staging
export KMS_KEY_NAME=myproject
aws kms create-key --description $KMS_KEY_NAME --output json | jq .KeyMetadata.Arn | xargs aws kms create-alias --alias-name alias/$KMS_KEY_NAME --target-key-id
```

# Step 2: secret encryption

```
aws kms encrypt --key-id alias/$KMS_KEY_NAME --plaintext fileb://myproject_secrets --query CiphertextBlob --output text > myproject_secrets.encrypted
```

  - myproject_secrets should be deleted and never committed.
  - myproject_secrets.encrypted can be safely stored in the project repository.


# Step 3: secret decryption and kubernetes secret creation

```
cat myproject_secrets.encrypted | base64 -d > myproject_secrets.encrypted.raw
aws kms decrypt --ciphertext-blob fileb://myproject_secrets.encrypted.raw --output text --query Plaintext | base64 -d > myproject_secrets
kubectl create secret generic myproject_secrets --from-env-file=myproject_secrets
rm myproject_secrets myproject_secrets.encrypted.raw
```

# Notes

## AWS IAM permissions

The user/process creating the kubernetes secret needs at least the permission to decrypt with the key:

```
        {
            "Sid": "VisualEditor33",
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt"
            ],
            "Resource": [
                "arn:aws:kms:eu-west-1:896828068574:key/XXXXX"
            ]
        }
```

## Kubernetes secrets versionning

Versionning Kubernetes secrets allows rollbacks and treat configuration changes
as new deployments

Examples:

```
kubectl create secret generic myproject_secrets-$BUILD_NUMBER
```

or

```
kubectl create secret generic myproject_secrets-$GIT_COMMIT
```
