# Architecture Diagrams

This folder contains architecture diagrams for the Daily Agenda infrastructure project.

---

## 📁 Current Diagrams

- **architecture-diagram.txt** - Text-based architecture representation

---

## 🎨 Creating Visual Diagrams

To create professional visual diagrams for this project, follow these steps:

### Option 1: Draw.io (Free, Recommended)

1. **Visit:** https://app.diagrams.net/
2. **Download AWS Icons:**
   - Go to: https://aws.amazon.com/architecture/icons/
   - Download AWS Architecture Icons
3. **Import Icons:**
   - In draw.io: File → Open Library → Select AWS icons
4. **Create Diagram:**
   - Use the text diagram as reference
   - Drag and drop AWS icons
   - Connect with arrows
   - Add labels and descriptions
5. **Export:**
   - File → Export as → PNG
   - Save as `architecture-diagram.png`

### Option 2: Lucidchart (Free Tier Available)

1. **Visit:** https://www.lucidchart.com/
2. **Create Account:** Free tier available
3. **Use AWS Shape Library:**
   - Built-in AWS icons
   - Drag and drop interface
4. **Export:** PNG or PDF format

### Option 3: CloudCraft (AWS-Specific)

1. **Visit:** https://www.cloudcraft.co/
2. **Create Account:** Free tier for basic diagrams
3. **Build 3D Diagrams:**
   - Drag AWS services
   - Automatic connections
   - Cost estimation included
4. **Export:** PNG or PDF

### Option 4: AWS Architecture Icons + PowerPoint/Keynote

1. **Download Icons:** https://aws.amazon.com/architecture/icons/
2. **Open PowerPoint/Keynote**
3. **Import Icons:** Insert → Pictures
4. **Arrange Components:** Based on text diagram
5. **Export:** Save as PNG

---

## 📐 Diagram Requirements

### Architecture Diagram Should Include:

- [ ] VPC boundary (rectangle)
- [ ] Public subnet (rectangle inside VPC)
- [ ] Internet Gateway (icon)
- [ ] EC2 instance (icon)
- [ ] Security Group (dashed rectangle)
- [ ] EBS volume (icon)
- [ ] SSM Parameter Store (icon)
- [ ] Arrows showing data flow
- [ ] Labels for all components
- [ ] CIDR blocks for VPC and subnet
- [ ] Legend (if needed)

### Nested Stack Diagram Should Include:

- [ ] Parent stack (top-level box)
- [ ] Network stack (nested box)
- [ ] Application stack (nested box)
- [ ] Arrows showing dependencies
- [ ] Outputs and inputs labeled
- [ ] Color coding for different layers

### Data Flow Diagram Should Include:

- [ ] User/Browser (start point)
- [ ] Internet
- [ ] Internet Gateway
- [ ] Security Group
- [ ] EC2 Instance
- [ ] Apache/PHP
- [ ] SSM Parameter Store
- [ ] Numbered steps (1, 2, 3...)
- [ ] Arrows showing request/response flow

---

## 🎨 Design Guidelines

### Colors

- **Network Layer:** Blue tones
- **Compute Layer:** Orange tones
- **Security Layer:** Red tones
- **Storage Layer:** Green tones
- **Management Layer:** Purple tones

### Layout

- **Top to Bottom:** Internet → VPC → Subnet → Instance
- **Left to Right:** Request flow
- **Grouped:** Related components together

### Labels

- Clear and concise
- Include resource names
- Show CIDR blocks
- Indicate ports (e.g., "HTTP:80")

---

## 📊 Recommended Diagrams

### 1. High-Level Architecture
**File:** `architecture-diagram.png`
- Shows all infrastructure components
- VPC, subnets, EC2, security groups
- Internet connectivity
- Storage and configuration

### 2. Nested Stack Hierarchy
**File:** `nested-stack-diagram.png`
- Parent stack at top
- Network and application stacks below
- Arrows showing dependencies
- Outputs and inputs labeled

### 3. Data Flow Diagram
**File:** `data-flow-diagram.png`
- User request flow
- HTTP request path
- SSM parameter retrieval
- Response generation
- Numbered steps

### 4. Security Architecture
**File:** `security-diagram.png`
- Security group rules
- Network ACLs
- IAM roles
- Defense in depth layers

### 5. Deployment Process
**File:** `deployment-diagram.png`
- CloudFormation stack creation
- Resource provisioning order
- Bootstrap process
- Application deployment

---

## 📝 Example Diagram Structure

```
┌─────────────────────────────────────────┐
│           Internet                      │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│      Internet Gateway (IGW)             │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  VPC (10.0.0.0/16)                      │
│  ┌────────────────────────────────────┐ │
│  │ Public Subnet (10.0.0.0/24)        │ │
│  │  ┌──────────────────────────────┐  │ │
│  │  │ Security Group               │  │ │
│  │  │  ┌────────────────────────┐  │  │ │
│  │  │  │ EC2 Instance           │  │  │ │
│  │  │  │ - Apache               │  │  │ │
│  │  │  │ - PHP                  │  │  │ │
│  │  │  │ - Daily Agenda App     │  │  │ │
│  │  │  └────────────────────────┘  │  │ │
│  │  └──────────────────────────────┘  │ │
│  └────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

---

## 🔗 Useful Resources

- **AWS Architecture Icons:** https://aws.amazon.com/architecture/icons/
- **AWS Architecture Center:** https://aws.amazon.com/architecture/
- **Draw.io:** https://app.diagrams.net/
- **Lucidchart:** https://www.lucidchart.com/
- **CloudCraft:** https://www.cloudcraft.co/
- **AWS Well-Architected:** https://aws.amazon.com/architecture/well-architected/

---

## 📸 Screenshot Guidelines

If including screenshots:

### CloudFormation Console
- Stack creation page
- Stack events
- Stack outputs
- Resource list

### EC2 Console
- Instance details
- Security group rules
- Network interfaces

### Website
- Daily Agenda homepage
- Agenda items displayed
- Browser developer tools (optional)

---

## 💡 Tips

1. **Keep it Simple:** Don't overcomplicate diagrams
2. **Use Standard Icons:** AWS official icons preferred
3. **Label Everything:** Clear labels prevent confusion
4. **Show Flow:** Use arrows to indicate direction
5. **Color Code:** Use colors to group related components
6. **Add Legend:** If using custom symbols or colors
7. **High Resolution:** Export at least 1920x1080 for clarity
8. **Version Control:** Save source files (not just PNG)

---

## 📋 Checklist

Before finalizing diagrams:

- [ ] All components labeled
- [ ] Arrows show correct direction
- [ ] CIDR blocks included
- [ ] Port numbers shown
- [ ] Colors are consistent
- [ ] Legend included (if needed)
- [ ] High resolution (1920x1080+)
- [ ] File names are descriptive
- [ ] Saved in PNG format
- [ ] Source files backed up

---

**Need Help?** Open an issue on GitHub or refer to the AWS Architecture Center for examples.
