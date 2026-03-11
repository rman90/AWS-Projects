# 🚀 GitHub Repository Setup Guide

This guide will help you push the AWS-Projects repository to your GitHub account.

---

## 📋 Prerequisites

- [ ] GitHub account created (https://github.com)
- [ ] Git installed on your local machine
- [ ] Git configured with your name and email
  ```bash
  git config --global user.name "Ross Nesbitt"
  git config --global user.email "your-email@example.com"
  ```

---

## 🔧 Step 1: Create GitHub Repository

### Option A: Via GitHub Website (Recommended)

1. **Go to GitHub:** https://github.com/rman90
2. **Click:** "New" or "+" → "New repository"
3. **Repository Name:** `AWS-Projects`
4. **Description:** `AWS CloudFormation Infrastructure as Code - Daily Agenda Application`
5. **Visibility:** Public (for portfolio)
6. **Initialize:** 
   - ❌ Do NOT add README (we already have one)
   - ❌ Do NOT add .gitignore (we already have one)
   - ❌ Do NOT add license (we already have one)
7. **Click:** "Create repository"

### Option B: Via GitHub CLI

```bash
# Install GitHub CLI (if not installed)
# macOS: brew install gh
# Linux: See https://cli.github.com/

# Authenticate
gh auth login

# Create repository
gh repo create AWS-Projects --public --description "AWS CloudFormation Infrastructure as Code - Daily Agenda Application"
```

---

## 📦 Step 2: Initialize Local Repository

```bash
# Navigate to project directory
cd /home/ec2-user/environment/AWS-Projects

# Initialize git repository
git init

# Add all files
git add .

# Create initial commit
git commit -m "Initial commit: AWS CloudFormation Daily Agenda Infrastructure

- Non-nested CloudFormation template
- Nested stack architecture (parent, network, application)
- Comprehensive documentation
- Deployment guides and checklists
- Architecture diagrams and explanations
- Quick start guide
- Contributing guidelines"

# Verify commit
git log --oneline
```

---

## 🔗 Step 3: Connect to GitHub

```bash
# Add remote repository
git remote add origin https://github.com/rman90/AWS-Projects.git

# Verify remote
git remote -v

# Set main branch (if needed)
git branch -M main
```

---

## ⬆️ Step 4: Push to GitHub

```bash
# Push to GitHub
git push -u origin main

# Verify push
git status
```

---

## ✅ Step 5: Verify on GitHub

1. **Visit:** https://github.com/rman90/AWS-Projects
2. **Check:**
   - [ ] README.md displays correctly
   - [ ] All folders are present
   - [ ] Files are organized properly
   - [ ] Badges display correctly
   - [ ] Links work

---

## 🎨 Step 6: Enhance Repository (Optional)

### Add Topics/Tags

1. Go to repository page
2. Click "⚙️" next to "About"
3. Add topics:
   - `aws`
   - `cloudformation`
   - `infrastructure-as-code`
   - `iac`
   - `devops`
   - `cloud-engineering`
   - `aws-cloudformation`
   - `nested-stacks`
   - `vpc`
   - `ec2`

### Add Description

In the "About" section:
```
AWS CloudFormation Infrastructure as Code project demonstrating monolithic and nested stack architectures for deploying a Daily Agenda web application on AWS.
```

### Add Website (Optional)

If you deploy the application, add the URL to the repository website field.

### Enable GitHub Pages (Optional)

For documentation hosting:
1. Settings → Pages
2. Source: Deploy from branch
3. Branch: main, /docs
4. Save

---

## 📝 Step 7: Create Additional GitHub Features

### Create Issues Template

```bash
# Create .github directory
mkdir -p .github/ISSUE_TEMPLATE

# Create bug report template
cat > .github/ISSUE_TEMPLATE/bug_report.md << 'EOF'
---
name: Bug Report
about: Report a bug or issue
title: '[BUG] '
labels: bug
assignees: ''
---

**Describe the bug**
A clear description of the bug.

**To Reproduce**
Steps to reproduce:
1. Deploy stack with '...'
2. Access '...'
3. See error

**Expected behavior**
What you expected to happen.

**Environment**
- AWS Region:
- AWS CLI Version:
- CloudFormation Template:

**Additional context**
Any other relevant information.
EOF

# Create feature request template
cat > .github/ISSUE_TEMPLATE/feature_request.md << 'EOF'
---
name: Feature Request
about: Suggest an enhancement
title: '[FEATURE] '
labels: enhancement
assignees: ''
---

**Feature Description**
Clear description of the feature.

**Use Case**
Why is this feature needed?

**Proposed Solution**
How should this be implemented?

**Alternatives Considered**
Other approaches you've thought about.
EOF
```

### Create Pull Request Template

```bash
cat > .github/pull_request_template.md << 'EOF'
## Description
Brief description of changes.

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation update
- [ ] Code refactoring

## Checklist
- [ ] Templates validated
- [ ] Tested in AWS
- [ ] Documentation updated
- [ ] No sensitive data in code

## Testing
Describe how you tested these changes.
EOF
```

### Create GitHub Actions (Optional)

```bash
mkdir -p .github/workflows

cat > .github/workflows/validate-templates.yml << 'EOF'
name: Validate CloudFormation Templates

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Validate Non-Nested Template
        run: |
          aws cloudformation validate-template \
            --template-body file://non-nested-template/daily-agenda-stack.yaml
      
      - name: Validate Parent Stack
        run: |
          aws cloudformation validate-template \
            --template-body file://nested-stacks/parent-stack.yaml
      
      - name: Validate Network Stack
        run: |
          aws cloudformation validate-template \
            --template-body file://nested-stacks/network-stack.yaml
      
      - name: Validate Application Stack
        run: |
          aws cloudformation validate-template \
            --template-body file://nested-stacks/application-stack.yaml
EOF
```

---

## 🏷️ Step 8: Create Releases (Optional)

### Create First Release

```bash
# Tag the current commit
git tag -a v1.0.0 -m "Release v1.0.0: Initial release

Features:
- Non-nested CloudFormation template
- Nested stack architecture
- Comprehensive documentation
- Deployment guides
- Architecture diagrams"

# Push tag to GitHub
git push origin v1.0.0
```

### Create Release on GitHub

1. Go to repository page
2. Click "Releases" → "Create a new release"
3. Choose tag: v1.0.0
4. Release title: "v1.0.0 - Initial Release"
5. Description:
   ```markdown
   ## 🎉 Initial Release
   
   First production-ready release of AWS CloudFormation Daily Agenda Infrastructure.
   
   ### ✨ Features
   - Non-nested CloudFormation template for simple deployments
   - Nested stack architecture for modular infrastructure
   - Complete VPC networking setup
   - EC2 instance with automated bootstrapping
   - SSM Parameter Store integration
   - Comprehensive documentation
   
   ### 📚 Documentation
   - README with architecture diagrams
   - Step-by-step deployment guide
   - Architecture explanation
   - Quick start guide
   - Deployment checklist
   
   ### 🚀 Getting Started
   See [QUICKSTART.md](QUICKSTART.md) for rapid deployment.
   
   ### 📦 What's Included
   - CloudFormation templates (YAML)
   - Documentation (Markdown)
   - Architecture diagrams
   - Deployment scripts
   ```
6. Click "Publish release"

---

## 📊 Step 9: Add Repository Insights

### Enable Insights

1. Go to repository Settings
2. Enable:
   - [ ] Issues
   - [ ] Projects
   - [ ] Wiki (optional)
   - [ ] Discussions (optional)

### Add README Badges

Already included in README.md:
- AWS badge
- Infrastructure as Code badge
- License badge

### Add Additional Badges (Optional)

```markdown
[![GitHub stars](https://img.shields.io/github/stars/rman90/AWS-Projects?style=social)](https://github.com/rman90/AWS-Projects/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/rman90/AWS-Projects?style=social)](https://github.com/rman90/AWS-Projects/network/members)
[![GitHub issues](https://img.shields.io/github/issues/rman90/AWS-Projects)](https://github.com/rman90/AWS-Projects/issues)
[![GitHub last commit](https://img.shields.io/github/last-commit/rman90/AWS-Projects)](https://github.com/rman90/AWS-Projects/commits/main)
```

---

## 🔄 Step 10: Future Updates

### Making Changes

```bash
# Make changes to files
# ...

# Stage changes
git add .

# Commit changes
git commit -m "Update: Brief description of changes"

# Push to GitHub
git push origin main
```

### Creating Branches

```bash
# Create feature branch
git checkout -b feature/new-feature

# Make changes and commit
git add .
git commit -m "Add: New feature description"

# Push branch
git push origin feature/new-feature

# Create pull request on GitHub
```

---

## 📱 Step 11: Share Your Repository

### LinkedIn Post Template

```
🚀 Excited to share my latest AWS project!

I've built a production-ready Infrastructure as Code solution using AWS CloudFormation, demonstrating both monolithic and nested stack architectures.

🔧 Key Features:
• Complete VPC networking setup
• Automated EC2 provisioning
• Modular nested stack design
• Comprehensive documentation
• Production-ready templates

💡 This project showcases:
✅ AWS CloudFormation expertise
✅ Infrastructure as Code best practices
✅ DevOps automation
✅ Cloud architecture design

Check it out on GitHub: https://github.com/rman90/AWS-Projects

#AWS #CloudFormation #InfrastructureAsCode #DevOps #CloudEngineering #IaC
```

### Twitter Post Template

```
🚀 Just published my AWS CloudFormation Infrastructure as Code project!

✨ Features:
• VPC networking
• Nested stacks
• Automated deployment
• Full documentation

Perfect for learning IaC and AWS! 

🔗 https://github.com/rman90/AWS-Projects

#AWS #CloudFormation #DevOps #IaC
```

---

## ✅ Verification Checklist

- [ ] Repository created on GitHub
- [ ] Local repository initialized
- [ ] All files committed
- [ ] Remote added and verified
- [ ] Code pushed to GitHub
- [ ] README displays correctly
- [ ] All links work
- [ ] Topics/tags added
- [ ] Description added
- [ ] License visible
- [ ] Repository is public
- [ ] Issues enabled
- [ ] Release created (optional)
- [ ] Shared on social media (optional)

---

## 🎯 Success Criteria

Your repository is ready when:

✅ All files are visible on GitHub  
✅ README renders correctly with formatting  
✅ Badges display properly  
✅ Links navigate correctly  
✅ Repository appears professional  
✅ Code is well-organized  
✅ Documentation is comprehensive  

---

## 🔗 Useful Commands

```bash
# Check repository status
git status

# View commit history
git log --oneline --graph

# View remote repositories
git remote -v

# Pull latest changes
git pull origin main

# View differences
git diff

# Undo last commit (keep changes)
git reset --soft HEAD~1

# View branches
git branch -a

# Delete local branch
git branch -d branch-name

# Delete remote branch
git push origin --delete branch-name
```

---

## 📞 Need Help?

- **Git Documentation:** https://git-scm.com/doc
- **GitHub Docs:** https://docs.github.com
- **GitHub CLI:** https://cli.github.com/manual/

---

**Congratulations! Your portfolio project is now live on GitHub! 🎉**

**Repository URL:** https://github.com/rman90/AWS-Projects
