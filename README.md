# AWS CloudFormation — VPC, NAT, EC2 Stack

## 📌 Mục đích

Triển khai hạ tầng AWS bao gồm:
- 1 **VPC**
- 2 **Subnets** (Public & Private)
- 1 **NAT Gateway**
- 2 **Route Tables**
- 2 **EC2 Instances** (1 Public, 1 Private)

## 📂 Cấu trúc template

| **Thành phần** | **Mô tả** |
|----------------|-----------|
| VPC | Mạng riêng (CIDR block /16) |
| Internet Gateway | Kết nối internet |
| Public Subnet | Subnet cho EC2 public |
| Private Subnet | Subnet cho EC2 private |
| NAT Gateway | EC2 private đi internet |
| Route Table (Public) | Route 0.0.0.0/0 qua IGW |
| Route Table (Private) | Route 0.0.0.0/0 qua NAT |
| EC2 Public | VM public, SSH trực tiếp |
| EC2 Private | VM private, SSH qua jump |

## 🛠️ Giải thích các đoạn code chính

### 1️⃣ Tạo VPC

```yaml
VPC:
  Type: AWS::EC2::VPC
  Properties:
    CidrBlock: 10.0.0.0/16
