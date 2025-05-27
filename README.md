# BistraWP - Server Object Caching, WebP Rewrite, and Metrics Prep

This stage documents the core enhancements made to the BistraWP Base Tier after setting up the Valkey/Redis `object-cache.php` drop-in, but before beginning the CloudWatch agent install.

---

## ✅ 1. Final Redis Migration: Valkey ➝ Redis OSS

### Problem
- Valkey drop-in caused repeated 404s and `wp_cache_add()` redeclaration errors.
- Needed stable object cache integration with WordPress.

### Solution
- Migrated from Valkey to AWS Redis OSS using ElastiCache.
- Verified working connection using `redis-cli`.

### Test
```bash
redis-cli -h your-redis-host.amazonaws.com -p 6379 keys "*"
```
Expected Output:
```
1) "wp_cache:default:akira_test_key"
```

---

## ✅ 2. Verified Object Caching Integration

### Drop-in Status
- Used a production-grade, REST-compatible `object-cache.php` that doesn’t conflict with WP core.
- No redeclared core functions.
- Compatible with `phpredis`.

### Validation
- Verified cache keys appear on `redis-cli`.
- Admin panel and frontend confirmed faster.

---

## ✅ 3. Lambda@Edge for WebP Image Rewrite

### Goal
Serve `.webp` instead of `.jpg` or `.png` when browser supports it.

### Deployment Steps
- Created a Lambda@Edge function called `CFEdgeWebP`.
- Attached to CloudFront viewer request.
- Rewrites image URLs like `/media/image.jpg` ➝ `/media/image.webp` if `Accept: image/webp` is present.

### Test
```bash
curl -I -H "Accept: image/webp" https://your-cloudfront-domain.com/media/image.jpg
```
Expected headers:
```
content-type: image/webp
```

---

## ✅ 4. AMI Snapshot of BistraWP Base Stack

### Purpose
To save the stable state of our Ubuntu EC2 setup with:
- Connected RDS
- Connected S3 (media offload)
- Redis OSS caching
- CloudFront CDN
- WebP support

### AMI Creation (CLI)
```bash
aws ec2 create-image   --instance-id i-xxxxxxxxxxxxxxx   --name "bistrawp-base-image"   --description "AMI snapshot of BistraWP base with Redis + WebP"
```

---

## ✅ 5. BistraWP Monitoring Planning (Prep Stage)

We prepared to add CloudWatch logging and client-specific monitoring by:
- Researching Amazon CloudWatch Agent for Ubuntu
- Planning per-instance dashboards and metrics per client

---

## ✅ Summary

| Feature             | Status    |
|---------------------|-----------|
| Redis OSS           | ✅ Stable |
| WebP Rewrite        | ✅ Live   |
| S3 Offload          | ✅ Done   |
| CloudFront CDN      | ✅ Active |
| AMI Backup          | ✅ Saved  |
| Monitoring Agent    | 🕓 Next   |

---

## 🔜 Next Steps

1. Install CloudWatch agent on Ubuntu.
2. Configure unified metrics dashboards.
3. Enable per-client logging and cost usage tracking.
