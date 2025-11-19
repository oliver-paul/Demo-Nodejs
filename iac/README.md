# Sparrow Docs Infrastructure

This directory contains OpenTofu (Terraform) infrastructure as code for deploying a static website on AWS with CloudFront CDN.

## Architecture

The infrastructure includes:

1. **S3 Bucket** (`sparrowdocs.com`)
   - Static website hosting
   - Private bucket with CloudFront-only access
   - Automatic upload of `index.html`

2. **ACM Certificate**
   - SSL/TLS certificate for `sparrowdocs.com` and `www.sparrowdocs.com`
   - DNS validation required
   - Deployed in `us-east-1` (required for CloudFront)

3. **CloudFront Distribution**
   - Global CDN for fast content delivery
   - Origin Access Control (OAC) for secure S3 access
   - HTTPS redirect enabled
   - Custom error pages for SPA support

## Prerequisites

- [OpenTofu](https://opentofu.org/) >= 1.6.0 or Terraform >= 1.6.0
- AWS CLI configured with appropriate credentials
- AWS account with permissions to create:
  - S3 buckets
  - CloudFront distributions
  - ACM certificates
  - IAM policies

## Configuration

### Variables

You can customize the deployment by modifying variables in `variables.tofu` or creating a `terraform.tfvars` file:

```hcl
aws_region             = "us-east-1"
domain_name            = "sparrowdocs.com"
environment            = "production"
cloudfront_price_class = "PriceClass_100"
```

## Deployment Steps

### 1. Initialize OpenTofu

```bash
tofu init
```

Or with Terraform:

```bash
terraform init
```

### 2. Review the Plan

```bash
tofu plan
```

### 3. Apply the Infrastructure

```bash
tofu apply
```

Type `yes` when prompted to confirm.

### 4. DNS Validation for ACM Certificate

After applying, you'll need to validate the ACM certificate:

1. Get the validation records:
   ```bash
   tofu output acm_certificate_domain_validation_options
   ```

2. Add the CNAME records to your DNS provider (e.g., Route53, Cloudflare, etc.)

3. Wait for validation (can take 5-30 minutes)

### 5. Configure DNS for Your Domain

After the infrastructure is created, point your domain to CloudFront:

1. Get the CloudFront domain name:
   ```bash
   tofu output cloudfront_domain_name
   ```

2. Create DNS records:
   - **A Record** (or ALIAS if using Route53): `sparrowdocs.com` → CloudFront distribution
   - **CNAME Record**: `www.sparrowdocs.com` → CloudFront distribution

## Outputs

After successful deployment, you'll see:

- `s3_bucket_name` - Name of the S3 bucket
- `cloudfront_distribution_id` - CloudFront distribution ID
- `cloudfront_domain_name` - CloudFront URL (use this for DNS)
- `acm_certificate_arn` - ACM certificate ARN
- `website_url` - Your website URL (https://sparrowdocs.com)

## Updating Content

To update the website content:

1. Modify `index.html`
2. Run `tofu apply` to upload the new version
3. Invalidate CloudFront cache (optional, for immediate updates):
   ```bash
   aws cloudfront create-invalidation \
     --distribution-id $(tofu output -raw cloudfront_distribution_id) \
     --paths "/*"
   ```

## Security Features

- ✅ S3 bucket is private (no public access)
- ✅ CloudFront Origin Access Control (OAC) for secure S3 access
- ✅ HTTPS enforced (HTTP redirects to HTTPS)
- ✅ TLS 1.2+ only
- ✅ SSL/TLS certificate via ACM

## Cost Optimization

- Uses `PriceClass_100` by default (North America & Europe only)
- Change to `PriceClass_200` or `PriceClass_All` for global coverage
- S3 Standard storage class (consider S3 Intelligent-Tiering for large sites)

## Cleanup

To destroy all resources:

```bash
tofu destroy
```

**Warning:** This will permanently delete the S3 bucket, CloudFront distribution, and ACM certificate.

## Troubleshooting

### Certificate Validation Pending

If the ACM certificate is stuck in "Pending Validation":
- Verify DNS records are correctly configured
- Wait up to 30 minutes for DNS propagation
- Check AWS Certificate Manager console for validation status

### CloudFront 403 Errors

If you see 403 errors:
- Ensure the S3 bucket policy is correctly applied
- Verify Origin Access Control is attached to the distribution
- Check that `index.html` exists in the S3 bucket

### DNS Not Resolving

- Verify DNS records point to the CloudFront domain
- Allow time for DNS propagation (up to 48 hours)
- Use `dig` or `nslookup` to verify DNS records

## Additional Resources

- [OpenTofu Documentation](https://opentofu.org/docs/)
- [AWS CloudFront Documentation](https://docs.aws.amazon.com/cloudfront/)
- [AWS S3 Documentation](https://docs.aws.amazon.com/s3/)
- [AWS ACM Documentation](https://docs.aws.amazon.com/acm/)
