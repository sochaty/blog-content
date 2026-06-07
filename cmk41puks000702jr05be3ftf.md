---
title: "From Code to Cloud: Mastering EKS Deployment with Terraform and GitHub Actions"
seoTitle: "EKS Deployment: Terraform & GitHub Actions Guide"
seoDescription: "Learn to deploy an EKS cluster on AWS using Terraform, GitHub Actions, and AWS services for a seamless CI/CD pipeline automation"
datePublished: 2026-01-07T13:20:07.180Z
cuid: cmk41puks000702jr05be3ftf
slug: from-code-to-cloud-mastering-eks-deployment-with-terraform-and-github-actions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1767791871471/2b162f3f-2de7-4d7b-aa96-48fc05ccdab5.png
tags: aws, terraform, infrastructure-as-code, eks, github-actions-1

---

## Introduction

In the rapidly changing world of cloud computing, deploying and managing applications with reliability and scalability is crucial. This blog post delves into a real-world project that showcases the utilization of various AWS services and tools to achieve seamless deployment of an Elastic Kubernetes Service (EKS) cluster and application deployment on Amazon's Elastic Kubernetes Service (EKS) using Terraform and automated using GitHub Actions. Building on insights from the blog [**Exploring AWS EKS with eksctl**](https://blogs.sourishchakraborty.com/exploring-aws-eks-with-eksctl), we explore a comprehensive implementation leveraging AWS Parameter Store, AWS Secrets Manager, AWS EKS, and AWS Network Services, combined with GitHub Actions for CI/CD.

## Technologies Used

### AWS Systems Manager/ Parameter Store:

The AWS Parameter Store is a managed service that allows you to store configuration data securely. In this project, the AWS Parameter Store is employed to store inputs for the EKS cluster configuration. This ensures that sensitive information is kept separate from the codebase and can be dynamically retrieved during deployment.

### AWS Secrets Manager:

Securing sensitive information like SSH keys is paramount. AWS Secrets Manager offers a secure and scalable solution for storing, distributing, and rotating secrets. The project utilizes this service to securely manage SSH secret key files required for accessing the Bastion Host.

### AWS EKS:

Amazon EKS simplifies the process of deploying, managing, and scaling containerized applications using Kubernetes. The project sets up an EKS cluster with public and private node groups, each running a desired number of EC2 instances. This architecture provides a balance of security and accessibility.

### AWS Network Services:

The custom VPC and its components, including subnets and security groups, play a pivotal role in creating a secure and isolated networking environment for the EKS cluster and other resources. Properly configuring these components is essential for maintaining a robust infrastructure.

## Building Infrastructure Stack

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766930867445/f29583e8-bb82-40d8-b37d-0c61d47762c3.png align="left")

We are going to build an AWS EKS cluster within a custom multi-tier network. Deployed cloud components will contain an EKS control plane, an EKS data plane with public and private node-groups, IAM roles for the cluster as well as node-groups, and a bastion host EC2 instance in the public subnet to connect the worker EC2 node instances.

A custom VPC cloud with all required components will also be deployed in the due process. We will deploy a VPC with two subnets: one public and one private. A NAT Gateway will be deployed in the public subnet, with routes registered in the route table to direct incoming traffic to worker nodes in the private node group.

All these cloud components will be deployed by using a Terraform codebase with a workflow defined in GitHub Actions. After a successful deployment of the infrastructure stack, we can access various EKS components using the AWS console and the kubectl CLI utility.

### Code Structure

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766931024313/359a0a49-0054-41aa-a556-00a2d4fea765.png align="center")

The above figure shows that the codebase highlighted by red boxes is for infrastructure deployment. The **infra-deployment** code section has three different main execution sections. **deploy-bastion-host** is for deploying EC2 instances in the public subnet. This EC2 instance can connect to EKS worker nodes via port 22(SSH protocol). **deploy-network** builds the custom VPC network. **deploy-eks** deploys the AWS EKS cluster on a custom VPC network. Bastion host code uses a **compute** module. Network and EKS cluster build uses **network** and **k8s** modules for deploying cloud components. Just above the **infra-deployments** codebase section, there is **iam** codebase, which uses **iam-role** module to deploy cluster and node group-specific IAM roles. The **environments** folder contains all Terraform input variable values for the dev environment.

