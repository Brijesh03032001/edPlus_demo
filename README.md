# EdPlus Demo - AWS Deployment with CI/CD

> **Company-grade CI/CD pipeline for deploying static websites to AWS using GitHub Actions, Terraform, S3, and CloudFront**

![AWS](https://img.shields.io/badge/AWS-S3%20%2B%20CloudFront-orange)
![Terraform](https://img.shields.io/badge/Terraform-Infrastructure-purple)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-blue)

## 🏗️ Architecture

```
┌─────────────────┐
│  GitHub Actions │
│   (CI/CD)       │
└────────┬────────┘
         │
         ├─► Build Frontend
         │
         ├─► Terraform Apply
         │   ├─► Create S3 Bucket
         │   └─► Create CloudFront
         │
         ├─► Upload to S3
         │
         └─► Invalidate CloudFront Cache
                 │
                 ▼
         ┌──────────────────┐
         │   CloudFront CDN │
         │   (Global Edge)  │
         └────────┬─────────┘
                  │
                  ▼
         ┌──────────────────┐
         │    S3 Bucket     │
         │ (Static Hosting) │
         └──────────────────┘
```

## 📁 Project Structure

```
edplus_demo_lets_start/
│
├── site_need_to_be_deploy/    # Frontend application
│   ├── index.html
│   ├── about.html
│   ├── contact.html
│   ├── projects.html
│   ├── styles.css
│   └── script.js
│
├── terraform/                  # Infrastructure as Code
│   ├── provider.tf            # AWS provider configuration
│   ├── main.tf                # S3 + CloudFront resources
│   ├── variables.tf           # Input variables
│   ├── terraform.tfvars       # Variable values
│   └── outputs.tf             # Output values
│
├── .github/
│   └── workflows/
│       └── deploy.yml         # CI/CD pipeline
│
└── README.md                  # This file
```

## 🚀 Quick Start

### Prerequisites

- AWS Account with appropriate permissions
- GitHub repository
- Terraform >= 1.0 (for local testing)

### 1️⃣ Setup AWS Credentials

**Option A: Using OIDC (Recommended for Production)**

1. Create an OIDC provider in AWS IAM
2. Create an IAM role with S3, CloudFront, and IAM permissions
3. Add the role ARN to GitHub Secrets as `AWS_ROLE_ARN`

**Option B: Using Access Keys (Quick Setup)**

1. Create an IAM user with programmatic access
2. Attach policies: `AmazonS3FullAccess`, `CloudFrontFullAccess`
3. Add to GitHub Secrets:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`

### 2️⃣ Configure Terraform

Edit `terraform/terraform.tfvars`:

```hcl
bucket_name = "your-unique-bucket-name-2026"  # Must be globally unique!
aws_region  = "us-east-1"
environment = "prod"
```

### 3️⃣ Deploy

**Automated (via GitHub Actions):**
```bash
git add .
git commit -m "Initial deployment"
git push origin main
```

**Manual (local testing):**
```bash
cd terraform
terraform init
terraform plan
terraform apply
```

## 🔄 CI/CD Pipeline

The GitHub Actions workflow automatically:

1. ✅ **Triggers** on push to `main` branch
2. 🔐 **Authenticates** with AWS
3. 🏗️ **Provisions Infrastructure** via Terraform
   - Creates S3 bucket with static hosting
   - Sets up CloudFront distribution
   - Configures bucket policies and CORS
4. 📤 **Uploads Files** to S3 with optimized caching
   - Static assets: 1-year cache
   - HTML files: No cache (always fresh)
5. 🔄 **Invalidates CloudFront Cache** for immediate updates
6. 📊 **Reports** deployment details in GitHub Actions summary

## 📦 Infrastructure Details

### S3 Configuration
- Static website hosting enabled
- Public read access
- CORS configured
- Optimized cache headers

### CloudFront Configuration
- HTTPS enforced (redirects HTTP → HTTPS)
- Global edge locations (PriceClass_100)
- Gzip compression enabled
- SPA support (404 → index.html)
- Custom error pages

## 🔧 Customization

### Change AWS Region
Edit `terraform/terraform.tfvars`:
```hcl
aws_region = "eu-west-1"
```

### Add Custom Domain
1. Get SSL certificate in ACM (us-east-1)
2. Uncomment custom domain section in `terraform/main.tf`
3. Update Route53 or your DNS provider

### Modify Cache Settings
Edit the `aws s3 sync` commands in `.github/workflows/deploy.yml`

## 📊 Outputs

After deployment, you'll get:

- **Website URL**: CloudFront distribution URL
- **S3 Bucket**: Bucket name and ARN
- **CloudFront ID**: For manual cache invalidation

View outputs:
```bash
cd terraform
terraform output
```

## 🧹 Cleanup

To destroy all AWS resources:

```bash
cd terraform
terraform destroy
```

Or delete from AWS Console:
1. Empty and delete S3 bucket
2. Delete CloudFront distribution (wait 15-20 min)

## 💰 Cost Estimate

**Monthly costs (light traffic):**
- S3: ~$0.50 (1GB storage + requests)
- CloudFront: Free tier → $0-5
- **Total: < $5/month**

## 🔒 Security Best Practices

✅ Use OIDC instead of access keys  
✅ Enable Terraform state locking (use S3 backend)  
✅ Never commit `terraform.tfstate` or secrets  
✅ Use CloudFront signed URLs for private content  
✅ Enable AWS CloudTrail for audit logs  

## 🐛 Troubleshooting

**Issue: Bucket name already exists**
- S3 bucket names are globally unique
- Change `bucket_name` in `terraform.tfvars`

**Issue: CloudFront changes take time**
- CloudFront deployments can take 15-20 minutes
- Cache invalidation may take 5-10 minutes

**Issue: 403 Forbidden errors**
- Check bucket policy in S3
- Verify public access settings

## 📚 Learn More

- [AWS S3 Static Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [CloudFront Documentation](https://docs.aws.amazon.com/cloudfront/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [GitHub Actions](https://docs.github.com/en/actions)

## 📝 License

MIT License - feel free to use for your projects!

---

**Built with ❤️ for production-grade deployments**
