# Create a Customer Master Key (CMK) using the CLI

This step assumes you have permission via Identity and Access Management (IAM) to use AWS KMS.

## Create the CMK

```bash
aws kms create-key --description "KeyName" --region REGION
```

This command does NOT create a default key policy.  When using the AWS Console, one is created as part of the process.

The key policy is a resource policy that provides control mechanisms for who can **use** the CMK as well as who can **administer** the CMK.

## Output from create-key

```json
{
    "KeyMetadata": {
        "Origin": "AWS_KMS",
        "KeyId": "########-####-####-####-############",
        "Description": "KeyName",
        "KeyManager": "CUSTOMER",
        "Enabled": true,
        "KeyUsage": "ENCRYPT_DECRYPT",
        "KeyState": "Enabled",
        "CreationDate": ##########.###,
        "Arn": "arn:aws:kms:REGION:############:key/########-####-####-#####-############",
        "AWSAccountId": "############"
    }
}
```

The output is in JSON.  

## Create an alias

This is optional but makes using the key easier.

```bash
aws kms create-alias --target-key-id TARGET-KEY-ID --alias-name "alias/NAME" --region REGION
```

* The alias name must start with: alias/
* The REGION is specified in lower case

## Example Key Policy

This is from the AWS documentation, Using [Key Policies in AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html#key-policy-default)

When using the AWS Console, a default policy is built for you automatically.  There is a "wizard" of sorts that will walk you through granting appropriate access to IAM Users and Roles.

The AWS Root User Account must have full permissions to the CMK.  This allows permissions to be delegated appropriately.

```json
{
  "Version": "2012-10-17",
  "Id": "key-consolepolicy-2",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow access for Key Administrators",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:user/KMSAdminUser",
        "arn:aws:iam::111122223333:role/KMSAdminRole"
      ]},
      "Action": [
        "kms:Create*",
        "kms:Describe*",
        "kms:Enable*",
        "kms:List*",
        "kms:Put*",
        "kms:Update*",
        "kms:Revoke*",
        "kms:Disable*",
        "kms:Get*",
        "kms:Delete*",
        "kms:TagResource",
        "kms:UntagResource",
        "kms:ScheduleKeyDeletion",
        "kms:CancelKeyDeletion"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Allow use of the key",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:user/KMSUser",
        "arn:aws:iam::111122223333:role/KMSRole",
        "arn:aws:iam::444455556666:root"
      ]},
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Allow attachment of persistent resources",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:user/KMSUser",
        "arn:aws:iam::111122223333:role/KMSRole",
        "arn:aws:iam::444455556666:root"
      ]},
      "Action": [
        "kms:CreateGrant",
        "kms:ListGrants",
        "kms:RevokeGrant"
      ],
      "Resource": "*",
      "Condition": {"Bool": {"kms:GrantIsForAWSResource": "true"}}
    }
  ]
}
```

## Attach the key policy to CMK

Create the key policy and save it.  The following example assumes a filename of `policy.json`.

```bash
$ aws kms put-key-policy --key-id ########-####-####-####-############ --policy-name default --cli-input-json file://policy.json
```

If you have already have a CMK with a default policy, you can get it and edit it to suit your needs.

```bash
aws kms get-key-policy --policy-name default --key-id ########-####-####-####-############ > policy.json
```

## (My advice) Start with the AWS Console

From an automation perspective, using the console is problematic.  However, when building a CMK, the AWS Console will prompt you for the proper permissions and create a working key policy. 

Once built using the console, it can be saved and stored in a code repository with version control.

Then, automation becomes easier.