> Full code can be accessed [here](https://github.com/sochaty/eks-with-tf/tree/main/terraform)

## Building Application/Platform Stack

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766931333222/09259562-0a8e-4335-8076-aa45ef86c6b5.png align="center")

We are going to deploy a simple application stack by using the Kubernetes Terraform provider. The application stack will use an Nginx image from Docker to deploy an application pod and a Kubernetes Load Balancer Service, specifically an AWS Classic Load Balancer.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766931370671/1b0eaf43-6253-4b4f-8c03-25d80037fcbc.png align="center")

The above figure shows the simple view of the application deployed in the EKS Cluster using the Terraform Kubernetes Provider.

### Code Structure

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766932438446/65b4e437-9e8a-47db-8190-aa901bf54f26.png align="center")

The above figure shows the codebase highlighted by red boxes is for the application/platform. The **platform-deployment** code section has one main execution section called deploy-app. deploy-app uses the platform module to deploy a simple Nginx web server application using the Kubernetes Terraform Provider.

> Full code can be accessed [here](https://github.com/sochaty/eks-with-tf/tree/main/terraform)

## EKS Elastic Network Interface Significance

### What are ENIs?

Elastic Network Interfaces (ENIs) are logical networking devices that can be attached to Amazon EC2 instances or AWS Fargate tasks. ENIs provide a way for instances and tasks to communicate with each other and with the outside world.

### How do ENIs work with Amazon EKS?

When we created the above Amazon EKS cluster, Amazon EKS created a set of ENIs in our account. These ENIs are attached to the EKS control plane, which is a group of Amazon EC2 instances that manage our cluster. The ENIs allow the control plane to communicate with our pods, which are the containers that run our applications (here, our **demoapp** and other system-level pods).

### What are the benefits of using ENIs with Amazon EKS?

There are several benefits to using ENIs with Amazon EKS:

* Performance: ENIs provide a dedicated network interface for each instance or task, which can improve performance.
    
* Security: ENIs can be associated with security groups, which can help to control access to our pods.
    
* Scalability: ENIs can be scaled up or down as needed, which makes them a good choice for clusters that are expected to change in size.
    

The figure in the above section shows the ENIs that were created for our Amazon EKS cluster. These ENIs were created in our EKS VPC, but they are attached to the EKS control plane master nodes. The control plane master nodes are responsible for managing our cluster, so it is important that they have a dedicated network interface.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1766932899379/d2ac1339-df5f-4aaa-a12a-55f8fb84a4cb.png align="center")

The above figure shows the Network Interfaces created by the EKS Control Plane in our EKS VPC. These are created and managed by EKS. Network Interface description is outlined as Amazon EKS &lt;CLUSTER-NAME&gt;.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767023141745/c15a7af8-8053-4da0-9c84-797b2a461eee.png align="center")

The above figure shows the VPC ID, which is of our deployed custom VPC. The Owner is our Account ID, and the IP address is actually a private IP Address from our EKS VPC public subnet. The instance owner is the Amazon VPC Account ID where the EKS Control plane is hosted.

## GitHub Action Workflows

GitHub Actions offer a powerful way to automate workflows directly within the GitHub repository. In this project, three distinct pipelines have been set up.

### EKS-With-Terraform CI

This workflow/pipeline can be triggered upon code changes (currently, it’s a manual run). It runs tests, linters, and checks to ensure code quality. It acts as an initial gatekeeper before any further deployment actions. There are five jobs defined in this workflow. These jobs are;

* iam\_ci
    
* network\_ci
    
* eks\_ci
    
* bastion\_host\_ci
    
* app\_ci
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767023376813/8f5a4153-1468-4e3f-b137-936ddf1219f7.png align="center")

