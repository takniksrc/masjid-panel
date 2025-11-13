# MasjidPanel — Static Site (Production)

This repository hosts the **MasjidPanel production website** as a static site, deployed automatically to **AWS S3 + CloudFront** from GitHub.

---

## 🧭 Architecture

| Component | Purpose | Notes |
|------------|----------|-------|
| **Route 53** | DNS management | Hosted zone `masjidpanel.com` |
| **ACM (us-east-1)** | SSL certificate for apex + www | Auto-renewed |
| **S3 (private)** | Stores site assets | Bucket: `masjidpanel.com` |
| **CloudFront + OAC** | Global CDN, HTTPS, redirect www→apex | Distribution `E3GSZN7WHWDZZT` |
| **CodeBuild** | Deploys on push | Inline buildspec, no artifacts |
| **GitHub (main branch)** | Source of truth | Triggers CodeBuild via CodeConnections |

---

## 📁 Repository Layout

```
/infra
    iam-policy.json
    redirect-function.js

/site
    index.html
    robots.txt
    sitemap.xml
    /assets
        ...static assets...
```

Only the **/site** directory is deployed.  
The **/infra** folder is ignored from deployment and safe for internal configs.

Add this to `.gitignore`:

```
infra/
```

---

## ⚙️ Deployment Flow

1. Developer pushes to `main`.
2. CodeBuild runs the inline buildspec:
   - Uploads contents of `/site` → S3 bucket root.
   - Invalidates CloudFront (`/*`)
3. CloudFront serves fresh content globally.

---

## 🚀 BuildSpec Summary (inline)

```yaml
version: 0.3
phases:
  build:
    commands:
      - echo "Static deploy for masjidpanel.com"
  post_build:
    commands:
      - set -e
      - aws s3 sync "$BUILD_DIR" s3://$BUCKET --delete
      - aws cloudfront create-invalidation --distribution-id "$DISTRIBUTION_ID" --paths "/*"
```

Environment variables:
- `BUILD_DIR=site`
- `BUCKET=masjidpanel.com`
- `DISTRIBUTION_ID=E3GSZN7WHWDZZT`

Role permissions:
- `s3:ListBucket`, `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject`
- `cloudfront:CreateInvalidation`

---

## 🌐 Redirect Function (www → apex)

Deployed as CloudFront Function at Viewer Request:

```javascript
function handler(event) {
  var req = event.request;
  var host = req.headers.host.value.toLowerCase();
  if (host === 'www.masjidpanel.com') {
    var loc = 'https://masjidpanel.com' + req.uri;
    if (req.querystring && req.querystring.length > 0) loc += '?' + req.querystring;
    return {
      statusCode: 301,
      statusDescription: 'Moved Permanently',
      headers: { location: { value: loc } }
    };
  }
  return req;
}
```

---

## 🧪 Validation

```bash
curl -I https://masjidpanel.com
curl -I https://www.masjidpanel.com
aws cloudfront list-invalidations --id E3GSZN7WHWDZZT
```

Should see:
- `200 OK` on apex
- `301` from www → apex
- Recent invalidation entries

---

## 🧰 Maintenance

- SSL: Auto-renewed via ACM  
- Deploys: Trigger on push to `main`  
- Invalidations: Auto during deploy  
- S3 bucket: private, OAC-only access

---

Last updated: 2025-10-30
