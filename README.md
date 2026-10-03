# AWS Static Website with Automatic Deployment (CI/CD)

A simple website hosted on AWS S3. Every time I push my code to GitHub, it updates the website automatically using GitHub Actions.

**Live site:** http://aws-static-website-yourname-123.s3-website-us-east-1.amazonaws.com

---

## What I built
- A static website (HTML, CSS, JavaScript)
- An AWS S3 bucket that hosts the website
- An IAM user that can only touch that one bucket
- A GitHub Actions pipeline that deploys the site on every push

## Tools used
AWS S3, AWS IAM, AWS Budgets, Git, GitHub, GitHub Actions

---

## Step-by-step: how I did it

### Step 1: Create the website
I made a folder with 3 files:
- `index.html` (the page)
- `style.css` (the design)
- `script.js` (the JavaScript)

### Step 2: Push the code to GitHub
1. Created a new repository on GitHub.
2. Ran these commands in my website folder:

```
git init
git add .
git commit -m "Initial static website"
git branch -M main
git remote add origin https://github.com/Bharath9542530187/-aws-static-website.git
git push -u origin main
```

### Step 3: Set a billing alert
1. AWS Console, then Billing and Cost Management, then Budgets.
2. Created a **Monthly cost budget** of **$5** with an email alert.
3. This warns me if my spending gets high, for example when I later use EC2 or load balancers and forget to turn them off.

### Step 4: Create the S3 bucket
- Bucket type: General purpose
- Namespace: Global
- Region: us-east-1
- Block all public access: turned **off** (a public website needs this)
- Versioning: disabled
- Encryption: SSE-S3 (default)

### Step 5: Upload the files and turn on website hosting
1. Uploaded `index.html`, `style.css` and `script.js` to the bucket.
2. Bucket, Properties tab, **Static website hosting**, Enable, index document `index.html`.

### Step 6: Make the website public
Bucket, Permissions tab, **Bucket policy**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::aws-static-website-yourname-123/*"
    }
  ]
}
```

### Step 7: Create a safe IAM user for GitHub
1. **Policy** (JSON editor). It only allows access to my one bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::aws-static-website-yourname-123"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::aws-static-website-yourname-123/*"
    }
  ]
}
```

2. Created the IAM user `github-actions-user` and attached the policy.
3. Created an **access key** for it (use case: Third-party service).

### Step 8: Save the keys in GitHub Secrets
Repository, Settings, Secrets and variables, Actions. Added 3 secrets:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION` (`us-east-1`)

The keys are never written in my code.

### Step 9: Create the GitHub Actions workflow
Created the file `.github/workflows/deploy.yml`:

```yaml
name: Deploy to S3

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Sync files to S3
        run: |
          aws s3 sync . s3://aws-static-website-yourname-123 \
            --delete \
            --exclude ".git/*" \
            --exclude ".github/*" \
            --exclude "README.md"
```

### Step 10: Test it
1. Changed a line in `index.html` and pushed it.
2. The **Actions** tab showed a green tick.
3. Refreshed the website and the change was live.

### Step 11: Rotate the access key
My first key was exposed by mistake, so I replaced it:
1. Created a new key.
2. Updated the 2 secrets in GitHub.
3. Tested that the pipeline still worked.
4. Deactivated and deleted the old key.

---

## What I learned
- How to host a static website on **AWS S3**
- Why a public website needs a **bucket policy**
- What **IAM** is, and why I give a user only the permissions it needs (least privilege)
- How to use **Git commands** to push code to GitHub
- What **CI/CD** means: every push deploys my site automatically
- How to write a **GitHub Actions** workflow file
- How to store keys safely in **GitHub Secrets** instead of in code
- That if a key is ever shared by mistake, I should **replace it and delete the old one**
- Why a **billing alert** is useful: it warns me before paid services like EC2, load balancers or NAT gateways get expensive
- That IAM is free, but some AWS services charge while they are running
- How to read errors and fix them step by step
