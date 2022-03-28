# Lab2: EKS for CNF engineer (Application creation and private ECR repo)

## AWS Region
* ONLY use 'us-west-2' (Oregon) AWS region during these labs.

## 1. Create a Docker Image 
* Prepare docker environment (login your bastion host EC2 instance)
  ````
  sudo yum install docker -y
  sudo usermod -aG docker ${USER}
  sudo service docker start
  ````
  Logout from Bastion and Login back.

  > **_NOTE:_** You have to re-configure AWS Credential (copy "export AWS_..." again). 
  
  Verify docker works
  ````
  docker
  ````
* Create a directory for Dockerfile creation.
  ````
  mkdir dockerimage && cd dockerimage
  ````
* Create a Dockerfile file (file name to be Dockerfile with below contents)
  ````
  cat <<EoF > Dockerfile
  FROM ubuntu:focal
  ENV DEBIAN_FRONTEND noninteractive
  RUN apt-get update && apt-get install -y net-tools iputils-ping iproute2 python python3 pip vim
  RUN pip install boto3 requests
  WORKDIR /
  EoF
  ````
* Build a Docker image.
  ```` 
  docker build .
  ````
* Verify Docker image created. And note down image ID
  ````
  docker images 
  ````
Example output:

````
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
<none>       <none>    1da914777598   9 seconds ago   526MB <-- This your image
ubuntu       focal     e784f03641c9   5 days ago      65.6MB
````

> **_NOTE:_**  Write down IMAGE ID for image you created (above example 1da914777598) - you need it below

## 2. Upload Image to the ECR Repository
Run below commands at Bastion host (where you created a docker image) 

* Create an Elastic Container Repository (ECR) named my-ecr in us-west-2
  ````
  aws ecr create-repository --repository-name my-ecr --region us-west-2
  ````
Note down "repositoryUri"

> **_NOTE:_** Replace \<URI from ECR\> with URI of your ECR repo, and IMAGE Id from above "docker images" command.

*Use your URI / Account ID and Image ID of your docker (you can retrieve all information from above docker images command and from AWS ECR console)*

Login to ECR:
  ````
  aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin <URI from ECR>
  ````
  Tag Image:
  ````
  docker tag <IMAGE ID you created> <URI from ECR>:latest
  ````
  Push Image:
  ````
  docker push <URI from ECR>:latest
  ````
  
## 3. Create Multus App using uploaded Docker image from the ECR
* Create a new directory at your bastion host (under /home/ec2-user/ or other location you like)
````
mkdir k8s-environment && cd k8s-environment
````
* Install multus CNI (This uses us-west-2 as default image location - below is for arm64).
 ````
 kubectl apply -f https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/master/config/multus/v3.7.2-eksbuild.2/aws-k8s-multus.yaml
 ````
Validate that demonset started the kube-multus-xxxxx PoD(s):
````
kubectl get pod -A
````

* Create below networkAttachmentDefinition (multus-ipvlan.yaml) and apply it to the cluster.

````yaml
cat <<EoF > multus-ipvlan.yaml
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: ipvlan-conf
spec:
  config: '{
      "cniVersion": "0.3.0",
      "type": "ipvlan",
      "master": "eth1",
      "mode": "l2",
      "ipam": {
        "type": "host-local",
        "subnet": "10.0.4.0/24",
        "rangeStart": "10.0.4.70",
        "rangeEnd": "10.0.4.80",
        "gateway": "10.0.4.1"
      }
    }'
EoF
````

  ````
  kubectl apply -f multus-ipvlan.yaml
  ````

* Deploy your docker using above network attachment. (create a file named, app-ipvlan.yaml)
````yaml
 cat <<EoF > app-ipvlan.yaml
 apiVersion: v1
 kind: Pod
 metadata:
   name: samplepod
   annotations:
     k8s.v1.cni.cncf.io/networks: ipvlan-conf
 spec:
   containers:
   - name: samplepod
     command: ["/bin/bash", "-c", "trap : TERM INT; sleep infinity & wait"]
     image: <URI from ECR>:latest
EoF
````
> **_NOTE:_** Update image: with value from your image in ECR (URI)

  ````
  kubectl apply -f app-ipvlan.yaml
  ````
  ````
  kubectl describe pod samplepod
  ````
* Verify your Pod has 2 interfaces (eth0 for default K8s networking and net1 for Multus interface (10.0.4.0/24)
````
kubectl exec -it samplepod -- /bin/bash
````  
````
root@samplepod:/# ifconfig
````
### Automated Multus pod IP management on EKS / VPC

Multus pods are using ipvlan CNI, which means that the mac-address of the pod remains same as the master interface. In this case AWS VPC will not be aware of the assumed IP address of the pod, since the IP allocations to these pods hasn’t happened via VPC. VPC is only aware of the IP addresses allocated on the ENI on EC2 worker nodes. To make these IPs routable in VPC network, please refer to Automated Multus pod IP management on EKS: https://github.com/aws-samples/eks-automated-ipmgmt-multus-pods to automate the pod IP assignment seamlessly, without any change in application code.

## 4. Clean up environment
1. Go to CloudFormation and "Delete" "eks-workers" stack you created. 
2. After above is deleted also "Delete" the "AWS-Infra" stack.
3. Delete and empty also S3 bucket your created. 

## What next? 
* Look around in the environment - EKS, EC2, Lambda, EventBridge - how things relate ?
* Validate alternate connectivity methods - example tunnel SSH session through Session Manager [SSH-SSM](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-getting-started-enable-ssh-connections.html)
* Update some parameter in CFN - what happens ?
* Official AWS Multus Guide: https://github.com/aws-samples/eks-install-guide-for-multus (Includes latest updates)
* Look additional EKS labs in [eksworksop.com](https://www.eksworkshop.com/)
  * Especially: IAM/RBAC/IRSA related and how to work those with EKS
* You can also walk through below blog post contents. You already have done the most of parts of setup process guided in the blog (Note that we used arm64 here) - in blog will build your own functional Open Source 4G EPC Core on EKS environment within 45 min following steps similar to this course. https://aws.amazon.com/blogs/opensource/open-source-mobile-core-network-implementation-on-amazon-elastic-kubernetes-service/

* Go Build your CNF apps with AWS!