# Deploy Spring Boot application with MySql database in Amazon EKS (Amazon Elastic Kubernetes Service)

# Click here to watch the video for demonstration.

In this project, I have demonstrated:

1.Create and setup custom VPC with private and public subnet having NAT gateway and Internet gateway.
2.Deployed and setup MySql database in private subnet using Amazon RDS service.
3.Create EKS cluster in private subnet and deploy springboot application.
4.Deployed Internet facing AWS Application load balance to access to application.
5.Connect to MySql database which deployed in private subnet from local machine.
Here I have created REST API's to add and retrieve ExchangeRate in the MySql database.

# Prerequisites:

.Docker or Docker alternative - Colima
.AWS Account with configured access from local machine and AWS CLI installation
.Helm
.kubectl
.eksctl or From Official eksctl website
.IDE and JDK
.MySql Client: DBeaver

# Perform following steps for application deployment:

1.Setup Custom VPC in AWS account using VPC service including NAT Gateway and Internet Gateway.

2.Create a Security Group having inbound rule with access from anywhere. Select VPC created in Step-1 while creating Security group (Note. You can create multiple security groups with restricted access.).

3.Create a MySql database subnet group having a private subnet. (using Amazon RDS service).

4.Create a MySql database and use the subnet group created in Step-3. Also use existing security group created in Step-2

5.Create an EC2 instance key pair and download it on a local machine.

6.Launch EC2 instance.

    Use EC2 instance key pair created in step-5 while launching EC2 instance.
    Use the existing security group which was created in Step-2.

7.Once the EC2 instance and MySql databases are in Running/Active state. Connect MySql database from local machine.

    .Open terminal on local machine and cd into ec2 key pair directory.
    .Change permission of ec2 key pair file using command : chmod 0400 ec2-db-key-pair.pem (Note: here ec2-db-key-pair.pem is the file  which is downloaded when we created ec2-key. Change name accordingly.)
    .Then run the following command:
        ssh -i "YOUR_EC2_KEY" -L LOCAL_PORT:RDS_ENDPOINT:REMOTE_PORT EC2_USER@EC2_HOST -N -f
    .Replace YOUR_EC2_KEY with your actual key pair name and other values accordingly.

8.Then open the database client and connect to the database.

9.Clone this repository.

10.From the terminal cd into your project directory and build project using command:

    MYSQL_HOSTNAME=127.0.0.1 MYSQL_PORT=3306 MYSQL_DATABASE=ytlecture MYSQL_USERNAME=root MYSQL_PASSWORD=your_password ./gradlew clean build

    or to build without running test run this command ./gradlew clean assemble

11.Create a docker repository in AWS ECS service. Give it name as springboot-mysql-eks

12.Follow the push commands from AWS ECS repository push command options. (make sure docker is started on the local machine.).

# AWS Application Load Balancer setup steps:

1.Tags all the subnet with the following tags:
i.For Private subnet:
key - kubernetes.io/role/internal-elb
value - 1
ii.For Public subnet:
kubernetes.io/role/elb
value - 1
iii.For more and updated information Click here.
2.Create IAM Policy for AWS Load Balancer Controller. For more information Click here.
i.Open a new terminal. CD into application repository cluster directory.
ii.Get policy document using command :
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
iii.Or you can use policy from repo using cluster/iam_policy.json (Optional)
iv.Apply the policy using command :
aws iam create-policy \
 --policy-name AWSLoadBalancerControllerIAMPolicy \
 --policy-document file://iam_policy.json
3.Create cluster using command: eksctl create cluster -f cluster.yaml
4.Once the cluster is created, then run following commands:
i.kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master"
ii.helm repo add eks https://aws.github.io/eks-charts
iii.helm repo update eks
iv.helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
 --namespace kube-system \
 --set clusterName=spring-test-cluster \
 --set serviceAccount.create=false \
 --set serviceAccount.name=aws-load-balancer-controller
v. kubectl -n kube-system rollout status deployment aws-load-balancer-controller
vi. kubectl get deployment -n kube-system aws-load-balancer-controller
5.Once the application load balancer is ready. Open new terminal and cd into project directory.
6.Run the command to deploy application :
helm install mychart ytchart
7.Check the deployments using kubectl get all
8.Go into EC2 service in AWS account. Select AWS Load balancer and copy DNS name to test application.

# Sample JSON Request for the API :

{
"sourceCurrency" : "USD",
"targetCurrency" : "AUD",
"amount": 1.50,
"lastUpdated" : "2023-11-29"
}

# Following is list of api created in this project:

# To add ExchangeRate in database :

    curl -X POST http://localhost:8080/addExchangeRate \
    -d '{ "sourceCurrency" : "USD", "targetCurrency" : "AUD", "amount": 1.52, "lastUpdated" : "2023-11-22" }' \
    -H 'Content-Type: application/json'

# To get exchange rate based on SourceCurrency and TargetCurrency :

     curl -X GET 'http://localhost:8080/getAmount?sourceCurrency=USD&targetCurrency=AUD'

# Note : In the actual production environment, do not commit files with credentials like we have mentioned in secrets.yaml file.

1.Make sure to delete all the component in AWS after your practice.
2.Run command to delete the cluster eksctl delete cluster -f cluster.yaml
3.If you get any error, manually stop the EC2 instances and delete the Auto-Scaling group.
4.Go to Cloudformation and delete all the stacks.
5.Delete MySql database manually.
6.Delete NAT gateway from VPC.
7.Delete VPC.
