# 🚀 AWS S3 Static Website Deployment Guide

## 📋 Prerequisites

- AWS Account ([Sign up here](https://aws.amazon.com/))
- Your static website files (index.html, styles.css, etc.)

---

## 🎯 S3 Static Website Hosting

### Step 1: Create an S3 Bucket

1. **Log in to AWS Console**
   - Navigate to [S3 Console](https://console.aws.amazon.com/s3/)

2. **Create Bucket**
   - Click "Create bucket"
   - Bucket name: `mywebsite-static-hosting` (must be globally unique)
   - Region: Choose closest to your users (e.g., `ap-south-1` for India)
   - **Uncheck** "Block all public access"
   - Acknowledge the warning
   - Click "Create bucket"

### Step 2: Configure Static Website Hosting

1. **Open your bucket** and go to the **Properties** tab
2. Scroll to **Static website hosting**
3. Click **Edit**
4. Enable static website hosting
5. Index document: `index.html`
6. Error document: `index.html` (optional)
7. Click **Save changes**
8. **Note the endpoint URL** (e.g., `http://mywebsite-static-hosting.s3-website.ap-south-1.amazonaws.com`)

### Step 3: Upload Your Files

1. Go to the **Objects** tab
2. Click **Upload**
3. Add your files:
   - `index.html`
   - `styles.css`
   - Any images or other assets
4. Click **Upload**

### Step 4: Set Bucket Policy for Public Access

1. Go to the **Permissions** tab
2. Scroll to **Bucket policy**
3. Click **Edit**
4. Paste this policy (replace `YOUR-BUCKET-NAME`):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
        }
    ]
}
```

5. Click **Save changes**

### Step 5: Test Your Website

Visit the S3 website endpoint URL from Step 2!

🎉 **Your website is now live!**

---

## 💰 Cost Estimate

### S3 Static Website Hosting
- **Storage**: ~$0.023 per GB/month
- **GET Requests**: $0.0004 per 1,000 requests
- **Data Transfer**: First 100 GB free, then $0.09/GB
- **Typical small static site**: $0.50 - $2/month

### AWS Free Tier (First 12 Months)
- 5 GB of S3 storage
- 20,000 GET requests
- 2,000 PUT requests

**Your small static website will likely cost almost nothing!**

---

## 🔄 Updating Your Website

### Method 1: AWS Console (Easy)
1. Go to your S3 bucket
2. Click on the **Objects** tab
3. Select the file you want to update
4. Click **Delete** (or **Upload** to overwrite)
5. Click **Upload** to add the new version
6. Changes are live immediately!

### Method 2: AWS CLI (Faster)
```bash
# Upload a single file
aws s3 cp index.html s3://mywebsite-static-hosting/

# Upload all files (sync entire folder)
aws s3 sync . s3://mywebsite-static-hosting/

# Upload with cache control
aws s3 cp index.html s3://mywebsite-static-hosting/ --cache-control "max-age=3600"
```

---

## 🌐 Custom Domain Setup (Optional)

If you want to use your own domain (e.g., `www.mywebsite.com`):

### Step 1: Bucket Naming
- Your bucket name MUST match your domain exactly
- Example: `www.mywebsite.com` or `mywebsite.com`

### Step 2: Configure DNS
1. Go to your domain registrar (GoDaddy, Namecheap, etc.)
2. Add a CNAME record:
   - **Name**: `www`
   - **Type**: `CNAME`
   - **Value**: `mywebsite-static-hosting.s3-website.ap-south-1.amazonaws.com`
   - **TTL**: 3600

### Using Route 53 (AWS DNS)
1. Create a hosted zone in Route 53
2. Create an A record with "Alias" pointing to S3 endpoint
3. Update nameservers at your domain registrar

---

## 🔒 Security Best Practices

### 1. Enable Bucket Versioning
Protects against accidental deletions and overwrites.

1. Go to bucket **Properties**
2. Find **Bucket Versioning**
3. Click **Edit** → **Enable**
4. Save changes

### 2. Monitor Access
1. Go to **Properties** → **Server access logging**
2. Enable logging to track who accesses your site

### 3. Restrict by IP (Optional)
Add condition to bucket policy to only allow specific IPs:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": "YOUR-IP-ADDRESS/32"
                }
            }
        }
    ]
}
```

---

## 🐛 Troubleshooting

### ❌ 403 Forbidden Error
**Cause**: Files aren't publicly accessible

**Fix**:
1. Check bucket policy is applied correctly
2. Verify "Block all public access" is turned OFF
3. Ensure files are uploaded
4. Check object ACLs (should allow public read)

### ❌ 404 Not Found Error
**Cause**: File doesn't exist or wrong path

**Fix**:
1. Verify `index.html` exists in bucket root
2. Check filename spelling (case-sensitive!)
3. Confirm index document is set to `index.html`
4. Make sure file uploaded successfully

