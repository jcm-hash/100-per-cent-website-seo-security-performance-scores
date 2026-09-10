# Grade A+ Website Security, SEO & Performance By Chris Binnie

Learn how to achieve perfect scores for website security, performance and SEO using AWS S3 and CloudFront, plus some of the gotchas. This article shows real-world examples achieving A+ SSL Labs, 100% PageSpeed and 125/100 Mozilla Observatory scores. I hope you find it useful.

**Last Updated:** 6th July 2026<br>
**Author:** Chris Binnie, cybersecurity consultant, author & writer

---

## Introduction

Achieving enterprise-level website security, performance and SEO scores is not only possible but somewhat surprisingly it's really cost-effective. The information in this guide was hard-won and demonstrates how to build websites that achieve:

- **SSL Labs Grade:** A+
- **Mozilla HTTP Observatory:** 125/100 (the scale awards bonus points above 100)
- **Google PageSpeed:** 100/100 (both mobile and desktop)
- **Google Lighthouse:** 100 for Performance, Best Practices and SEO; 94 for Accessibility (see the 2026 update at the end)
- **Security Headers:** Nine security headers configured (listed in the case study below)

**⚠️ Important:** The code examples and configurations in this guide are only for reference. Improper configuration of security headers, CloudFront Functions or AWS services can cause downtime or introduce security vulnerabilities. Use the information in this article at your own risk; you have been suitably warned!

