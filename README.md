# MusicCPR Infrastructure on AWS via OpenTofu

## tfvars

Make a file called `terraform.tfvars` in the root of this repo with content like:

```txt
aws_profile = "cprprod" # should correspond to a profile name in ~/.aws/creds or similar
# aws_region = "us-east-1" # default
project_name = "cpr"
# instance_type = "t3.micro" # default
# db_instance_class = "db.t3.micro" # default
db_username = "STRING"
db_password = "STRING"
ses_domain = "musiccpr.org"
ses_email  = "aws@musiccpr.org"
```

## Process resulting in this repo: 

1. Ray said we should use OpenTofu.
2. I asked claude to
    ```txt
    ec2-with-rds.tf to use opentofu to define infrastructure to host a full stack web app (django): an ec2 instance with an s3 bucket and an rds instance (postgres)
    ```
3. it did, and told me

    ```md
    Done. I've created a complete OpenTofu configuration for your Django stack with:

    **Resources created:**

    * VPC with public and private subnets across 2 availability zones
    * EC2 instance (Ubuntu 22.04) with:
        * Security group allowing SSH, HTTP, and HTTPS
        * IAM role for S3 access
        * User data script installing Python, nginx, and PostgreSQL client
    * RDS PostgreSQL 15 instance with:
        * Encrypted storage
        * Private subnet placement (only accessible from EC2)
        * Auto-scaling storage up to 100GB
    * S3 bucket for static/media files with:
        * Versioning enabled
        * Public access blocked
        * IAM policy granting EC2 access
    
    **To use this:**

    1. Create a terraform.tfvars file with your credentials:

        > db_username = "your_db_admin"
        > db_password = "your_secure_password"
    2. Ensure you have an SSH key at ~/.ssh/id_rsa.pub (or update line 303) [sic]
    3. Run:
        tofu init
        tofu plan
        tofu apply
    
    The outputs will give you the EC2 public IP, RDS endpoint, and S3 bucket name to configure in your Django settings.
    ```
4. I created an ssh key and called it cpr_tofu and updated the corresponding line (323 at the time)
5. I updated my ~/.aws/credentials file with current keys for cprprod profile
6. I replaced the generated provider config with 
    ```txt
    provider "aws" {
        shared_config_files      = ["~/.aws/config"]
        shared_credentials_files = ["~/.aws/credentials"]
        profile                  = "cprprod"
    }
    ```
7. I was getting permissions errors so i attached a new policy
    ```json
    {
        "Effect": "Allow",
        "Action": [
            "ec2:DescribeAvailabilityZones",
            "ec2:DescribeImages"
        ],
        "Resource": "*"
    }
    ```
8. i asked claude to add backup to ec2 and rds. needed more perms
    ```txt
    "backup:CreateBackupVault",
    "backup:DeleteBackupVault",
    "backup:DescribeBackupVault",
    "backup:ListBackupVaults",
    "backup:CreateBackupPlan",
    "backup:DeleteBackupPlan",
    "backup:GetBackupPlan",
    "backup:ListBackupPlans",
    "backup:CreateBackupSelection",
    "backup:DeleteBackupSelection",
    "backup:GetBackupSelection",
    "backup:ListBackupSelections",
    "backup:TagResource",
    "backup:UntagResource",
    "backup:ListTags"
    ```
9. asked claude to add ses identity domain and identity email address. it let me know:
    ```txt
    **New variables (add to your terraform.tfvars):**
    
    ses_domain = "yourdomain.com"
    ses_email  = "noreply@yourdomain.com"
    
    **Resources added:**
    
    * Domain identity with DKIM signing and custom MAIL FROM domain
    * Email identity for a specific email address
    * IAM policy allowing EC2 to send emails via SES (restricted to your specified sender address)
    
    DNS records you'll need to add after apply:

    1. TXT record for domain verification (value in ses_domain_verification_token output)
    2. 3 CNAME records for DKIM (values in ses_dkim_tokens output)
    3. MX record for mail-from domain: mail.yourdomain.com → feedback-smtp.<region>.amazonses.com
    4. TXT record for mail-from SPF: mail.yourdomain.com → v=spf1 include:amazonses.com ~all
    
    The email identity will receive a verification email from AWS that must be clicked to confirm.
    ```