# We R Robots Blog Infrastructure

Infrastructure and deployment configuration for a Hugo-based blog deployed to AWS CloudFront.

## Infrastructure Overview

This blog uses the following technology stack:

- **Static Site Generator**: Hugo with m10c theme
- **Hosting**: AWS S3 + CloudFront
- **Security**: AWS WAF Rules
- **Authentication**: Origin Access Controls
- **Deployment**: Automated bash script with AWS CLI
- **URI Handling**: CloudFront Function for URL rewrites

## Infrastructure Components

### AWS Configuration

#### S3
- Bucket configured for static website hosting
- Protected by Origin Access Controls
- Not directly accessible to the public

#### CloudFront
- Distribution configured with S3 origin
- URI rewrite function enabled
- Cache invalidation on deployment

#### Security
- WAF rules for additional protection
- Origin Access Controls for S3 bucket access
- Encrypted credentials for deployment

### CloudFront Function (URI-Rewrite)

The following function handles URL rewriting for clean URLs:

```javascript
function handler(event) {
    var request = event.request;
    var uri = request.uri;
    
    // Check whether the URI is missing a file name.
    if (uri.endsWith('/')) {
        request.uri += 'index.html';
    } 
    // Check whether the URI is missing a file extension.
    else if (!uri.includes('.')) {
        request.uri += '/index.html';
    }

    return request;
}
```

## Deployment Script

The deployment process is automated through a bash script that handles credential management, site building, and AWS deployment:

```bash
#!/bin/bash

# Decrypt AWS credentials and export them
eval $(gpg --decrypt aws_credentials.gpg 2>/dev/null)

# Set AWS CLI configuration
aws configure set aws_access_key_id "$AWS_ACCESS_KEY_ID"
aws configure set aws_secret_access_key "$AWS_SECRET_ACCESS_KEY"
aws configure set default.region us-east-2  # Adjust the region as necessary

# Clear public folder
echo "Nuking Public Directory"
rm -rf public/*

# Create AWS Cloudfront invalidation to clear cache
aws cloudfront create-invalidation --distribution-id $AWS_CLOUDFRONT_DISTRIBUTION_ID --paths "/*"

# Build and minify the Hugo site
echo "Building and minifying the Hugo site..."
hugo --minify

# Deploy with Hugo
echo "Deploying site with Hugo..."
hugo deploy

# Clear AWS CLI configuration for security
aws configure set aws_access_key_id ""
aws configure set aws_secret_access_key ""
aws configure set default.region ""

echo "Deployment complete."
```

### Deployment Process

1. AWS credentials are stored in an encrypted file (`aws_credentials.gpg`)
2. The script decrypts credentials and configures AWS CLI
3. Clears the existing public directory
4. Invalidates CloudFront cache
5. Builds and minifies the Hugo site
6. Deploys to AWS using Hugo's built-in deployment
7. Cleans up credentials for security

### Required Environment Variables

The `aws_credentials.gpg` file should contain:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_CLOUDFRONT_DISTRIBUTION_ID`

## Hugo Configuration

The site configuration (`config.toml`) includes:

```toml
baseURL = 'https://blog.werrobots.cloud'
languageCode = 'en-us'
title = 'We R Robots'

theme = "m10c"

[deployment]
[[deployment.targets]]
  name = "production"
  URL = "s3://we-r-robots-blog?region=us-east-2"
```

## Disclaimers

### Hugo
This project uses Hugo, a powerful open-source static site generator. Hugo is not my project - it is developed and maintained by its own community. For more information about Hugo, visit:
- Hugo Documentation: https://gohugo.io/documentation/
- Hugo GitHub: https://github.com/gohugoio/hugo
- Hugo Community: https://discourse.gohugo.io/

### Deploy Script License (MIT)

The deployment script (`deploy.sh`) is released under the MIT License:

```
MIT License

Copyright (c) 2024 We R Robots

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

The deployment script is provided as-is, without any warranties or guarantees. Users are responsible for their own AWS configurations and costs.