The above figure shows the GitHub Actions CI workflow, which executes some scans for format checking, configuration validation, static code scans for compliance, and vulnerability detection.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767023438374/359ac289-5570-4ce3-8b16-ce8bc83367c5.png align="center")

The above figure shows a snapshot of the CI workflow from the GitHub Actions summary page. The workflow is called **EKS-With-Terraform CI**.

> Full code for workflows can be accessed [here](https://github.com/sochaty/eks-with-tf/tree/main/.github/workflows)

### EKS-With-Terraform Infrastructure Deployment

Once the code passes the CI workflow/pipeline, this workflow is triggered manually or automatically. It leverages Terraform to deploy the EKS cluster, bastion host, and application stack. AWS Parameter Store and AWS Secrets Manager are used to fetch the necessary configuration securely.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767023503161/09a06f64-3917-4e6b-b99d-33036bf569ff.png align="center")

The above figure shows a snapshot of full-stack infrastructure and application/platform deployment workflow from the GitHub Actions summary page. The workflow is called **EKS-With-Terraform Infrastructure Deployment**.

There are five different jobs defined in this workflow.

**Deploy Network:** This job deploys a custom VPC with all its required cloud components

**Deploy IAM:** This job deploys two IAM roles. One role is related to Cluster Operations, and the other role is for Node Groups.

**Deploy EKS:** This job is related to the above two jobs. Once network and IAM roles are deployed, then this job starts it’s execution. It deploys an EKS cluster using the already deployed Network.

**Deploy App:** This job deploys the Nginx web server application in the already deployed EKS cluster. The application is deployed in the default namespace. Classic Load Balancer service is also provisioned for accessing the application.

All the initial inputs for Infrastructure deployment were fetched from the AWS System Manager/ Parameter Store.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767023560414/82119c70-6517-4c98-ac67-7708e7260f2c.png align="center")

The above figure shows a snapshot of inputs stored in the Parameter Store.

> Full code for wokflows can be accessed [here](https://github.com/sochaty/eks-with-tf/tree/main/.github/workflows)

### EKS-With-Terraform Deployment Delete

This workflow/pipeline allows for the graceful dismantling of the infrastructure when needed. It cleans up all the resources created during the stack deployment process.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767023616350/6a3f706a-a602-4e4f-94ee-9c73a75e28e5.png align="center")

The above figure shows a snapshot of deleting the deployment workflow from the GitHub Actions summary page. The workflow is called **EKS-With-Terraform Deployment Delete.**

> Full code for wokflows can be accessed [here](https://github.com/sochaty/eks-with-tf/tree/main/.github/workflows)

### EKS Cluster Verification

The deployed cluster can be verified with a visual inspection of deployed components in the AWS Management Console.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767023683215/1975a9e5-4520-4906-a02d-f94a7ebbdd7f.png align="center")

The above figure shows the deployed cluster in the Amazon Elastic Kubernetes Service Clusters listing.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767023731423/17c63234-f9bd-427d-8d16-500190d5a3eb.png align="center")

The above figure shows the cluster overview information.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767023946312/ac450d2d-7cdb-4fb9-8f0c-bcebec9149eb.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767023977150/a4f1605b-b1a3-4251-bd5f-d8b160b884ea.png align="center")

The above figures show the Kubernetes workloads, like pods and deployments, in the resources section of the deployed cluster.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024026887/25c3e648-3e12-49fb-9d3c-5894b068d840.png align="center")

The above figure shows deployed k8s services. The Load Balancer service called demoapp-lb-service is highlighted with the red box. We can use it’s generated URL to browse the Nginx home page in our webpage browser.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024098211/b75827a2-3e29-474c-bb58-4c8131de0ad4.png align="center")

The above figure shows the Nginx home page accessed by the URL of the load balancer service.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024136518/52491eae-7e02-4885-aa96-2013fa16f95c.png align="center")

