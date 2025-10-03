# AWS Retail Store Sample - ECS Minimal CloudFormation

This CloudFormation template deploys the AWS Retail Store Sample application to Amazon ECS using a minimal configuration with in-memory persistence for all services.

## Architecture

The deployment creates:

- **VPC**: Custom VPC with public and private subnets across 2 AZs
- **ECS Cluster**: Fargate cluster with Container Insights enabled
- **5 Microservices**: UI, Catalog, Cart, Checkout, and Orders services
- **Application Load Balancer**: Exposes the UI service publicly
- **Service Discovery**: Private DNS namespace for inter-service communication
- **In-Memory Persistence**: All services use in-memory storage (no external databases)

## Services

| Service  | Image | Port | Persistence | Health Check |
|----------|-------|------|-------------|--------------|
| UI       | retail-store-sample-ui:1.3.0       | 8080 | N/A | /actuator/health |
| Catalog  | retail-store-sample-catalog:1.3.0   | 8080 | in-memory | /health |
| Cart     | retail-store-sample-cart:1.3.0      | 8080 | in-memory | /actuator/health |
| Checkout | retail-store-sample-checkout:1.3.0  | 8080 | in-memory | /health |
| Orders   | retail-store-sample-orders:1.3.0    | 8080 | in-memory | /actuator/health |

## Prerequisites

- AWS CLI configured with appropriate permissions
- CloudFormation permissions to create VPC, ECS, ALB, IAM resources

## Deployment

### Using AWS CLI

```bash
aws cloudformation create-stack \
  --stack-name retail-store-ecs-minimal \
  --template-body file://ecs-retail-store.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters ParameterKey=EnvironmentName,ParameterValue=ecs-retail-store
```

### Using AWS Console

1. Open the AWS CloudFormation console
2. Choose "Create stack" > "With new resources"
3. Upload the `ecs-retail-store.yaml` template
4. Provide stack name and parameters
5. Acknowledge IAM resource creation
6. Create the stack

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| EnvironmentName | ecs-retail-store | Name prefix for all resources |
| ContainerInsightsSetting | enhanced | Container Insights setting (enabled/enhanced/disabled) |
| OpenTelemetryEnabled | false | Enable OpenTelemetry tracing (not implemented in minimal) |

## Outputs

| Output | Description |
|--------|-------------|
| ApplicationURL | URL to access the retail store application |
| ECSClusterName | Name of the created ECS cluster |
| VPCId | ID of the created VPC |
| PrivateSubnets | Comma-separated list of private subnet IDs |
| PublicSubnets | Comma-separated list of public subnet IDs |

## Accessing the Application

After deployment completes (5-10 minutes), access the application using the `ApplicationURL` from the stack outputs:

```bash
aws cloudformation describe-stacks \
  --stack-name retail-store-ecs-minimal \
  --query 'Stacks[0].Outputs[?OutputKey==`ApplicationURL`].OutputValue' \
  --output text
```

## Monitoring

- **CloudWatch Logs**: All service logs are sent to `/retail-store-ecs-minimal-tasks` log group
- **Container Insights**: Enabled by default for cluster-level metrics
- **Health Checks**: Each service has health check endpoints configured

## Cleanup

```bash
aws cloudformation delete-stack --stack-name retail-store-ecs-minimal
```

## Differences from Terraform Version

This CloudFormation template provides the same functionality as the Terraform ECS minimal configuration:

- ✅ Same VPC configuration (10.0.0.0/16 with public/private subnets)
- ✅ Same ECS cluster with Container Insights
- ✅ Same 5 services with in-memory persistence
- ✅ Same ALB configuration exposing UI service
- ✅ Same Service Connect for inter-service communication
- ✅ Same IAM roles and security groups
- ❌ OpenTelemetry support not implemented (parameter exists but not used)

## Cost Considerations

This minimal deployment uses:
- 5 Fargate tasks (1 vCPU, 2GB RAM each)
- 1 Application Load Balancer
- 1 NAT Gateway
- CloudWatch Logs storage