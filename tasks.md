# DevOps Practice Tasks

Welcome to the comprehensive collection of practice tasks designed to test and improve your skills in AWS, DevOps, Terraform, Linux, ECS, and EKS. Each section contains hands-on tasks that progress from basic to advanced levels.

## Table of Contents
- [AWS Tasks (50)](#aws-tasks)
- [DevOps Tasks (20)](#devops-tasks)  
- [Terraform Tasks (20)](#terraform-tasks)
- [Linux & EC2 Tasks (20)](#linux--ec2-tasks)
- [ECS Tasks (20)](#ecs-tasks)
- [EKS Tasks (20)](#eks-tasks)

**Note**: Solutions are available in the `solutions/` folder with corresponding filenames.

---

## AWS Tasks

### Basic Level (Tasks 1-15)

1. **Account Setup**: Create a new AWS account and configure billing alerts for $10, $50, and $100.

2. **IAM User Creation**: Create an IAM user with programmatic access and attach the `PowerUserAccess` policy.

3. **S3 Bucket Management**: Create an S3 bucket with versioning enabled and upload a file with different versions.

4. **EC2 Instance Launch**: Launch a t3.micro EC2 instance in the default VPC with a custom security group allowing SSH and HTTP.

5. **Key Pair Management**: Create an EC2 key pair and use it to launch an instance, then connect via SSH.

6. **Security Group Configuration**: Create a security group that allows inbound traffic on ports 22, 80, and 443 from specific IP ranges.

7. **Elastic IP**: Allocate an Elastic IP and associate it with your EC2 instance.

8. **S3 Static Website**: Configure an S3 bucket to host a static website with a simple HTML page.

9. **CloudWatch Monitoring**: Set up a CloudWatch alarm to monitor EC2 CPU utilization above 80%.

10. **RDS Database**: Create a MySQL RDS instance in a private subnet with proper security group configuration.

11. **Route 53 Basics**: Register a domain (or use existing) and create a hosted zone with basic DNS records.

12. **Lambda Function**: Create a simple Lambda function that logs "Hello World" and test it.

13. **SNS Topic**: Create an SNS topic and subscription to send email notifications.

14. **CloudFormation Template**: Write a basic CloudFormation template to create a VPC with public and private subnets.

15. **Cost Optimization**: Identify and implement cost optimization strategies for your AWS resources.

### Intermediate Level (Tasks 16-35)

16. **VPC Creation**: Create a custom VPC with public and private subnets across two availability zones.

17. **NAT Gateway Setup**: Configure NAT Gateway for private subnet internet access.

18. **Application Load Balancer**: Create an ALB to distribute traffic between multiple EC2 instances.

19. **Auto Scaling Group**: Set up an Auto Scaling Group with launch templates and scaling policies.

20. **RDS Multi-AZ**: Configure RDS with Multi-AZ deployment and automated backups.

21. **CloudFront Distribution**: Set up CloudFront to serve content from an S3 bucket with custom domain.

22. **IAM Roles and Policies**: Create custom IAM roles and policies following the principle of least privilege.

23. **Lambda with API Gateway**: Create a REST API using API Gateway that triggers Lambda functions.

24. **DynamoDB Table**: Create a DynamoDB table with GSI and implement basic CRUD operations.

25. **ECS Cluster**: Set up an ECS cluster and run a containerized application.

26. **ECR Repository**: Create an ECR repository and push a Docker image to it.

27. **Systems Manager**: Use SSM to manage EC2 instances without SSH access.

28. **CloudTrail Setup**: Configure CloudTrail for auditing and compliance.

29. **KMS Key Management**: Create and use KMS keys for encryption at rest.

30. **Secrets Manager**: Store and retrieve database credentials using AWS Secrets Manager.

31. **ElastiCache**: Set up Redis ElastiCache cluster for application caching.

32. **SQS Queue**: Create SQS queues for decoupling application components.

33. **CodeCommit Repository**: Set up CodeCommit repository and push code to it.

34. **CodeBuild Project**: Create a CodeBuild project to compile and test your application.

35. **CodePipeline**: Set up a complete CI/CD pipeline using CodePipeline.

### Advanced Level (Tasks 36-50)

36. **Multi-Region Setup**: Deploy infrastructure across multiple AWS regions with cross-region replication.

37. **Disaster Recovery**: Implement a disaster recovery strategy with RTO and RPO requirements.

38. **Network ACLs**: Configure Network ACLs for additional network security layers.

39. **Transit Gateway**: Set up Transit Gateway to connect multiple VPCs.

40. **AWS Organizations**: Create an AWS Organization with multiple accounts and SCPs.

41. **Cost and Usage Reports**: Set up detailed billing reports and analyze costs using Athena.

42. **Lambda Layers**: Create and use Lambda layers for code reusability.

43. **Step Functions**: Build a serverless workflow using AWS Step Functions.

44. **EKS Cluster**: Deploy and manage a Kubernetes cluster using Amazon EKS.

45. **Service Mesh**: Implement AWS App Mesh for microservices communication.

46. **Data Pipeline**: Create a data processing pipeline using AWS Glue and Athena.

47. **Machine Learning**: Deploy a machine learning model using SageMaker.

48. **Security Compliance**: Implement AWS Config rules for compliance monitoring.

49. **Performance Optimization**: Optimize application performance using AWS X-Ray and Performance Insights.

50. **Hybrid Cloud**: Set up VPN or Direct Connect for hybrid cloud connectivity.

---

## DevOps Tasks

### Basic Level (Tasks 1-8)

1. **Git Workflow**: Set up a Git repository with branching strategy (main, develop, feature branches).

2. **Jenkins Installation**: Install Jenkins on a Linux server and configure basic security.

3. **Simple Pipeline**: Create a Jenkins pipeline that pulls code, runs tests, and builds an application.

4. **Docker Basics**: Create a Dockerfile for a web application and build/run the container locally.

5. **Version Control**: Implement semantic versioning and tag releases in your Git repository.

6. **Code Quality**: Integrate SonarQube with Jenkins for code quality analysis.

7. **Artifact Management**: Set up Nexus or Artifactory for storing build artifacts.

8. **Basic Monitoring**: Configure Prometheus and Grafana for basic application monitoring.

### Intermediate Level (Tasks 9-16)

9. **Multi-Stage Pipeline**: Create a Jenkins pipeline with multiple stages (build, test, security scan, deploy).

10. **Jenkins Agents**: Configure Jenkins master-slave architecture with multiple build agents.

11. **GitLab CI/CD**: Implement CI/CD using GitLab CI with multiple environments.

12. **Container Registry**: Set up a private Docker registry and integrate it with your pipeline.

13. **Environment Management**: Create and manage multiple environments (dev, staging, prod) with different configurations.

14. **Blue-Green Deployment**: Implement blue-green deployment strategy for zero-downtime releases.

15. **Security Scanning**: Integrate security scanning tools (OWASP ZAP, Clair) into your pipeline.

16. **Configuration Management**: Use Ansible to automate server configuration and application deployment.

### Advanced Level (Tasks 17-20)

17. **GitOps Implementation**: Implement GitOps using ArgoCD or Flux for Kubernetes deployments.

18. **Multi-Cloud Pipeline**: Create a pipeline that can deploy to multiple cloud providers.

19. **Compliance Pipeline**: Implement compliance checks and automated reporting in your CI/CD pipeline.

20. **DevSecOps**: Integrate security at every stage of the DevOps pipeline with automated security testing.

---

## Terraform Tasks

### Basic Level (Tasks 1-8)

1. **First Infrastructure**: Create a Terraform configuration to provision a single EC2 instance.

2. **Variable Management**: Refactor your configuration to use input variables and output values.

3. **State Management**: Configure remote state storage using S3 backend with DynamoDB locking.

4. **Resource Dependencies**: Create resources with implicit and explicit dependencies.

5. **Data Sources**: Use data sources to reference existing AWS resources in your configuration.

6. **Local Values**: Implement local values for complex expressions and repeated values.

7. **Count Parameter**: Use count to create multiple similar resources.

8. **Terraform Modules**: Create your first reusable module for a web server setup.

### Intermediate Level (Tasks 9-16)

9. **Complex Networking**: Create a complete VPC with multiple subnets, route tables, and gateways.

10. **For Each Loop**: Use for_each to create resources from a map or set of strings.

11. **Conditional Resources**: Implement conditional resource creation using count and for_each.

12. **Module Registry**: Publish a module to the Terraform Registry and use community modules.

13. **Workspace Management**: Use Terraform workspaces to manage multiple environments.

14. **Dynamic Blocks**: Implement dynamic blocks for creating repeated nested blocks.

15. **Provider Configurations**: Configure multiple provider instances (multi-region deployment).

16. **Remote Module Sources**: Use modules from different sources (Git, HTTP, local paths).

### Advanced Level (Tasks 17-20)

17. **Custom Provider**: Create or contribute to a custom Terraform provider.

18. **Complex State Operations**: Perform advanced state operations (import, move, taint).

19. **Terraform Cloud**: Integrate with Terraform Cloud for collaborative infrastructure management.

20. **Policy as Code**: Implement Terraform Sentinel policies for governance and compliance.

---

## Linux & EC2 Tasks

### Basic Level (Tasks 1-8)

1. **System Information**: Write scripts to gather and display comprehensive system information.

2. **User Management**: Create users, groups, and manage permissions on a Linux system.

3. **File Operations**: Perform advanced file operations using command-line tools (find, grep, sed, awk).

4. **Process Management**: Monitor, manage, and troubleshoot system processes.

5. **Cron Jobs**: Set up automated tasks using cron for system maintenance.

6. **Log Analysis**: Analyze system logs to troubleshoot issues and monitor system health.

7. **Package Management**: Install, update, and manage software packages on different Linux distributions.

8. **Network Configuration**: Configure network interfaces, routing, and firewall rules.

### Intermediate Level (Tasks 9-16)

9. **Web Server Setup**: Install and configure Apache/Nginx with SSL certificates.

10. **Database Installation**: Install and configure MySQL/PostgreSQL with proper security settings.

11. **Backup Solutions**: Implement automated backup solutions for system and application data.

12. **Performance Tuning**: Optimize Linux system performance for specific workloads.

13. **Security Hardening**: Implement security best practices and hardening measures.

14. **System Monitoring**: Set up comprehensive system monitoring using various tools.

15. **Load Balancer Configuration**: Configure HAProxy or Nginx as a load balancer.

16. **Docker on Linux**: Install Docker and manage containerized applications on Linux.

### Advanced Level (Tasks 17-20)

17. **High Availability Setup**: Configure high availability using Pacemaker/Corosync.

18. **Custom Kernel Compilation**: Compile and install a custom Linux kernel.

19. **Advanced Networking**: Set up VLANs, bonding, and advanced network configurations.

20. **Performance Profiling**: Use advanced tools for system performance profiling and optimization.

---

## ECS Tasks

### Basic Level (Tasks 1-8)

1. **ECS Cluster Creation**: Create an ECS cluster using both EC2 and Fargate launch types.

2. **Task Definition**: Create a task definition for a simple web application container.

3. **Service Deployment**: Deploy a service with desired count and health checks.

4. **Container Registry**: Push container images to ECR and reference them in task definitions.

5. **Load Balancer Integration**: Integrate ECS service with Application Load Balancer.

6. **Environment Variables**: Configure environment variables and secrets in task definitions.

7. **Logging Configuration**: Set up CloudWatch logging for container applications.

8. **Auto Scaling**: Configure service auto scaling based on CPU and memory metrics.

### Intermediate Level (Tasks 9-16)

9. **Multi-Container Tasks**: Create task definitions with multiple containers and shared volumes.

10. **Service Discovery**: Implement service discovery using AWS Cloud Map.

11. **Secrets Management**: Use AWS Secrets Manager and Parameter Store with ECS.

12. **Custom Networking**: Configure custom VPC networking for ECS tasks.

13. **Capacity Providers**: Set up and use ECS capacity providers for cost optimization.

14. **Blue-Green Deployment**: Implement blue-green deployments using ECS and CodeDeploy.

15. **Monitoring and Alerting**: Set up comprehensive monitoring using CloudWatch and custom metrics.

16. **CI/CD Integration**: Create a complete CI/CD pipeline for ECS deployments.

### Advanced Level (Tasks 17-20)

17. **Service Mesh**: Implement AWS App Mesh with ECS for microservices architecture.

18. **Spot Instance Integration**: Use Spot instances with ECS for cost optimization.

19. **Cross-Region Deployment**: Deploy ECS services across multiple AWS regions.

20. **Advanced Security**: Implement task IAM roles, security groups, and compliance monitoring.

---

## EKS Tasks

### Basic Level (Tasks 1-8)

1. **EKS Cluster Setup**: Create an EKS cluster using eksctl or AWS CLI.

2. **Node Group Configuration**: Set up managed node groups with different instance types.

3. **kubectl Configuration**: Configure kubectl to interact with your EKS cluster.

4. **Pod Deployment**: Deploy your first pod and service to the EKS cluster.

5. **Ingress Controller**: Install and configure AWS Load Balancer Controller.

6. **Persistent Storage**: Set up EBS CSI driver and create persistent volumes.

7. **Namespace Management**: Create and manage multiple namespaces for different applications.

8. **ConfigMaps and Secrets**: Use ConfigMaps and Secrets for application configuration.

### Intermediate Level (Tasks 9-16)

9. **Helm Package Manager**: Install Helm and deploy applications using Helm charts.

10. **Horizontal Pod Autoscaler**: Configure HPA for automatic pod scaling.

11. **Cluster Autoscaler**: Set up cluster autoscaler for automatic node scaling.

12. **RBAC Configuration**: Implement Role-Based Access Control for security.

13. **Network Policies**: Configure network policies for pod-to-pod communication.

14. **Monitoring Stack**: Deploy Prometheus, Grafana, and AlertManager on EKS.

15. **CI/CD with EKS**: Create a CI/CD pipeline that deploys to EKS using GitOps.

16. **Service Mesh**: Deploy Istio service mesh on EKS cluster.

### Advanced Level (Tasks 17-20)

17. **Multi-Cluster Management**: Set up and manage multiple EKS clusters.

18. **Spot Instances**: Configure EKS to use Spot instances for cost optimization.

19. **Custom CNI**: Implement custom CNI solutions for advanced networking.

20. **Backup and Disaster Recovery**: Implement comprehensive backup and DR strategies for EKS.

---

## How to Use This Task Collection

1. **Choose Your Level**: Start with tasks appropriate to your current skill level
2. **Practice Regularly**: Complete 2-3 tasks per week to maintain momentum
3. **Document Your Work**: Keep notes and screenshots of your implementations
4. **Share and Collaborate**: Work with others and share your solutions
5. **Build Progressive**: Each task builds upon previous knowledge

**Good luck with your DevOps journey!** 🚀
