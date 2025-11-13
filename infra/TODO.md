# TODO — MasjidPanel Infrastructure

## 🧹 Caching and Performance Tuning

- [ ] Split cache control:
  - HTML/XML/TXT → `Cache-Control: no-cache`
  - Assets → `public,max-age=31536000,immutable`
- [ ] Implement optional cache header update:
    ```bash
    aws s3 cp s3://masjidpanel.com/ s3://masjidpanel.com/ --recursive       --exclude "*" --include "*.html" --metadata-directive REPLACE --cache-control "no-cache"
    ```
- [ ] Optional: Brotli/gzip static asset compression before upload.

---

## 📈 Logging and Monitoring

- [ ] Enable CloudFront Access Logs → S3  
- [ ] Enable S3 Server Access Logging  
- [ ] Add CloudWatch alarms (5xx, latency)

---

## 🔄 Automation Improvements

- [ ] Move inline buildspec → `buildspec.yml` in repo
- [ ] Optional multi-stage CodePipeline
- [ ] Notifications via SNS/Slack webhook

---

## 🛡️ Security Hardening

- [ ] Add AWS Config rule to detect public buckets
- [ ] Rotate CodeBuild role every 90 days
- [ ] (Optional) Add AWS WAF for DDoS protection

---

## 🗂 Tagging and Documentation

- [ ] Add tags (`Project=MasjidPanel`, `Env=Prod`)
- [ ] Document CloudFront + Route 53 architecture screenshots

