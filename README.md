# Portfolio — Hosted on AWS S3 + CloudFront

A single-page responsive portfolio (Hero, About, Projects, Contact), deployed on
AWS S3 and served over HTTPS using CloudFront.

## Live URL
https://d2bil4lnxfrjds.cloudfront.net

## Tech used
- HTML5 + CSS3 (no framework, no build step)
- Google Fonts (Space Grotesk + Inter)
- Font Awesome icons
- AWS S3 for storage/static hosting
- AWS CloudFront as CDN + HTTPS

## Steps I followed

### 1. Built the site locally
`index.html` and `styles.css` — no backend, pure static files.

### 2. Pushed code to GitHub
```bash
git init
git add .
git commit -m "initial portfolio"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

### 3. Created an S3 bucket
- Bucket name: `<your-name>-portfolio` (must be globally unique)
- Region: closest to me (ap-south-1 / Mumbai)
- Unchecked "Block all public access" (since this is a public static site)
- Enabled **Static website hosting** under Properties → set `index.html` as the index document

### 4. Added a bucket policy
Allowed public `s3:GetObject` on the bucket so anyone can read the files:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<your-name>-portfolio/*"
    }
  ]
}
```

### 5. Uploaded the files
```bash
aws s3 sync . s3://<your-name>-portfolio --exclude ".git/*"
```

### 6. Tested the S3 website endpoint
Confirmed the site loaded at the S3 static website URL (HTTP only at this stage).

### 7. Created a CloudFront distribution
- Origin: the S3 bucket (static website endpoint, not the REST API endpoint)
- Viewer protocol policy: **Redirect HTTP to HTTPS**
- Minimum TLS version: TLSv1.2
- Default root object: `index.html`

### 8. Verified HTTPS
Waited for the distribution to deploy (~5–15 min), then opened the CloudFront
domain (`xxxxxx.cloudfront.net`) and confirmed it loads over HTTPS with a valid
padlock.

## Why CloudFront and not just the S3 URL directly?
S3 static website endpoints only serve over HTTP. CloudFront sits in front of
S3, caches content at edge locations closer to users (faster load times), and
is what actually issues the HTTPS certificate. So S3 = storage, CloudFront =
delivery + security layer on top.