### ⚠️ Website Not Loading
**Cause**: Using wrong URL

**Fix**:
- Use the **website endpoint**, NOT the bucket endpoint
- ✅ Correct: `http://bucket-name.s3-website-region.amazonaws.com`
- ❌ Wrong: `https://bucket-name.s3.amazonaws.com`

### 🔄 Changes Not Showing
**Cause**: Browser cache

**Fix**:
1. Hard refresh: `Ctrl + F5` (Windows) or `Cmd + Shift + R` (Mac)
2. Clear browser cache
3. Try incognito/private browsing mode

### 💸 Unexpected Costs
**Cause**: High traffic or large files

**Fix**:
1. Check CloudWatch metrics for request counts
2. Set up billing alerts
3. Optimize images (compress, use WebP)
4. Consider CloudFront if traffic is high

---

## 📊 Monitoring Your Website

### CloudWatch Metrics
1. Go to **Metrics** tab in your S3 bucket
2. View:
   - Number of requests
   - Bytes downloaded
   - 4xx/5xx errors

### Set Up Billing Alerts
1. Go to **AWS Billing Dashboard**
2. Create alert for budget threshold (e.g., $5/month)
3. Receive email if costs exceed limit

---

## 🚀 Optional Enhancements

### 1. Error Page
Create custom `404.html` page:
1. Go to **Properties** → **Static website hosting**
2. Set **Error document** to `404.html`
3. Upload your custom 404.html file

### 2. Optimize Images
```bash
# Compress images before uploading
# Use tools like TinyPNG, ImageOptim
# Or convert to WebP format for smaller size
```

### 3. Add Cache Headers
```bash
# Set cache control when uploading
aws s3 cp styles.css s3://mywebsite-static-hosting/ \
  --cache-control "public, max-age=31536000"
```

### 4. Backup Your Bucket
```bash
# Download entire bucket
aws s3 sync s3://mywebsite-static-hosting/ ./backup/
```

---

## 🎨 File Organization Tips

Organize your bucket for better management:

```
mywebsite-static-hosting/
│
├── index.html
├── styles.css
├── images/
│   ├── logo.png
│   ├── hero.jpg
│   └── icon.svg
├── js/
│   └── script.js
└── fonts/
    └── custom-font.woff2
```

Upload folders:
```bash
aws s3 sync ./images s3://mywebsite-static-hosting/images/
```

---

## 📱 Testing Your Website

### Check Different Devices
- Desktop browser
- Mobile phone
- Tablet
- Different browsers (Chrome, Firefox, Safari)

### Use Browser DevTools
- Check for broken links
- Verify images load
- Check console for errors
- Test responsive design

### Online Tools
- [Google PageSpeed Insights](https://pagespeed.web.dev/)
- [GTmetrix](https://gtmetrix.com/)
- Check website speed and optimization

---

## ✅ Deployment Checklist

- [ ] AWS account created and logged in
- [ ] S3 bucket created with unique name
- [ ] "Block all public access" unchecked
- [ ] Static website hosting enabled
- [ ] Index document set to `index.html`
- [ ] All files uploaded (HTML, CSS, images)
- [ ] Bucket policy configured for public read
- [ ] Website endpoint URL noted
- [ ] Website tested and accessible
- [ ] (Optional) Bucket versioning enabled
- [ ] (Optional) Custom domain configured

---

## 📚 Useful AWS CLI Commands

```bash
# List all files in bucket
aws s3 ls s3://mywebsite-static-hosting/

# Delete a file
aws s3 rm s3://mywebsite-static-hosting/old-file.html

# Copy file with public read ACL
aws s3 cp index.html s3://mywebsite-static-hosting/ --acl public-read

# Make entire bucket public
aws s3api put-bucket-acl --bucket mywebsite-static-hosting --acl public-read

# Get bucket website configuration
aws s3api get-bucket-website --bucket mywebsite-static-hosting

# Download entire bucket
aws s3 sync s3://mywebsite-static-hosting/ ./local-backup/
```

---

## 🎓 Next Steps

Once comfortable with S3:
1. **Add HTTPS**: Consider CloudFront for SSL
2. **Automate**: Set up GitHub Actions for auto-deployment
3. **Analytics**: Add Google Analytics
4. **SEO**: Optimize meta tags and sitemap
5. **Performance**: Compress and minify assets

---

## 📞 Getting Help

- **AWS Documentation**: [S3 Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- **AWS Support**: Check the [AWS Support Center](https://console.aws.amazon.com/support/)
- **Community**: [AWS Forums](https://forums.aws.amazon.com/)
- **Stack Overflow**: Tag your questions with `amazon-s3`

---

## 🎉 Congratulations!

Your static website is now hosted on AWS S3! 

**Your website URL**: `http://mywebsite-static-hosting.s3-website-ap-south-1.amazonaws.com`

Share it with the world! 🌍
