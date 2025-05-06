# NT548 - Bài tập thực hành 01: Triển khai VPC, EC2 với CloudFormation

## Giới thiệu

Dự án này sử dụng AWS CloudFormation để tự động triển khai một môi trường AWS bao gồm VPC, Subnet, Internet Gateway, NAT Gateway, Security Group và EC2 instance. Kiến trúc được thiết kế với một subnet public và một subnet private theo mô hình bảo mật tốt nhất.


### Các thành phần chính:

1. **VPC**: Một mạng ảo riêng biệt với dải địa chỉ 10.0.0.0/16
2. **Public Subnet**: Subnet có thể truy cập từ internet (10.0.1.0/24)
3. **Private Subnet**: Subnet không thể truy cập trực tiếp từ internet (10.0.2.0/24)
4. **Internet Gateway**: Cho phép các tài nguyên trong public subnet kết nối ra internet
5. **NAT Gateway**: Cho phép các tài nguyên trong private subnet kết nối ra internet
6. **Route Tables**: Định cấu hình định tuyến cho các subnet
7. **Security Groups**: Kiểm soát lưu lượng mạng đến và đi từ EC2 instances
8. **EC2 Instances**: Máy chủ ảo trong public và private subnet

## Cách sử dụng

### Yêu cầu tiên quyết

- Tài khoản AWS với quyền truy cập đầy đủ vào các dịch vụ EC2, CloudFormation
- AWS CLI đã được cài đặt và cấu hình (hoặc sử dụng AWS Management Console)
- Key pair đã tạo trong AWS để SSH vào EC2 instances

### Các bước triển khai

#### Sử dụng AWS Management Console

1. Đăng nhập vào AWS Management Console
2. Điều hướng đến dịch vụ CloudFormation
3. Nhấp vào "Create stack" > "With new resources (standard)"
4. Chọn "Upload a template file" và tải lên file `nt548-lab1-cfn.yml`
5. Điền các thông tin sau:
   - **Stack name**: Tên cho stack (Ví dụ: `aws-lab01-cloudformation`)
   - **KeyName**: Tên của key pair bạn đã tạo (mặc định: `cloudformation-lab01`)
   - **MyIP**: Địa chỉ IP public của bạn theo định dạng CIDR (Ví dụ: `125.235.239.47/32`)
6. Nhấp "Next" qua các màn hình tiếp theo
7. Kiểm tra và nhấp "Create stack"

#### Sử dụng AWS CLI

```bash
aws cloudformation create-stack \
  --stack-name aws-lab01-cloudformation \
  --template-body file://nt548-lab1-cfn.yml \
  --parameters \
      ParameterKey=KeyName,ParameterValue=cloudformation-lab01 \
      ParameterKey=MyIP,ParameterValue=125.235.239.47/32
```

### Kết nối tới EC2 Instances

#### Kết nối tới EC2 Public Instance

```bash
ssh -i path/to/cloudformation-lab01.pem ec2-user@[PUBLIC_IP]
```

Trong đó `[PUBLIC_IP]` là địa chỉ IP public của EC2 instance, có thể tìm thấy trong phần Outputs của stack CloudFormation.

#### Kết nối tới EC2 Private Instance (thông qua EC2 Public Instance)

1. Đầu tiên, copy private key sang EC2 Public Instance:
   ```bash
   scp -i path/to/cloudformation-lab01.pem path/to/cloudformation-lab01.pem ec2-user@[PUBLIC_IP]:~/.ssh/
   ```

2. Thiết lập quyền cho private key trên EC2 Public Instance:
   ```bash
   ssh -i path/to/cloudformation-lab01.pem ec2-user@[PUBLIC_IP] "chmod 400 ~/.ssh/cloudformation-lab01.pem"
   ```

3. SSH tới EC2 Public Instance:
   ```bash
   ssh -i path/to/cloudformation-lab01.pem ec2-user@[PUBLIC_IP]
   ```

4. Từ EC2 Public Instance, SSH tới EC2 Private Instance:
   ```bash
   ssh -i ~/.ssh/cloudformation-lab01.pem ec2-user@[PRIVATE_IP]
   ```

Trong đó `[PRIVATE_IP]` là địa chỉ IP private của EC2 instance, có thể tìm thấy trong phần Outputs của stack CloudFormation.

## Giải thích mã CloudFormation

### Parameters

- `KeyName`: Tên key pair EC2 để SSH vào instances
- `MyIP`: Địa chỉ IP public của bạn để giới hạn truy cập SSH
- `LatestAmiId`: Tự động lấy AMI ID mới nhất cho Amazon Linux 2

### Resources

- `MyVPC`: Tạo một VPC với dải địa chỉ 10.0.0.0/16
- `MyInternetGateway` và `AttachGateway`: Tạo và gắn Internet Gateway vào VPC
- `PublicSubnet` và `PrivateSubnet`: Tạo các subnet trong các AZ khác nhau
- `NatGatewayEIP` và `NatGateway`: Tạo NAT Gateway để các tài nguyên trong private subnet có thể truy cập internet
- `PublicRouteTable` và `PrivateRouteTable`: Tạo và cấu hình Route Tables
- `PublicRoute` và `PrivateRoute`: Định cấu hình định tuyến cho các subnet
- `PublicEC2SecurityGroup` và `PrivateEC2SecurityGroup`: Cấu hình Security Groups cho EC2 instances
- `PublicEC2Instance` và `PrivateEC2Instance`: Khởi tạo EC2 instances trong các subnet tương ứng

### Outputs

- `PublicEC2PublicIP`: Địa chỉ IP public của EC2 instance trong Public Subnet
- `PrivateEC2PrivateIP`: Địa chỉ IP private của EC2 instance trong Private Subnet

## Dọn dẹp tài nguyên

Để tránh phát sinh chi phí không mong muốn, hãy xóa stack CloudFormation khi không sử dụng:

```bash
aws cloudformation delete-stack --stack-name aws-lab01-cloudformation
```

hoặc sử dụng AWS Management Console:
1. Điều hướng đến dịch vụ CloudFormation
2. Chọn stack `aws-lab01-cloudformation`
3. Nhấp vào "Delete" và xác nhận

## Vấn đề thường gặp và cách khắc phục

### Stack creation failed (ROLLBACK_IN_PROGRESS)

Kiểm tra tab Events trong CloudFormation để xem lỗi cụ thể. Các lỗi phổ biến bao gồm:

1. **Key Pair không tồn tại**: Đảm bảo key pair đã được tạo trong cùng một region
2. **IP không đúng định dạng**: Đảm bảo IP của bạn có định dạng CIDR đúng (Ví dụ: 125.235.239.47/32)
3. **Không đủ quyền**: Đảm bảo tài khoản AWS của bạn có đủ quyền để tạo các tài nguyên
4. **Giới hạn tài nguyên**: Kiểm tra giới hạn tài nguyên của tài khoản AWS

## Liên hệ

Nếu bạn có bất kỳ câu hỏi hoặc gặp vấn đề, vui lòng liên hệ:
- Email: [22520871@gm.uit.edu.vn]
- SDT: [0839276650]
- [Minh-Nguyen"]
- Hoặc tạo Issue trên repository này