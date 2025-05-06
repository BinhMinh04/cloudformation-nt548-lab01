# AWS CloudFormation VPC Stack

## Mục đích
Triển khai VPC với:
- 1 Public Subnet + EC2
- 1 Private Subnet + EC2
- NAT Gateway, Route tables

## Triển khai
```bash
aws cloudformation create-stack \
  --stack-name devops-vpc-stack \
  --template-body file://Lab01.yaml \
