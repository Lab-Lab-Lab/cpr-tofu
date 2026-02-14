# MusicCPR Infrastructure on AWS via OpenTofu

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