# eks-cicd-dr Project

eks-cicd-dr is prototyping project for CI/CD and DR (Disaster Recovery) based on EKS. eks-cicd-dr project consists of the following git repositories.

* [aws-config](https://github.com/aws-playground/eks-cicd-dr_aws-configs) : Guide to set AWS resources and EKS clusters
* [service-peccy-web](https://github.com/aws-playground/common_service-peccy-web) : Service peccy's web server 
* [service-peccy-app](https://github.com/aws-playground/common_service-peccy-app) : Service peccy's app server
* [deploy-eks](https://github.com/aws-playground/eks-cicd-dr_deploy-eks) : K8s manifests for GitOps with ArgoCD

## Service Peccy

<img src="/images/service-peccy-web.png" width="400"/>

Servic peccy is a simple web service to demonstrate eks-cicd-dr project. Service peccy consists of **3-tier** architecture. Service peccy uses AWS RDS Aurora (MySQL) to store peccy's hobby and AWS EFS to store peccy's picture.

## Architecture

### Peccy service (3-tier Architecture) on AWS EKS cluster

<img src="/images/architecture-eks-cluster.png" width="800"/>

* EKS Node Group
  * Manage : Node group for controller and plugins. No need to high spec EC2 instance because controller and plugins do not process larget traffic. And don't need a large number of EC2 Instances. Two EC2 instances are recommended.
  * Web Server : Node group for service peccy's web server containers.
  * App Server : Node group for service peccy's app server containers.
* Controller and Plugins
  * ALB Controller : Control AWS ALB through k8s ingress object.
  * ExternalDNS Controller : Control AWS Route 53 through k8s ingress object.
  * EFS Controller, Node : Control AWS EFS through k8s storage class and persistance volume.
  * Fluentd Bit : Collect and send container's logs to AWS CloudWatch Logs.
  * CloudWatch Agent : Collect and send container's metrics, EC2 instance's metrics to AWS CloudWatch Metrics.

### Peccy service CI with AWS CodePipeline and AWS CodeBuild

<img src="/images/architecture-ci.png" width="800"/>

* Run unit test and build container image through AWS CodePipeline and AWS CodeBuild.
* Push container image to two AWS ECR located in different regions for DR.
* AWS CodePipeline process is notifed to service developers and devops engineers through AWS Chatbot.

### Peccy service CD with ArgoCD on AWS EKS cluster

<img src="/images/architecture-cd.png" width="800"/>

* ArgoCD polls k8s manifest git repo and apply new version of k8s manifest. 

### Disaster recovery on one region (AZ Down)

<img src="/images/architecture-dr-one-region.png" width="800"/>

* Target RPO, RTO
  * RPO = 0 min : Multi-AZ Sync Replication
  * RTO < 10 min : Duplication and Automatically Fail Over
* Prepare DR
  * Containers
    * Deploy containers through K8s deployment.
    * Automatically fail over by K8s.
  * EFS
    * Storage : Using Multi-AZ storage to prevent data loss.
    * ENI : Duplicate ENI for HA.
  * RDS Aurora
    * Storage : Using Multi-AZ storage to prevent data loss.
    * EC2 Instance : Automatically promoted a reader instance to writer instance by RDS Aurora.

### Disaster recovery on multiple regions (Region Down)

<img src="/images/architecture-dr-multi-region.png" width="800"/>

* Target RPO, RTO
  * RPO < 10 min : Cross-region Async Replication
  * RTO < 30 min : Manually Change Primary and Provisioning Computing Resource
* Prepare DR
  * Secondary EKS Cluster Node Group
    * Set number of Web Server and App Server EKS node group to 0 to reduce cost of EC2 instances.
    * Set two Manage EKS node group instances. And install all controllers, plugins, ArgoCD and BotKube.
  * Secondary EKS Cluster
    * Apply the same k8s manifest for service peccy as the primary EKS cluster.
  * Route 53
    * Set Failover Routing rule to route traffic to secondary EKS cluster's ALB automatically.
* Perform DR
  * First : EFS
    * Remove EFS replication.
    * After removing EFS replication, Read-only EFS promoted to Writable EFS.
  * Second : RDS Aurora
    * Run global database failover to change primary RDS Aurora.
  * Third : Node Group
    * Increase number of Web Server and App Server EKS node group
    * After increasing EKS node gorups, Web/App server container run automatically by K8s.