Update! If you are interested in learning for free about AI Security, Linux & Kubernetes try my new sites, follow the footer links on: [getjailbroken](https://www.getjailbroken.com).

---

This page focuses on two real-world examples ([chrisbinnie.com](https://www.chrisbinnie.com) and [chrisbinnie.co.uk](https://www.chrisbinnie.co.uk)) and following some unwelcome eyestrain shares the architecture, configuration and implementation strategies that deliver enterprise-level results whilst maintaining costs of only around $1-$2 monthly (excluding annual Domain Name fees).

---

## Table of Contents

1. [Creating The Perfect Website: Security, Performance & SEO](#the-perfect-website-trinity)
2. [Architecture Foundation: Why Static Sites Win](#architecture-foundation)
3. [AWS Infrastructure Setup: S3 and CloudFront](#aws-infrastructure-setup)
4. [Security Headers: The Real-World Configuration](#security-headers)
5. [Performance Optimisation: 100% PageSpeed Scores](#performance-optimisation)
6. [SEO Excellence: Technical Foundations](#seo-excellence)
7. [Real-World Case Studies](#real-world-case-studies)
8. [Maintenance and Monitoring](#maintenance-and-monitoring)
9. [Conclusion and Next Steps](#conclusion)

---

## Creating The Perfect Website: Security, Performance & SEO {#the-perfect-website-trinity}

Modern websites face a challenging paradox: it's really tricky to get it right. Users demand instant loading times and seamless experiences, whilst security threats grow increasingly sophisticated. Search engines prioritise both security and performance in their ranking algorithms, making these three pillars—security, performance and SEO—fundamentally interconnected.

### Why Traditional Approaches Fail

Most websites struggle to achieve excellence across all three domains because they:

- **Rely on complex content management systems** with hundreds of dependencies
- **Implement server-side processing** that introduces vulnerabilities and latency
- **Use multiple third-party scripts** that compromise both security and performance
- **Employ frameworks** that add unnecessary bloat and attack surface
- **Depend on shared hosting** with limited control over infrastructure

The solution lies in architectural simplicity: **static sites with modern security headers, global content delivery and zero dependencies**.

### The Three Pillars Explained

**Security** encompasses protecting your website and users from threats including cross-site scripting (XSS), clickjacking, man-in-the-middle attacks and data breaches. Perfect security requires properly configured TLS/SSL, comprehensive HTTP security headers and minimal attack surface.

**Performance** measures how quickly your website loads and becomes interactive. Core Web Vitals—Largest Contentful Paint (LCP), First Input Delay (FID) and Cumulative Layout Shift (CLS)—directly impact user experience and search rankings.

**SEO** (Search Engine Optimisation) determines your website's visibility in search results. Technical SEO requires fast loading times, mobile optimisation, proper schema markup and secure connections—all factors that overlap with security and performance.


---

## Architecture: And Why Static Sites Win {#architecture-foundation}

Static websites represent the optimal architecture for achieving perfect security, performance and SEO scores. Unlike dynamic sites requiring server-side processing, databases and complex application logic, static sites serve pre-rendered HTML, CSS and JavaScript files directly to users. The factors influencing the design were firstly to avoid absolutely all library dependencies (to stop the ever-growing number of supply chain attacks) and secondly show some simple functionality, but keep any javascript or CSS as secure and clean as possible. It was frustrating at times but after the initial headaches the sites work well in most browsers and on both mobiles and desktops.

### The Static Site Advantage

**Security Benefits:**
- **Zero server-side vulnerabilities:** No SQL injection, remote code execution or business logic flaws
- **Minimal attack surface:** Only HTML, CSS and JavaScript—no backend to compromise
- **No authentication complexity:** Eliminates session management vulnerabilities
- **Simplified dependency management:** Zero npm packages means zero CVEs

**Performance Benefits:**
- **Instant serving:** No database queries or server-side rendering
- **Efficient caching:** Static files cache perfectly at CDN edge locations
- **Minimal payload:** Typically 20-50KB total vs 2-3MB for typical CMS sites
- **Fast TTFB:** Time To First Byte under 20ms with proper CDN configuration achieves enterprise-level performance

**SEO Benefits:**
- **Blazing fast loading:** Google prioritises fast sites in rankings
- **Perfect mobile scores:** Lightweight pages excel on mobile devices
- **Reliable uptime:** No server crashes or database failures
- **HTTPS by default:** Modern hosting platforms enforce secure connections

### When Static Sites Work Best

Static sites excel for:
- Portfolio and personal websites
- Documentation and technical guides
- Marketing and landing pages
- Company information sites
- Blog and content platforms (with static site generators)

Static sites may not suit:
- Real-time applications requiring user interaction
- E-commerce platforms with complex inventory management
- Social networks with dynamic user-generated content
- Applications requiring authentication and personalisation

---

## AWS Infrastructure Setup: S3 and CloudFront {#aws-infrastructure-setup}

Amazon Web Services provides the ideal infrastructure for hosting secure, performant static websites through S3 (Simple Storage Service) for storage and CloudFront for global content delivery. The main reason for choosing AWS was the excellent performance and the DDoS protection that CloudFront offers.

### Step 1: S3 Bucket Configuration

Create an S3 bucket with these essential settings:
```bash
# Create S3 bucket (replace with your domain)
aws s3 mb s3://www.your-domain.com --region us-east-1

# Enable versioning for backup and recovery
aws s3api put-bucket-versioning \
  --bucket www.your-domain.com \
  --versioning-configuration Status=Enabled

# Enable server-side encryption
aws s3api put-bucket-encryption \
  --bucket www.your-domain.com \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }'

# Configure bucket policy for CloudFront access
cat > bucket-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AllowCloudFrontAccess",
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity YOUR-OAI-ID"
    },
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::www.your-domain.com/*"
  }]
}
EOF

aws s3api put-bucket-policy \
  --bucket www.your-domain.com \
  --policy file://bucket-policy.json

```

---

## Performance Optimisation: 100% PageSpeed Scores {#performance-optimisation}

Achieving perfect performance scores requires attention to every aspect of page delivery, from initial request to final render.

### HTML Optimisation

Create minimal, semantic HTML:

```html
<!DOCTYPE html>
<html lang="en-GB">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Expert cybersecurity guidance and Linux server security resources">
    <title>Linux Server Security | Chris Binnie</title>
    
    <!-- One small same-origin stylesheet: allowed by style-src 'self' -->
    <link rel="stylesheet" href="/styles.css">
    
    <!-- Favicon -->
    <link rel="icon" href="/favicon.ico" sizes="any">
    <link rel="icon" href="/favicon.svg" type="image/svg+xml">

    <!-- defer: found early by the preload scanner, runs after parsing -->
    <script src="/script.js" defer></script>
</head>
<body>
    <header>
        <h1>Linux Server Security</h1>
    </header>
    
    <main>
        <!-- Content here -->
    </main>
</body>
</html>
```

**Key optimisations:**
- Keep CSS small and on your own origin; over HTTP/2 one extra same-origin request is cheap
- Put scripts in the `<head>` with `defer`: they download early and still run after the page is parsed (better than the old "scripts at the bottom" rule)
- Use system fonts (no third-party font origin to connect to, and nothing for `font-src 'self'` to block)
- Use semantic HTML5 elements
- Implement proper viewport configuration
- Add preconnect hints for external resources

**Watch out:** the popular "inline critical CSS, then load the rest with `media="print" onload="this.media='all'"`" pattern does not work under the Content-Security-Policy shown in the case study below. `style-src 'self'` blocks the inline `<style>` block and `script-src 'self'` blocks the inline `onload` handler, so the stylesheet never applies. If you want inline CSS, add its SHA-256 hash to `style-src`.

### CSS Optimisation

Write efficient, minimal CSS:

```css
/* Use modern CSS features */
:root {
    --primary-colour: #0066cc;
    --spacing: 1rem;
}

/* Mobile-first responsive design */
body {
    font-size: clamp(1rem, 2.5vw, 1.125rem);
    line-height: 1.6;
    max-width: 70ch;
    margin: 0 auto;
    padding: var(--spacing);
}

/* Efficient selectors */
.container {
    display: grid;
    gap: var(--spacing);
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
}

/* Opacity animates on the compositor; no will-change needed */
.fade-in {
    animation: fadeIn 0.3s ease-in;
}

/* Respect users who have asked for less motion. A near-zero duration is
   safer than animation: none, which can leave anything that starts at
   opacity: 0 invisible for exactly these users */
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}

@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}
```

**Performance principles:**
- Keep specificity low so rules are easy to override (selector speed stopped mattering years ago); watch out for bloating!
- Use CSS custom properties for maintainability
- Implement mobile-first responsive design
- Leverage modern layout techniques (Grid, Flexbox)
- Avoid a permanent `will-change`: it costs memory, and opacity/transform animations are already composited. If used at all, add it just before an animation and remove it after
- Honour `prefers-reduced-motion`

### JavaScript Optimisation

Write performant vanilla JavaScript:

```javascript
// Use modern JavaScript features
const initApp = () => {
    // Efficient DOM queries
    const elements = {
        nav: document.querySelector('nav'),
        main: document.querySelector('main')
    };
    
    // Debounced scroll handler
    let ticking = false;
    const handleScroll = () => {
        if (!ticking) {
            window.requestAnimationFrame(() => {
                // Scroll logic here
                ticking = false;
            });
            ticking = true;
        }
    };
    
    // Passive event listeners
    window.addEventListener('scroll', handleScroll, { passive: true });
    
    // Intersection Observer for lazy loading
    const imageObserver = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                const img = entry.target;
                img.src = img.dataset.src;
                imageObserver.unobserve(img);
            }
        });
    });
    
    document.querySelectorAll('img[data-src]').forEach(img => {
        imageObserver.observe(img);
    });
};

// Execute when DOM ready
if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', initApp);
} else {
    initApp();
}
```

**JavaScript best practices:**
- Use `requestAnimationFrame` for smooth animations
- Implement passive event listeners for better scroll performance
- Use native `loading="lazy"` on images; an Intersection Observer with `data-src` is only needed for other lazy work, and hides images from browsers without JavaScript
- Avoid framework overhead with vanilla JavaScript
- Minimise DOM manipulation and reflows

### Image Optimisation

Implement modern image formats and techniques (this route wasn't used, choosing heavily optimised JPGs instead):

```html
<!-- Responsive images with modern formats -->
<picture>
    <source 
        srcset="/images/hero.avif" 
        type="image/avif">
    <source 
        srcset="/images/hero.webp" 
        type="image/webp">
    <img 
        src="/images/hero.jpg" 
        alt="Linux server security" 
        width="1200" 
        height="630"
        fetchpriority="high">
</picture>
```

**Image guidelines:**
- Convert images to WebP or AVIF formats (often much smaller than JPEG at similar quality; check each image, as savings vary)
- Specify explicit width and height to prevent layout shift
- Use `loading="lazy"` only for below-the-fold images; never lazy-load the hero (it delays Largest Contentful Paint), give it `fetchpriority="high"` instead
- Implement responsive images with `srcset`
- Compress images to appropriate quality levels

### CloudFront Compression

Enable CloudFront compression for text resources:

```bash
# Configure compression in CloudFront distribution
aws cloudfront update-distribution \
  --id YOUR-DISTRIBUTION-ID \
  --distribution-config '{
    "DefaultCacheBehavior": {
      "Compress": true,
      "CachePolicyId": "658327ea-f89d-4fab-a63d-7e88639e58f6"
    }
  }'
```

CloudFront automatically compresses:
- HTML files
- CSS stylesheets
- JavaScript files
- JSON and XML data
- SVG images

Compression reduces file sizes by 70-90% for text-based resources. It's a real eye-opener if you do some Brotli and gzip compression tests. In my experience it makes a big difference to the browsing UX.

The `update-distribution` call above is simplified for illustration; the real command needs the *complete* distribution config plus its ETag, so fetch it with `aws cloudfront get-distribution-config`, set `"Compress": true` in the file, then pass it back with `--distribution-config file://config.json --if-match <ETag>`. (Or tick "Compress objects automatically" in the console.)

CDNs compress on the fly at a moderate level to save CPU. Measured in September 2026, the chrisbinnie.com home page (23,932 bytes) arrives as 5,649 bytes of Brotli, which matches Brotli quality 5 locally; maximum quality (11) would give 4,835. Both are far inside the ~14 KB that fits in the first round trip, so for a page this size there is nothing to gain. On a larger page it can be worth uploading pre-compressed files.

---

## SEO Excellence: Technical Foundations {#seo-excellence}

Perfect SEO requires both a technical implementation and content strategy. This section focuses on both technical SEO foundations.

### Schema.org Structured Data

Implement JSON-LD structured data:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "Linux Server Security",
  "url": "https://www.chrisbinnie.com",
  "author": {
    "@type": "Person",
    "name": "Chris Binnie",
    "url": "https://www.chrisbinnie.com",
    "sameAs": [
      "https://www.linkedin.com/in/chrisbinnie",
      "https://github.com/chrisbinnie"
    ]
  },
  "description": "Expert Linux server security guidance and cybersecurity resources",
  "inLanguage": "en-GB"
}
</script>

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Complete Linux Server Security Guide",
  "author": {
    "@type": "Person",
    "name": "Chris Binnie"
  },
  "datePublished": "2025-10-01",
  "dateModified": "2025-10-01",
  "image": "https://www.chrisbinnie.com/images/linux-security.jpg"
}
</script>
```

### Meta Tags and OpenGraph

Implement comprehensive meta tags:

```html
<!-- Primary Meta Tags -->
<title>Linux Server Security: Complete Hardening Guide | Chris Binnie</title>
<meta name="description" content="Expert Linux server security guidance covering Ubuntu, CentOS, Debian hardening, intrusion detection and defence strategies.">
<meta name="author" content="Chris Binnie">
<link rel="canonical" href="https://www.chrisbinnie.com/linux-server-security">

<!-- Open Graph / Facebook -->
<meta property="og:type" content="website">
<meta property="og:url" content="https://www.chrisbinnie.com/linux-server-security">
<meta property="og:title" content="Linux Server Security: Complete Hardening Guide">
<meta property="og:description" content="Expert Linux server security guidance covering Ubuntu, CentOS, Debian hardening, intrusion detection and defence strategies.">
<meta property="og:image" content="https://www.chrisbinnie.com/images/og-image.jpg">
<meta property="og:locale" content="en_GB">

<!-- Twitter -->
<meta property="twitter:card" content="summary_large_image">
<meta property="twitter:url" content="https://www.chrisbinnie.com/linux-server-security">
<meta property="twitter:title" content="Linux Server Security: Complete Hardening Guide">
<meta property="twitter:description" content="Expert Linux server security guidance covering Ubuntu, CentOS, Debian hardening, intrusion detection and defence strategies.">
<meta property="twitter:image" content="https://www.chrisbinnie.com/images/twitter-image.jpg">
```

Two tags often seen in older guides are left out on purpose: `<meta name="keywords">` has been ignored by Google since 2009, and `<meta name="title">` is not a standard tag (the `<title>` element does that job).

### XML Sitemap

Generate and submit an XML sitemap:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
    <url>
        <loc>https://www.chrisbinnie.com/</loc>
        <lastmod>2025-10-01</lastmod>
        <changefreq>monthly</changefreq>
        <priority>1.0</priority>
    </url>
    <url>
        <loc>https://www.chrisbinnie.com/linux-server-security</loc>
        <lastmod>2025-10-01</lastmod>
        <changefreq>weekly</changefreq>
        <priority>0.8</priority>
    </url>
</urlset>
```

Google ignores `<changefreq>` and `<priority>`; an accurate `<lastmod>` is the part that is used, so only change it when the page really changes.

Submit to search engines:
- Google Search Console: https://search.google.com/search-console
- Bing Webmaster Tools: https://www.bing.com/webmasters

### Robots.txt Configuration

Get the basics right and make sure that you create a proper robots.txt file:

```
User-agent: *
Allow: /
Sitemap: https://www.chrisbinnie.com/sitemap.xml

# Block specific paths if needed
Disallow: /private/
Disallow: /admin/
```

---

## Real-World Case Studies {#real-world-case-studies}

Despite it being a single-page website, it boasts enterprise-level SEO, performance and security:

### Case Study: chrisbinnie.com

**Architecture:**
- Amazon S3 bucket for static file storage
- CloudFront distribution with global edge locations
- CloudFront Functions for security header injection
- ACM certificate for TLS/SSL
- DNS managed externally (not using Route 53)

**Content:**
- Ultra-minimal HTML structure
- Essential CSS only
- Vanilla JavaScript (no frameworks)
- Total page weight: 22.8 KB
- Optimised for single-page book promotion

**Test Results:**

| Test | Result |
|---|---|
| SSL Labs (Qualys) rating | A+ (certificate, protocol support, key exchange and cipher strength) |
| Mozilla HTTP Observatory | A+, 125/100, 10 of 10 tests passed |
| PageSpeed Insights: Performance | 100 |
| PageSpeed Insights: Accessibility | 94 |
| PageSpeed Insights: Best Practices | 100 |
| PageSpeed Insights: SEO | 100 |

PageSpeed scores are for both mobile and desktop.

### Security Headers Implemented {#security-headers}

```bash
$ curl -I https://www.chrisbinnie.com

HTTP/2 200 
content-type: text/html
content-length: 22811
server: AmazonS3
via: 1.1 220eccae845bbee6b6bb000837ec3cd0.cloudfront.net (CloudFront)
x-amz-server-side-encryption: AES256

# Security Headers
strict-transport-security: max-age=31536000; includeSubDomains; preload
content-security-policy: default-src 'self'; script-src 'self'; style-src 'self'; font-src 'self'; img-src 'self' data: https:; connect-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self'; upgrade-insecure-requests; require-trusted-types-for 'script'
x-frame-options: DENY
x-content-type-options: nosniff
x-xss-protection: 1
referrer-policy: strict-origin-when-cross-origin
permissions-policy: camera=(), microphone=(), geolocation=(), interest-cohort=()
cross-origin-opener-policy: same-origin
cross-origin-resource-policy: same-origin

# Caching
cache-control: public, max-age=60, must-revalidate
x-cache: Hit from cloudfront
```

Two of these are worth revisiting. `X-XSS-Protection` is deprecated; modern browsers have removed the XSS auditor, and MDN recommends sending `X-XSS-Protection: 0` or omitting it, relying on the CSP instead. `interest-cohort` in `Permissions-Policy` was Google's FLoC, which was abandoned; Chrome logs an "unrecognized feature" warning for it, so it can be dropped.

**(Very) Rough Estimated Monthly Costs:**

| AWS service (estimated) | Monthly cost |
|---|---:|
| S3 storage (1 GB) | $0.023 |
| S3 requests (10,000) | $0.01 |
| CloudFront data transfer (5 GB) | $0.50 |
| CloudFront requests (50,000) | $0.06 |
| CloudFront Functions | $0.01 |
| ACM certificate | $0.00 (free) |
| **Total** | **$0.60** |

With moderate traffic (50K requests): $1-$2 a month.

**Additional Site:** The chrisbinnie.co.uk domain achieves identical security and performance scores using the same architecture and security header configuration.

**Key Insight:** Perfect scores across all testing platforms achieved with minimal infrastructure cost and zero dependencies. Total page weight of 22.8 KB enables instant loading globally via CloudFront edge caching.

---

## Maintenance and Monitoring {#maintenance-and-monitoring}

Maintaining perfect scores requires some ongoing attention, but thankfully not too much! Static sites demand minimal maintenance compared to traditional platforms and should be relatively painless to run.

### Automated Monitoring

Implement automated security checks:

```bash
#!/bin/bash
# monthly-security-audit.sh

DOMAIN="www.chrisbinnie.com"
DATE=$(date +%Y-%m-%d)
REPORT_DIR="./security-reports"

mkdir -p "$REPORT_DIR"

echo "Running security audit for $DOMAIN on $DATE"

# Check SSL Labs grade
echo "Checking SSL configuration..."
# The first call starts a scan and returns "IN_PROGRESS"; poll until READY
curl -s "https://api.ssllabs.com/api/v3/analyze?host=$DOMAIN&startNew=on" > /dev/null
until curl -s "https://api.ssllabs.com/api/v3/analyze?host=$DOMAIN&all=done" \
      | tee "$REPORT_DIR/ssllabs-$DATE.json" | grep -qE '"status":"(READY|ERROR)"'; do
  sleep 30
done

# Check security headers
echo "Checking security headers..."
curl -sI "https://$DOMAIN" > "$REPORT_DIR/headers-$DATE.txt"

# Check for mixed content
echo "Scanning for mixed content..."
wget --spider --recursive --level=2 "https://$DOMAIN" 2>&1 | grep "http://" > "$REPORT_DIR/mixed-content-$DATE.txt"

# Check certificate expiry
echo "Checking certificate expiry..."
echo | openssl s_client -servername "$DOMAIN" -connect "$DOMAIN:443" 2>/dev/null | openssl x509 -noout -dates > "$REPORT_DIR/cert-expiry-$DATE.txt"

echo "Security audit complete. Reports saved to $REPORT_DIR"
```

### Performance Monitoring

Set up continuous performance monitoring using this simple script, alter it in any way that helps. Even just logging the history of scores periodically can help make historical comparisons if something breaks.

```bash
#!/bin/bash
# performance-monitor.sh

DOMAIN="www.chrisbinnie.com"
LIGHTHOUSE_API_KEY="your-api-key"

# Run Lighthouse audit
# The API takes GET, not POST; list each category or only Performance is returned
curl -s \
  "https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=https://$DOMAIN&strategy=mobile&category=performance&category=accessibility&category=seo&category=best-practices&key=$LIGHTHOUSE_API_KEY" \
  -o lighthouse-report.json

# Parse results
PERFORMANCE=$(jq '.lighthouseResult.categories.performance.score * 100' lighthouse-report.json)
ACCESSIBILITY=$(jq '.lighthouseResult.categories.accessibility.score * 100' lighthouse-report.json)
SEO=$(jq '.lighthouseResult.categories.seo.score * 100' lighthouse-report.json)

echo "Performance: $PERFORMANCE%"
echo "Accessibility: $ACCESSIBILITY%"
echo "SEO: $SEO%"

# Alert if scores drop below 95%
if (( $(echo "$PERFORMANCE < 95" | bc -l) )); then
    echo "ALERT: Performance score dropped below 95%" | mail -s "Performance Alert" admin@example.com
fi
```

### Update Procedures

Static sites require minimal updates:

**Monthly tasks:**
- Run security audit script
- Check certificate expiry (ACM auto-renews)
- Verify all security headers present
- Test site on multiple devices and browsers

**Quarterly tasks:**
- Review and update content
- Optimise images further if needed
- Audit third-party resources (if any)
- Review CloudFront distribution settings
- Analyse cost and usage patterns

**Annual tasks:**
- Comprehensive security audit
- Review AWS infrastructure for new features
- Update documentation
- Benchmark against competitors
- Consider new web standards (HTTP/3, etc.)

### Deployment Workflow

Implement efficient content updates:

```bash
#!/bin/bash
# deploy.sh - Simple deployment script

BUCKET="s3://www.chrisbinnie.com"
DISTRIBUTION_ID="YOUR-DISTRIBUTION-ID"

echo "Starting deployment..."

# Sync files to S3 with appropriate cache headers.
# "immutable" is only safe if a file's NAME changes whenever its content does
# (e.g. styles.3f9a1c.css). See the note below.
aws s3 sync ./public/ $BUCKET \
  --delete \
  --cache-control "public, max-age=31536000, immutable" \
  --exclude "*.html" \
  --exclude "sitemap.xml" \
  --exclude "robots.txt"

# HTML files with shorter cache
aws s3 sync ./public/ $BUCKET \
  --delete \
  --cache-control "public, max-age=3600, must-revalidate" \
  --exclude "*" \
  --include "*.html" \
  --include "sitemap.xml" \
  --include "robots.txt"

# Invalidate CloudFront cache for updated content
aws cloudfront create-invalidation \
  --distribution-id $DISTRIBUTION_ID \
  --paths "/*"

echo "Deployment complete!"
```

**A gotcha with `immutable`:** it tells browsers never to re-check a file for a year. A CloudFront invalidation clears the edge, but not the copies already in visitors' browsers. So if `styles.css` keeps the same name and you change it, returning visitors keep the old version for up to a year. Either put a content hash in the filename (and update the HTML reference in the build), or drop `immutable` and use a shorter `max-age` for files whose names don't change.

---

## Conclusion and Next Steps {#conclusion}

In my experience achieving perfect website security, performance and SEO scores is entirely achievable through architectural simplicity, comprehensive security headers and modern hosting infrastructure. The examples of chrisbinnie.com and chrisbinnie.co.uk demonstrate that professional results don't require complex solutions or significant investment.

### Things To Think About

**Architecture:**
- Static sites provide superior security and performance
- AWS S3 and CloudFront offer professional-grade hosting for minimal cost
- Zero dependencies eliminate supply chain vulnerabilities
- Simplicity reduces maintenance overhead

**Security:**
- Comprehensive HTTP security headers are essential
- SSL/TLS configuration must prioritise modern protocols
- Regular auditing validates security posture
- Defence in depth through multiple security layers

**Performance:**
- Minimal page weight (under 30KB) achieves excellent scores
- CloudFront edge caching delivers sub-20ms TTFB globally
- Image optimisation and modern formats crucial
- Vanilla JavaScript outperforms framework overhead

**SEO:**
- Technical SEO depends on speed and security
- Structured data improves search visibility
- Mobile optimisation is non-negotiable
- HTTPS and security headers contribute to rankings

### Implementation Checklist

**Phase 1: Starter For Ten**
- Create AWS account and configure IAM
- Set up S3 bucket with encryption
- Configure CloudFront distribution
- Request ACM certificate
- Configure DNS (external provider)

**Phase 2: Security**
- Implement CloudFront Functions for headers
- Configure HSTS with preload
- Create comprehensive CSP policy
- Test with SSL Labs, Mozilla Observatory, SecurityHeaders
- Validate all security headers present

**Phase 3: Performance**
- Optimise HTML structure
- Minimise CSS and JavaScript
- Convert images to WebP/AVIF
- Enable CloudFront compression
- Test with PageSpeed Insights and Lighthouse

**Phase 4: SEO**
- Implement schema.org structured data
- Configure comprehensive meta tags
- Create and submit XML sitemap
- Set up Google Search Console
- Implement proper robots.txt

**Phase 5: Monitoring**
- Set up monitoring solution
- Create automated security audit script
- Schedule regular performance testing
- Implement deployment workflow
- Document procedures

### Measuring Success

Track these metrics monthly:

**Security Metrics:**
- SSL Labs grade (target: A+)
- Mozilla Observatory score (target: 120/100)
- SecurityHeaders.com grade (target: A+)
- Zero security incidents
- No mixed content warnings

**Performance Metrics:**
- PageSpeed Insights (target: 100/100)
- Lighthouse scores (target: 100% all categories)
- TTFB (target: <50ms)
- Page weight (target: <30KB)
- Core Web Vitals (target: all green)

**Business Metrics:**
- Search ranking positions
- Organic traffic growth
- Page load abandonment rate
- User engagement metrics
- Monthly hosting costs

### Further Reading

**Official Documentation:**
- [AWS S3 Documentation](https://docs.aws.amazon.com/s3/)
- [CloudFront Developer Guide](https://docs.aws.amazon.com/cloudfront/)
- [Mozilla Web Security Guidelines](https://infosec.mozilla.org/guidelines/web_security)
- [Google Web Fundamentals](https://developers.google.com/web)

**Testing Tools:**
- [SSL Labs SSL Test](https://www.ssllabs.com/ssltest/)
- [Mozilla HTTP Observatory](https://observatory.mozilla.org/)
- [Google PageSpeed Insights](https://pagespeed.web.dev/)
- [SecurityHeaders.com](https://securityheaders.com/)
- [WebPageTest](https://www.webpagetest.org/)

**Books by Chris Binnie:**
- *Cloud Native Security* (Wiley, 2021) - Container security and Kubernetes hardening
- *Linux Server Security: Hack and Defend* (Wiley, 2016) - Comprehensive Linux hardening
- *Practical Linux Topics* (Apress, 2015) - Essential Linux administration and security

**Community Resources:**
- [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)
- [Web.dev by Google](https://web.dev/)
- [MDN Web Docs](https://developer.mozilla.org/)

### The End Is Nigh

The journey to perfect website security, performance and SEO scores begins with architectural decisions. By choosing static site hosting on AWS S3 and CloudFront, implementing comprehensive security headers through CloudFront Functions and optimising every aspect of content delivery, you can achieve enterprise-level professional results—for monthly costs of $1-$2.

The real-world examples presented demonstrate that these aren't theoretical ideals but practical, proven architectures delivering exceptional results in production environments. With total monthly costs under $2, zero dependencies to maintain and perfect scores across all testing platforms, this approach represents the optimal balance of security, performance, cost and maintenance overhead.

Whether building a personal portfolio, company website or documentation platform, these principles and implementation strategies provide a roadmap to excellence. Start with the foundation, implement security comprehensively, optimise performance relentlessly and maintain your infrastructure diligently. The result will be a website that excels in every measurable metric whilst requiring minimal ongoing attention and delivering enterprise-level performance.

2026 update: PageSpeed now reports 94% for accessibility, not 96%. Mozilla Observatory now ranks the sites 125/100!

---

## Related Resources

Explore these comprehensive guides for deeper insights into specific topics:

- **[Linux Server Security Guide](https://chrisbinnie.github.io/linux-server-security)** - Complete hardening and protection manual covering Ubuntu, CentOS, RHEL and Debian server security
- **[Cloud Native Security Resources](https://www.chrisbinnie.com)** - Expert guidance on container security, Kubernetes hardening and DevSecOps practices
- **[Cybersecurity Books By Chris Binnie](https://www.chrisbinnie.co.uk)** - Professional insights and practical guides on infrastructure security and threat protection
- **[AWS Infrastructure Security](https://chrisbinnie.github.io/aws-cloud-security)** - Detailed tutorials on securing cloud infrastructure and implementing best practices
- **[Kubernetes Security](https://chrisbinnie.github.io/kubernetes-security)** - Harden your Kubernetes clusters, fixing network policies and RBAC configurations

---

**About the Author**

[Chris Binnie](https://chrisbinnie.github.io/about.html) is a cybersecurity consultant and author with nearly thirty years of experience in information security. He has authored multiple cybersecurity books including *Cloud Native Security* (co-authored with Rory McCune), *Linux Server Security: Hack and Defend* and *Practical Linux Topics*. Based in Edinburgh, Scotland, Chris has been a regular contributor to Linux Magazine and ADMIN Magazine for around 15 years. His practical experience includes founding a colocation business in 2001 to achieve zero downtime through redundant infrastructure and designing cloud-based streaming platforms serving HD video to 77 countries in 2012.

---

**⚠️ Important:** All information in this guide is provided for reference purposes only and should be treated as untested! Before implementing any of these configurations on production systems, thoroughly test them in development environments. Use the information at your own risk.
