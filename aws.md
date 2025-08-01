# AWS Services: Advanced Deep Dive

This guide provides an in-depth breakdown of major AWS services, covering their advanced features, architecture patterns, integrations, and real-world use cases.

---

## Table of Contents

1. [EC2 (Elastic Compute Cloud)](#ec2-elastic-compute-cloud)
2. [Lambda (Serverless Compute)](#lambda-serverless-compute)
3. [Elastic Beanstalk](#elastic-beanstalk)
4. [ECS & EKS](#ecs--eks)
5. [S3 (Simple Storage Service)](#s3-simple-storage-service)
6. [EBS & EFS](#ebs--efs)
7. [VPC (Virtual Private Cloud)](#vpc-virtual-private-cloud)
8. [Route 53](#route-53)
9. [CloudFront](#cloudfront)
10. [API Gateway](#api-gateway)
11. [RDS & Aurora](#rds--aurora)
12. [DynamoDB](#dynamodb)
13. [ElastiCache](#elasticache)
14. [IAM (Identity and Access Management)](#iam-identity-and-access-management)
15. [KMS, Cognito, GuardDuty](#kms-cognito-guardduty)
16. [CloudWatch, CloudTrail, Config](#cloudwatch-cloudtrail-config)
17. [CodeCommit, CodeBuild, CodeDeploy, CodePipeline](#codecommit-codebuild-codedeploy-codepipeline)
18. [Athena, Redshift, EMR, SageMaker](#athena-redshift-emr-sagemaker)
19. [SNS, SQS, Step Functions, CloudFormation](#sns-sqs-step-functions-cloudformation)

---

## 1. EC2 (Elastic Compute Cloud)

**Purpose:** Provision scalable virtual machines (instances).

**Advanced Concepts**:
- Instance Types: General purpose, compute-optimized, memory-optimized, GPU, etc.
- Auto Scaling: Automatically increase/decrease instances based on load.
- Placement Groups: Cluster, spread, partition for high availability or performance.
- Spot Instances: Bid unused capacity at reduced rates.
- Elastic IPs: Static IPv4 addresses for dynamic cloud computing.
- Security Groups & NACLs: Firewall and subnet-level controls.

**Diagram:**
```
+-----------+        +---------------+
|  Internet |<-----> |  Load Balancer|
+-----------+        +---------------+
                            |
        +-------------------+-------------------+
        |                   |                   |
  +-----------+       +-----------+       +-----------+
  |  EC2-1    |       |  EC2-2    |  ...  |  EC2-N    |
  +-----------+       +-----------+       +-----------+
                   (Auto Scaling Group)
```

**Use Cases:** Web servers, batch processing, legacy app hosting, custom AMIs, multi-tier architectures.

---

## 2. Lambda (Serverless Compute)

# AWS Lambda & API Gateway Deep Dive with Example, Security Best Practices & Instructional Diagram

This guide covers:
- What is AWS Lambda?
- How to expose Lambda via API Gateway (step-by-step)
- A sample Lambda function and API Gateway integration
- Security best practices
- Visual diagram (ASCII and downloadable image)
- Step-by-step instruction

---

## 1. What is AWS Lambda?

AWS Lambda is a serverless compute service that runs your code in response to events and automatically manages the underlying compute resources.

- **No server management**
- **Automatic scaling**
- **Pay only for usage**
- **Event-driven**

---

## 2. How Lambda & API Gateway Work Together

API Gateway acts as a “front door” for applications to access data, business logic, or functionality from Lambda functions.

### Architecture Diagram (ASCII)

```
+---------+    HTTPS   +-------------------+   Invoke   +---------------+
|  Client | ---------> |  API Gateway      | ---------> | Lambda        |
+---------+            | - Auth (Cognito)  |            | - Least Priv. |
                       | - Rate Limiting   |            | - Logs        |
                       | - WAF             |            | - VPC (opt)   |
                       +-------------------+            +---------------+
                              |                                 |
                              v                                 v
                        [CloudWatch]                      [Other AWS Services]
```

---

### Architecture Diagram (Image)

You can use this open-source diagram, which does not require AWS account authentication and is free to use:

![Lambda API Gateway Integration Diagram](https://raw.githubusercontent.com/aws-samples/aws-serverless-workshops/master/WebApplication/images/architecture-overview.png)

*Source: [AWS Serverless Workshops on GitHub](https://github.com/aws-samples/aws-serverless-workshops)*

If you want to ensure always-available images, download the image and add it to your repo, then reference it via a local path.

---

## 3. Step-by-Step: Exposing Lambda with API Gateway

### Prerequisites

- AWS account
- IAM permissions to create Lambda, API Gateway

---

### Step 1: Create Your Lambda Function

1. Navigate to **AWS Lambda** in the AWS Console.
2. Click **Create function**.
3. Choose **Author from scratch**.
4. Set:
   - Name: `HelloWorldFunction`
   - Runtime: Python 3.x or Node.js 18.x
5. Paste the code below:

**Python Example:**
```python
def lambda_handler(event, context):
    name = event.get("queryStringParameters", {}).get("name", "World")
    return {
        "statusCode": 200,
        "body": f"Hello, {name}!"
    }
```

**Node.js Example:**
```javascript
exports.handler = async (event) => {
    const name = event.queryStringParameters?.name || "World";
    return {
        statusCode: 200,
        body: JSON.stringify({ message: `Hello, ${name}!` }),
    };
};
```

6. Click **Deploy**.

---

### Step 2: Create an API Gateway

1. Navigate to **API Gateway** in the AWS Console.
2. Click **Create API** > REST API (or HTTP API for simpler setup).
3. For REST API: Choose **REST API** > **Build**.
4. Name your API (e.g., `HelloWorldAPI`).

---

### Step 3: Connect Lambda to API Gateway

1. In your API, create a **Resource** (e.g., `/hello`).
2. Add a **GET Method**.
3. Choose **Lambda Function** as integration type.
4. Select your Lambda function (e.g., `HelloWorldFunction`).
5. Grant permission when prompted.

---

### Step 4: Deploy API & Test

1. Click **Actions > Deploy API**.
2. Create a new stage (e.g., `prod`).
3. Copy the **Invoke URL**.
4. Test with your browser or curl:

```bash
curl "https://<api-id>.execute-api.<region>.amazonaws.com/prod/hello?name=Alice"
```

**Response:**
```json
{"message":"Hello, Alice!"}
```

---

### Step 5: Secure Your API

- **Use IAM roles with least privilege for Lambda**
- **Enable API Gateway authentication**
   - Use API Keys, Cognito User Pools, or Lambda Authorizer
- **Enable AWS WAF** (Web Application Firewall) for API Gateway
- **Use HTTPS only**
- **Enable request validation and throttling**
- **Encrypt environment variables with KMS**
- **Monitor with CloudWatch**

---

## 4. Security Best Practices Checklist

- [x] Lambda uses an IAM role with **only required permissions**
- [x] API Gateway uses **authentication/authorization** (API Key, Cognito, or Lambda Authorizer)
- [x] Enable **logging** in both Lambda and API Gateway
- [x] Validate all **inputs** in Lambda
- [x] Use **VPC** for Lambda if accessing private resources
- [x] Enable **encryption** for environment variables
- [x] Set appropriate **timeout and memory** for Lambda
- [x] Use **rate limiting and throttling** in API Gateway
- [x] Protect API with **WAF**

---

## 5. References

- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [API Gateway Lambda Proxy Integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-integrations.html)
- [Lambda Security Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [API Gateway Security](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-control-access-to-api.html)

---

*For more screenshots and step-by-step images, see the [AWS official getting started guide for Lambda & API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/getting-started.html).*


---

## 3. Elastic Beanstalk

**Purpose:** Platform as a Service (PaaS) for rapid deployment of web apps.

**Advanced Concepts:**
- Supports multiple languages/runtimes: Node.js, Python, Java, .NET, Go, Ruby, Docker, etc.
- Environment Tiers: Web server vs. worker.
- Configuration: Managed via config files (.ebextensions) for fine-tuning.
- Blue/Green Deployments: Minimize downtime during app upgrades.

**Diagram:**
```
[ Developer Code ] --> [ Elastic Beanstalk ] --> [ EC2, Load Balancer, RDS, etc. (managed) ]
```

**Use Cases:** Web applications, REST APIs, background jobs.

---

## 4. ECS & EKS

### ECS (Elastic Container Service)

**Purpose:** Run Docker containers at scale.

**Advanced Concepts:**
- Launch Types: EC2 (self-managed), Fargate (serverless).
- Task Definitions: Blueprint for running containers.
- Service Auto Scaling, Load Balancers, Service Discovery.

### EKS (Elastic Kubernetes Service)

## AWS EKS Deep Dive

### What is AWS EKS?
AWS Elastic Kubernetes Service (EKS) is a managed Kubernetes service that simplifies deploying, managing, and scaling containerized applications using Kubernetes on AWS infrastructure. EKS removes the burden of managing the Kubernetes control plane by handling cluster management tasks such as patching, scaling, and availability.

### Core Concepts

- **Control Plane**: Managed by AWS, highly available, and scalable, running across multiple Availability Zones (AZs).
- **Worker Nodes**: EC2 instances or AWS Fargate that run your Kubernetes workloads.
- **Networking**: Integrates with AWS VPC, IAM, and security groups for secure, isolated networking.
- **Add-ons**: AWS supports managed add-ons (CoreDNS, kube-proxy, VPC CNI, etc.) for easier cluster operations.

### Key Features

- **Security and Compliance**: Integrates with IAM, supports private clusters, and provides encryption for data at rest and in transit.
- **Scalability**: Clusters can scale to thousands of nodes automatically.
- **High Availability**: Control plane runs across multiple AZs for fault tolerance.
- **Integration**: Works with AWS services like ELB, IAM, CloudWatch, ECR, and more.
- **Upgrades and Maintenance**: AWS manages the Kubernetes version and patching.

---

## Step-by-Step: Creating an Enterprise EKS Cluster using AWS Console

### Prerequisites
- AWS account with necessary permissions (EKS, VPC, IAM, EC2, etc.)
- VPC with at least two subnets in different AZs (for HA)
- IAM role for EKS cluster

### 1. Create or Select a VPC
- Use an existing VPC or create a new one with public/private subnets in at least two AZs.
- Ensure subnets have required tags:  
  - `kubernetes.io/cluster/<cluster-name>: shared`
  - `kubernetes.io/role/elb: 1` (public)
  - `kubernetes.io/role/internal-elb: 1` (private)

### 2. Create an EKS Cluster
1. Go to the **AWS Console** > **EKS** > **Add cluster** > **Create**.
2. Enter a **cluster name**.
3. Choose the **Kubernetes version**.
4. Select the **role** for EKS (must have required permissions: `eks.amazonaws.com`).
5. Select the **VPC** and at least two subnets (preferably private for enterprise workloads).
6. Configure **Cluster endpoint access** (Public, Private, or both).
7. Add optional cluster logging (CloudWatch).
8. Review and click **Create**.

### 3. Configure Node Groups
1. In the cluster page, go to the **Compute** tab > **Add Node Group**.
2. Enter a **name** and select the **node IAM role** (with permissions for EC2, EKS worker nodes).
3. Select **subnets** for the node group (should match with EKS subnets).
4. Choose **instance types** (e.g., m5.large for production).
5. Configure **scaling** settings (min/max/desired nodes).
6. Optionally, enable **remote access** (SSH key).
7. Review and create the node group.

### 4. Configure Networking and Security
- Ensure security groups allow necessary traffic to/from nodes and control plane.
- Use **IAM roles for service accounts** for fine-grained permissions.
- Enable **private endpoint** for enhanced security if required.

### 5. Install and Configure kubectl
- Install `kubectl` and `awscli` locally.
- Update kubeconfig:
  ```bash
  aws eks --region <region> update-kubeconfig --name <cluster-name>
  ```
- Validate cluster:
  ```bash
  kubectl get svc
  ```

### 6. Deploy Add-ons and Apps
- Use AWS EKS console or CLI to install add-ons like CoreDNS, kube-proxy, VPC CNI.
- Deploy applications via manifest files or Helm charts.

### 7. Set Up Logging and Monitoring
- Enable CloudWatch logging for cluster/control plane logs.
- Use AWS Container Insights and Prometheus/Grafana for monitoring.

### 8. Implement Security Best Practices
- Enable IAM Roles for Service Accounts (IRSA).
- Use Kubernetes RBAC.
- Use network policies.
- Enable encryption for secrets.

---

## Enterprise Considerations

- **Multi-AZ Deployments**: Always use subnets in at least two AZs for HA.
- **Private Endpoints**: Restrict API access via VPC endpoints.
- **Logging & Audit**: Enable CloudTrail, CloudWatch, and Kubernetes audit logs.
- **Automation**: Use Infrastructure as Code (CloudFormation/Terraform) for repeatable provisioning.
- **Backup/Recovery**: Use EBS snapshots, Velero, or AWS Backup for data protection.

---

## 5. S3 (Simple Storage Service)

**Purpose:** Object storage for the cloud.

**Advanced Concepts:**
- Storage Classes: Standard, Intelligent-Tiering, Glacier, Deep Archive.
- Versioning & Lifecycle Policies: Retain, archive, or delete objects automatically.
- Event Notifications: Trigger Lambda, SNS, SQS on object events.
- S3 Access Points & VPC Endpoints: Granular and private access.
- Cross-Region Replication.

**Diagram:**
```
+-------+          +-------+
| User  | <----->  |  S3   | <-----> [Lambda/Event Trigger]
+-------+          +-------+
                       |
                 [Multiple Buckets]
```

**Use Cases:** Static website hosting, backups, data lakes, media storage.

---

## 6. EBS & EFS

### EBS (Elastic Block Store)
- Persistent block storage for EC2.
- Snapshots for backup and disaster recovery.
- Encryption at rest and in transit.

### EFS (Elastic File System)
- NFS file system for multiple EC2 instances.
- Scalable, pay-per-use, POSIX-compliant.

**Diagram:**
```
[EC2 Instance] -- [EBS Volume]   (Block Storage)
[EC2-1]
[EC2-2]  \
[EC2-N] ---> [EFS] (Shared File System)
```

**Use Cases:** Databases (EBS), shared files (EFS), web content, development environments.

---

## 7. VPC (Virtual Private Cloud)

# AWS VPC Deep Dive: Public & Private Subnets, IGW, and NAT Gateway

This guide explains AWS VPC architecture in depth, focusing on the differences and relationships between public and private subnets, Internet Gateway (IGW), and NAT Gateway (NGW).

---

## 1. Core Concepts

### What is a VPC?
A **Virtual Private Cloud (VPC)** is an isolated, customizable network in AWS where you launch AWS resources.

### What is a Subnet?
A **subnet** is a range of IP addresses within a VPC. Subnets can be designated as **public** (routable to the internet via IGW) or **private** (not directly routable to the internet).

---

## 2. Public vs Private Subnet: Deep Dive

### Public Subnet
- **Definition:** A subnet whose route table contains a route to the Internet Gateway (IGW).
- **Usage:** Host resources that need direct access to the internet (e.g., web servers, load balancers, NAT Gateway itself).
- **Requirement:** Resources (like EC2) must have a public IP or Elastic IP to be accessible from the internet.

### Private Subnet
- **Definition:** A subnet **without** a direct route to the IGW. Instead, it may have a route to a NAT Gateway for outbound internet.
- **Usage:** Host resources that should not be directly accessed from the internet (e.g., databases, application servers).
- **Requirement:** No public IPs are assigned to instances here.

---

## 3. Internet Gateway (IGW) and NAT Gateway (NGW)

### Internet Gateway (IGW)
- **Purpose:** Allows communication between resources in your VPC and the internet.
- **Attachment:** One per VPC.
- **Subnet Placement:** IGW itself is not “placed” in a subnet; it is attached at the VPC level, but only subnets with a route to the IGW are considered public.

### NAT Gateway (NGW)
- **Purpose:** Allows resources in private subnets to initiate outbound connections to the internet (e.g., for software updates) but prevents inbound internet connections.
- **Deployment:** Must be launched in a **public subnet** (where it can access the IGW).
- **Elastic IP:** Requires an Elastic IP address.

---

## 4. Deep Dive: Routing

### Route Table for Public Subnet
| Destination | Target    | Purpose                |
|-------------|-----------|------------------------|
| VPC CIDR    | local     | Internal communication |
| 0.0.0.0/0   | igw-xxxx  | Internet access        |

### Route Table for Private Subnet
| Destination | Target      | Purpose                       |
|-------------|-------------|-------------------------------|
| VPC CIDR    | local       | Internal communication        |
| 0.0.0.0/0   | nat-xxxx    | Outbound internet via NAT GW  |

---

## 5. Can You Place IGW or NGW in a Subnet?

- **IGW:** Cannot be “placed” in a subnet. It is attached at the VPC level, and the subnet’s route table determines if it is public.
- **NAT Gateway:** Must be launched in a **public subnet** (because it needs a route to the IGW for outbound internet access).  
  **You cannot place a NAT Gateway in a private subnet**—it will not work.

---

## 6. Example Architecture and Diagram

### CIDR Block: `10.0.0.0/16`
- **Public Subnet:** `10.0.1.0/24` (for IGW, NAT GW, web servers)
- **Private Subnet:** `10.0.2.0/24` (for app servers, DB)

```
                        (Internet)
                             |
                          [IGW]
                             |
            +----------------+----------------+
            |                                 |
    [Public Subnet]                   [Private Subnet]
     10.0.1.0/24                        10.0.2.0/24
        |                                  |
   +----------+                       +----------+
   | NAT GW   |<----------------------+|  EC2     |
   | (with EIP)|   (Outbound only)    | (private) |
   +----------+                       +----------+
        |
   +----------+
   |  EC2     | (web server, optional)
   | (public) |
   +----------+
```

- **Route Table Public Subnet:**  
  `0.0.0.0/0` → `IGW`
- **Route Table Private Subnet:**  
  `0.0.0.0/0` → `NAT Gateway` (NAT GW in public subnet)

---

## 7. Detailed Steps: How to Build

1. **Create a VPC** (e.g., `10.0.0.0/16`)
2. **Create two subnets:**
   - Public: `10.0.1.0/24`
   - Private: `10.0.2.0/24`
3. **Create and attach an IGW** to the VPC.
4. **Create a route table for the public subnet:**
   - Add route: `0.0.0.0/0` → IGW.
   - Associate with the public subnet.
5. **Launch a NAT Gateway** in the public subnet:
   - Allocate an Elastic IP and associate it.
6. **Create a route table for the private subnet:**
   - Add route: `0.0.0.0/0` → NAT Gateway.
   - Associate with the private subnet.
7. **Launch EC2 instances:**
   - Public subnet: assign a public IP (for web server, NAT GW).
   - Private subnet: do **not** assign a public IP.

---

## 8. Key Points and Gotchas

- **IGW is VPC-wide, not per subnet; subnets are public if they have a route to IGW.**
- **NAT Gateway is always deployed in a public subnet.**
- **Private subnets never have direct routes to IGW.**
- **NAT Gateway enables only outbound internet from private subnets.**
- **Do not put sensitive resources (DB, etc.) in public subnets.**

---

## 9. References

- [AWS VPC documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Introduction.html)
- [NAT Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [VPC Scenario: Public and Private Subnets](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)

---
---

## 8. Route 53

**Purpose:** DNS and traffic management.

**Advanced Concepts:**
- Routing Policies: Simple, weighted, latency, failover, geolocation, multi-value answer.
- Health Checks: Route traffic away from unhealthy endpoints.
- Domain Registration: Manage domain names.

**Diagram:**
```
[User] --> [Route 53] --> [ELB, EC2, S3 Website, CloudFront, etc.]
```

---

## 9. CloudFront

**Purpose:** Content Delivery Network (CDN) for performance and security.

**Advanced Concepts:**
- Origin Types: S3, EC2, ELB, custom origins.
- Edge Locations: Global cache points.
- Signed URLs/Cookies: Secure content distribution.
- Lambda@Edge: Run functions at CDN edge.

**Diagram:**
```
[User]
  |
[Edge Location (CloudFront)]
  |
[Origin (S3/EC2/etc.)]
```

---

## 10. API Gateway

**Purpose:** Build, deploy, and secure APIs at scale.

**Advanced Concepts:**
- REST, HTTP, and WebSocket APIs.
- Request/Response Transformation.
- Usage Plans, API Keys, Throttling, Caching.
- Lambda Authorizers, Cognito integration for auth.

**Diagram:**
```
[Client] --> [API Gateway] --> [Lambda/ECS/EC2/RDS/Other]
```

---

## 11. RDS & Aurora

### RDS (Relational Database Service)
- Managed MySQL, PostgreSQL, MariaDB, Oracle, SQL Server.
- Automated backups, snapshots, Multi-AZ deployments.

### Aurora
- AWS-built relational DB compatible with MySQL/PostgreSQL.
- Auto-scaling, replication, high performance.

**Diagram:**
```
[App] --> [RDS/Aurora Primary] --> [Read Replicas]
```

---

## 12. DynamoDB

**Purpose:** Fully managed NoSQL database.

**Advanced Concepts:**
- Partition/Sort Keys, Global and Local Secondary Indexes.
- Streams for change data capture.
- On-demand & provisioned capacity, auto scaling.
- Transactions and DynamoDB Accelerator (DAX) for caching.

**Diagram:**
```
[App] <--> [DynamoDB Table] <--> [Streams, DAX]
```

---

## 13. ElastiCache

**Purpose:** Managed Redis/Memcached clusters for in-memory caching.

**Advanced Concepts:**
- Replication, cluster mode, backup/restore.
- Pub/Sub, data persistence (Redis).
- Enhanced security with VPC.

**Diagram:**
```
[App] <--> [ElastiCache Cluster] <--> [DB (RDS/DynamoDB)]
```

---

## 14. IAM (Identity and Access Management)

**Purpose:** Securely manage access to AWS resources.

**Advanced Concepts:**
- Users, Groups, Roles, Policies.
- Federated Identity: SAML, OIDC, Cognito.
- Service Roles, Resource-based Policies.
- MFA, Permission Boundaries, Policy Simulator.

**Diagram:**
```
[User/Admin] --> [IAM] --> [AWS Resources]
```

---

## 15. KMS, Cognito, GuardDuty

- **KMS:** Encryption key management, integrated with S3/EBS/RDS.
- **Cognito:** User authentication, SSO, social login, user pools.
- **GuardDuty:** Threat detection and continuous monitoring.

---

## 16. CloudWatch, CloudTrail, Config

- **CloudWatch:** Metrics, logs, alarms, dashboards.
- **CloudTrail:** API call auditing.
- **Config:** Resource inventory, change tracking, compliance auditing.

---

## 17. CodeCommit, CodeBuild, CodeDeploy, CodePipeline

- **CodeCommit:** Managed Git repositories.
- **CodeBuild:** Build/test service.
- **CodeDeploy:** Automated deployments.
- **CodePipeline:** Orchestrate CI/CD workflows.

**Diagram:**
```
[Push Code] --> [CodeCommit] --> [CodeBuild] --> [CodeDeploy] --> [Prod/Staging]
                        \_______________________________________/
                                 [CodePipeline]
```

---

## 18. Athena, Redshift, EMR, SageMaker

- **Athena:** Serverless SQL queries on S3 data.
- **Redshift:** Petabyte-scale data warehouse.
- **EMR:** Managed Hadoop/Spark clusters.
- **SageMaker:** End-to-end ML: data prep, training, deployment.

---

## 19. SNS, SQS, Step Functions, CloudFormation

- **SNS:** Pub/Sub, push notifications.
- **SQS:** Message queuing, decoupling microservices.
- **Step Functions:** Orchestrate workflows across AWS services.
- **CloudFormation:** Infrastructure as Code (IaC).

---

## Further Reading

- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Service Documentation](https://docs.aws.amazon.com/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)

---

## License

MIT License
