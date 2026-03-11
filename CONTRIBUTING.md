# Contributing to AWS-Projects

Thank you for your interest in contributing to this AWS CloudFormation infrastructure project! 🎉

---

## 🤝 How to Contribute

### Reporting Issues

If you find a bug or have a suggestion:

1. Check if the issue already exists in [Issues](https://github.com/rman90/AWS-Projects/issues)
2. If not, create a new issue with:
   - Clear title and description
   - Steps to reproduce (for bugs)
   - Expected vs. actual behavior
   - AWS region and CLI version
   - Relevant CloudFormation error messages

### Suggesting Enhancements

We welcome ideas for improvements:

- New AWS services integration
- Architecture improvements
- Documentation enhancements
- Cost optimization strategies
- Security hardening recommendations

### Pull Requests

1. **Fork the repository**
   ```bash
   git clone https://github.com/rman90/AWS-Projects.git
   cd AWS-Projects
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes**
   - Follow existing code style
   - Update documentation
   - Test your changes in AWS

3. **Validate CloudFormation templates**
   ```bash
   aws cloudformation validate-template \
     --template-body file://your-template.yaml
   ```

4. **Commit your changes**
   ```bash
   git add .
   git commit -m "Add: Brief description of changes"
   ```

5. **Push and create PR**
   ```bash
   git push origin feature/your-feature-name
   ```

---

## 📝 Code Style Guidelines

### CloudFormation Templates

- Use YAML format (not JSON)
- Indent with 2 spaces
- Add comments for complex logic
- Use descriptive resource names
- Include tags on all resources
- Document parameters with descriptions

**Example:**
```yaml
Resources:
  # VPC with DNS support enabled
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      EnableDnsSupport: true
      EnableDnsHostnames: true
      CidrBlock: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: MyVPC
        - Key: ManagedBy
          Value: CloudFormation
```

### Documentation

- Use clear, concise language
- Include code examples
- Add diagrams where helpful
- Keep README up to date
- Use proper markdown formatting

---

## 🧪 Testing

Before submitting a PR:

1. **Validate templates**
   ```bash
   aws cloudformation validate-template \
     --template-body file://your-template.yaml
   ```

2. **Deploy and test**
   ```bash
   aws cloudformation create-stack \
     --stack-name test-stack \
     --template-body file://your-template.yaml
   ```

3. **Verify resources**
   - Check all resources are created
   - Test application functionality
   - Verify outputs are correct

4. **Clean up**
   ```bash
   aws cloudformation delete-stack --stack-name test-stack
   ```

---

## 📋 Checklist

Before submitting your PR, ensure:

- [ ] CloudFormation templates are valid
- [ ] Templates have been tested in AWS
- [ ] Documentation is updated
- [ ] Code follows style guidelines
- [ ] Commit messages are clear
- [ ] No sensitive data (credentials, account IDs) in code
- [ ] Resources have appropriate tags
- [ ] Security best practices are followed

---

## 🔐 Security

- Never commit AWS credentials
- Use IAM roles, not access keys
- Follow least-privilege principle
- Report security issues privately

---

## 📄 License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

## 💬 Questions?

Feel free to open an issue for any questions about contributing!

---

**Thank you for contributing! 🙏**