The above figure shows the deployed EC2 instances, one from the public node group, one from the private node group, and named EC2 called **finance-dev-BastionHost** is the Bastion Host deployed in the public subnet.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024229692/609e722c-a9d9-48ae-b95b-1f57fe5cfc5a.png align="center")

The above figure shows the Security Groups created for the EKS Cluster and EKS Worker Node Groups (Public and Private).

### Verify Using Bastion Host

Deployed worker nodes can be verified by connecting with the Bastion Host to these worker EC2 nodes.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024309161/dfdba963-19a2-4cf6-a1f8-3d3c30fce848.png align="center")

The above figure shows how we can use the Public IP address of the Bastion Host to connect it. We have created a SSH key pair and stored that as a secret in AWS Secrets Manager. During the deployment process, we connected to Secrets Manager via Terraform and transferred it to the Bastion Host after provisioning. In this way, we have not kept any sensitive information in the code repository.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024429339/72dac05d-d5ab-474b-9f87-99e129a4253a.png align="center")

The above figure shows the SSH file (demokey1.pem) in the connected Bastion Host.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024514005/881d16f4-bd2f-471d-9a35-eff0e2f5808b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024544991/b8aaabca-b629-4cb8-adad-dd795056a5c2.png align="center")

The above figures show connected nodes from public and private node groups.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024594765/79e57391-9f4f-49df-85fb-9ed1f19361c9.png align="center")

The above figure shows the traffic routing of the connected worker node to the EKS API Server Endpoint. We have used wget &lt;eks-api-endpoint-url&gt;, and the result shows the IP addresses highlighted in the red box, which are routed to public IP addresses.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024641795/94d9c70a-9f82-4463-956e-f9d9ed60748b.png align="center")

The above figure shows the kubelet verification with ECR. With the command of grep kube, we can see in the result that the image is being pulled out from ECR (Elastic Container Registry). All the system-level pods were deployed from images in ECR. These ECRs were owned by Amazon.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024673192/867f1bd1-2b15-4a00-a4e3-599f6a6b9027.png align="center")

The above figure shows the command output of **cat /etc/Kubernetes/kubelet/kubelet-config.json**.Output is from **KubeletConfiguration**, which confirms that worker nodes were installed with Kubelet.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024705015/4306387f-4790-4343-889f-5031c0c87e7f.png align="center")

The above figure shows the output of the command **cat /var/lib/kubelet/kubeconfig**.

#### Verify K8s Deployed Components using kubectl

Deployed k8s components can be verified by executing kubectl commands.

`aws eks update-kubeconfig --region region-code --name my-cluster`

Use the above command to get the context of the deployed EKS cluster in the command prompt.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024837623/d756b44c-c099-4901-8b23-9a997db867f2.png align="center")

The above figure shows the command output of the command **kubectl get n**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024902293/ed77b384-728d-4834-b7f8-3d7d1948d45c.png align="center")

The above figure shows the command output of the command **kubectl get all -n default**. Here we can see that our deployed application (pod) and load balancer service are listed. We can copy the External-IP of the service to use this in the web browser to access the Nginx home page.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1767024944172/802bf4d0-dccd-4887-88c8-db5b32e1a39f.png align="center")

The above figure shows the command output of the command **kubectl get all -n kube-system**

## Summary

Just to summarize the post 😊

This blog exemplifies the convergence of modern cloud practices with the power of automation and orchestration tools. By harnessing AWS Parameter Store, AWS Secrets Manager, AWS EKS, and various networking components, developers can create a highly available and configurable environment for their applications. With GitHub Actions, the entire process is streamlined, offering efficient CI/CD pipelines for testing, deployment, and maintenance. By following the footsteps of the blog [**Exploring AWS EKS with eksctl**](https://blogs.sourishchakraborty.com/exploring-aws-eks-with-eksctl), this blog provides a hands-on approach to mastering the deployment of Kubernetes-based applications on AWS using Terraform.

That’s it for the day. Do let me know your feedback in the comment section.

I will be back with some other topic, till then Bye!