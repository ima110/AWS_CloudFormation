## CloudFornation Templates

The CrossStack folder contains two CloudFormation templates that are intended to be deployed as separate stacks so one can export values (for example networking outputs) and the other can import them:

- [CrossStack/network.yaml](CrossStack/network.yaml) — defines the network resources (VPC, subnets, route tables, etc.) and should export any values the application stack needs.
- [CrossStack/app.yaml](CrossStack/app.yaml) — defines application resources and is expected to import values exported by the network stack (via Fn::ImportValue).

Deployment notes:
- Deploy the network stack first so its Outputs are available to import.
- Then deploy the app stack which consumes the exported values.

- `CrossStack/network.yaml` — defines the network resources (VPC, subnets, route tables, etc.) and should export any values the application stack needs.
- `CrossStack/app.yaml` — defines application resources and is expected to import values exported by the network stack (via `Fn::ImportValue`).

Deployment notes:
- Deploy the network stack first so its Outputs are available to import.
- Then deploy the app stack which consumes the exported values.


 

## NestedStack folder

The NestedStack folder contains a parent template that deploys a nested child template:

- [NestedStack/main-stack.yaml](NestedStack/main-stack.yaml) — parent stack. Uses an AWS::CloudFormation::Stack resource and references the child template via TemplateURL.
- [NestedStack/vpc-template.yaml](NestedStack/vpc-template.yaml) — child template. Creates a VPC and exports the VpcId as an Output.

Notes:
- TemplateURL in the parent must point to a publicly accessible (or at least CloudFormation-accessible) S3 object. Upload the child template to S3 before creating the parent stack.
- Validate templates locally before upload where possible.

Example (Bash) workflow:

```bash
# Validate child template locally
aws cloudformation validate-template --template-body file://NestedStack/vpc-template.yaml --region us-east-1

# Create S3 bucket (if needed) and upload the child template
aws s3 mb s3://my-cfn-template-780604191148 --region us-east-1
aws s3 cp NestedStack/vpc-template.yaml s3://my-cfn-template-780604191148/vpc-template.yaml

# Validate parent template (parent references the S3 URL)
aws cloudformation validate-template --template-body file://NestedStack/main-stack.yaml --region us-east-1

# Deploy the parent stack (CloudFormation will create the nested stack from the S3 URL)
aws cloudformation deploy \
  --stack-name main-stack \
  --template-file ./NestedStack/main-stack.yaml \