# eks-cicd-dr Project

eks-cicd-dr is prototyping project for CI/CD and DR (Disaster Recovery) based on EKS. eks-cicd-dr project consists of the following git repositories.

* [aws-config](https://github.com/aws-playground/eks-cicd-dr_aws-configs) : Guide to set AWS resources and EKS clusters
* [service-peccy-web](https://github.com/aws-playground/eks-cicd-dr_service-peccy-web) : Service peccy's web server 
* [service-peccy-app](https://github.com/aws-playground/eks-cicd-dr_service-peccy-app) : Service peccy's app server
* [deploy-eks](https://github.com/aws-playground/eks-cicd-dr_deploy-eks) : K8s manifests for GitOps with ArgoCD

## Service Peccy

<img src="/image/service_peccy_web.png" width="400"/>

Servic peccy is a simple web service to demonstrate eks-cicd-dr project. Service peccy consists of **3-tier** architecture. Service peccy uses AWS RDS Aurora (MySQL) to store peccy's hobby and AWS EFS to store peccy's picture.

## Architecture

### Peccy service (3-tier Architecture) on AWS EKS cluster

<img src="/image/EKS_Cluster.png" width="800"/>

### Peccy service CI with AWS CodePipeline and AWS CodeBuild

<img src="/image/CI.png" width="800"/>

### Peccy service CD with ArgoCD on AWS EKS cluster

<img src="/image/CD.png" width="800"/>

### Disaster recovery on one region

<img src="/image/DR_One_Region.png" width="800"/>

### Disaster recovery on multiple regions

<img src="/image/DR_Multi_Region.png" width="800"/>

