## Setup Dataverse Infrastructure on Amazon Elastic Container Service for Kubernetes (EKS)

In this topic, we create Dataverse infrastructure on Amazon EKS cluster.
We registered dataverse.tk domain name. Register at [freenom](https://freenom.com) and search for your preferred domain then buy it for free.<br/>
With [freenom](https://freenom.com) we can plenty of free domains such as .tk , .cf , .ml , .ga, .gq without the need of any credit card or financial information.
   
### Prerequisites
Before setting up the Dataverse kubernetes cluster, we’ll need an [AWS account](https://aws.amazon.com/account/).
### Installing AWS CLI
First we need an installation of the [AWS Command Line Interface](https://aws.amazon.com/cli/).<br/>
AWS CLI requires Python 2.6.5 or higher. Installation using pip.
```commandline
$pip install awscli
```

Make sure to configure the AWS CLI to use your access key ID and secret access key:
````
$aws configure
AWS Access Key ID [****************E4PT]:
AWS Secret Access Key [****************iTjf]:
Default region name [eu-central-1]:
Default output format [None]:
````
The instruction to get access key ID an secret access key can be found on [AWS - Managing Access Keys for IAM Users - Managing Access Keys (Console)
](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html?icmpid=docs_iam_console#Using_CreateAccessKey).

### Installing Kubernetes Operations(KOPS) and Kubectl
On Mac OS X, we’ll use brew to install. 
```commandline
$brew update && brew install kops kubectl
```
Check the kops an kubectl version:
```commandline
$kops version

Version 1.13.0
$kubectl version
 Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.3", GitCommit:"2d3c76f9091b6bec110a5e63777c332469e0cba2", GitTreeState:"clean", BuildDate:"2019-08-19T12:36:28Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"darwin/amd64"}
 Server Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.10", GitCommit:"37d169313237cb4ceb2cc4bef300f2ae3053c1a2", GitTreeState:"clean", BuildDate:"2019-08-19T10:44:49Z", GoVersion:"go1.11.13", Compiler:"gc", Platform:"linux/amd64"}
```
We're also going to make use of [Helm](https://helm.sh/) - a package manager for Kubernetes, that will be explained as [it come up](#install-helm).
### Setting up a Kubernetes cluster using KOPS on AWS
Choose a cluster name, e.q. dans.dataverse.tk and save as kops environment variable:
```commandline
export KOPS_CLUSTER_NAME=dans.dataverse.tk
```
Create a S3 bucket to store the cluster state using AWS CLI:
```commandline
aws s3api create-bucket --bucket dans-dataverse-state-store --region eu-central-1 --create-bucket-configuration LocationConstraint=eu-central-1
```
Now when we check our bucket, we would see:
![s3](readme-imgs/s3.png "S3")

Once you’ve created the bucket, execute the following command:
```commandline
export KOPS_STATE_STORE=s3://dans-dataverse-state-store
```
For safe keeping we should add the two environment variables: **KOPS_CLUSTER_NAME** and **KOPS_STATE_STORE** to the ~/.bash_profile or ~/.bashrc configs or whatever the equivalent e.q _.zshrc_ .
Since we are on AWS we can use a S3 backing store. It is recommended to enabling versioning on the S3 bucket.<br/>
Let’s enable versioning to revert or recover a previous state store. 
```commandline
aws s3api put-bucket-versioning --bucket dans-dataverse-state-store  --versioning-configuration Status=Enabled
```
### Creating the cluster
AWS is now as ready as it can be, to generate the cluster configuration using the following command:
```commandline
kops create cluster --node-count 1 --master-size t2.small --master-volume-size 10 --node-size t2.small --node-volume-size 10 --zones eu-central-1a
```
![kops-configure](readme-imgs/kops-cluster-conf.png "kops configure")

Initiate the cluster with an update command:
```commandline
kops update cluster --name ${KOPS_CLUSTER_NAME} --yes
```
This will create the resources needed for your cluster to run. It will create a master and two node instances.

Wait for the cluster to start-up (sometimes it needs about a half hour), validate the cluster to ensure the master + 2 nodes have launched:
```commandline
kops validate cluster
```
![kops-validate](readme-imgs/kops-validate.png "kops validate")

If we validate too early, we’ll get an error. Wait a little longer for the nodes to launch, and the validate step will return without error.

Confirm that kubectl is connected to your Kubernetes cluster.
```commandline
kubeclt get nodes
```
![kub-nodes](readme-imgs/kub-nodes.png "kubeclt get nodes")

```commandline
kubeclt cluster-info
```
![cluster-info](readme-imgs/cluster-info.png "kops cluster info")


Now when we check our instances on AWS, we would see three new instances that would have got created. 
![aws-instances](readme-imgs/aws-instances.png "aws instances")

Our s3 bucket will now have some folder in it, which is basically our cluster configuration file.
![s3-folders](readme-imgs/s3-folders.png "s3-folders")

### Deploying Dataverse, ddi-converter-tool and filepreviewers to cluster
Clone this project:

        git clone https://github.com/ekoi/dataverse-kubernetes.git
        git checkout -b ddi-converter-tool 

Deploy dataverse:
        
        kubectl apply -k docs/aws-demo
        
Deploy fileviewers:
    
        kubectl apply -k k8s/filepreviewers
        
Deploy ddi-converter-tool:
    
        kubectl apply -k k8s/ddi-converter-tool
        
### Expose Kubernetes services on dataverse.tk
Some of the following steps will be done manually on AWS Route 53. 

On AWS Route 53, create hosted zone with domain name: dataverse.tk

![route53-hostedzone](readme-imgs/route53-hostedzone.png "route53-hostedzone")

![route53-hostedzone2](readme-imgs/route53-hostedzone2.png "route53-hostedzone2")

On Freenom, by clicking management tools - Use custom nameservers (enter below), change the DNS to Route 53. 

![freenom-dns](readme-imgs/freenom-dns.png "freenom-dns")

Deploy nginx:
    
        kubectl apply -k k8s/nginx

Confirm that the service has external ip.
        
        kubectl get svc --namespace=ingress-nginx
 
 ![ingress-nginx](readme-imgs/ingress-nginx.png "ingress-nginx")       
 
 Go to Route 53 and create dans.dataverse.tk, fileviewers.dataverse.tk and dct.dataverse.tk records with type CNAME 
 and the value is the external ip of ingress-nginx service.
 
 ![dans-dataverse-tk-cname](readme-imgs/dans-dataverse-tk-cname.png "dans-dataverse-tk-cname") 
 
 ![dans-dataverse-tk-cname2](readme-imgs/dans-dataverse-tk-cname2.png "dans-dataverse-tk-cname2") 
 
Now, the http://dans.dataverse.tk should be accessible from the browser.
     
### Setting certification manager to automated Let’s Encrypt ssl certification
 <a name="installing-helm">Installing _helm_ using brew</a>:
        
        brew install kubernetes-helm
        
Since Tiller Pod needs to run as a privileged service account, with cluter-admin ClusterRole, initialized helm with a service account, otherwise we will get "Error: release cert-manager failed: namespaces "kube-system" is forbidden".
        
         kubectl apply -f k8s/letscrypt/rbac-config.yaml
         helm init --service-account tiller
         helm repo update

Execute _helm ls_, there would not be any output from running this command and that is expected. 
         
         helm ls
                 
Install tiller service account:
        
        kubectl create serviceaccount tiller --namespace=kube-system
        kubectl create clusterrolebinding tiller-admin --serviceaccount=kube-system:tiller --clusterrole=cluster-admin
        kubectl apply -f k8s/letscrypt/00-crds.yaml
Output:
![crds](readme-imgs/crds.png "crds") 

Install certificate manager in kubernetes:
        
        kubectl label namespace default certmanager.k8s.io/disable-validation="true"
        helm repo add jetstack https://charts.jetstack.io
        helm repo update
        helm install --name cert-manager --namespace default jetstack/cert-manager
We can see what this process looks like:
![crt](readme-imgs/crt.png "crt")
    
Deploy certification issuer:

        kubectl create -f k8s/letscrypt/issure.yaml
        
Output:
![crti](readme-imgs/crti.png "crti")

Confirm the status of certificate:
    
        kubectl get certificate
        kubectl describe certificate
        
Output:
![crt-issuer](readme-imgs/crt-issuer.png "crt-issuer")

After waiting a bit, let's check it:
        
        helm list

Output:
![hls](readme-imgs/hls.png "hls")

Deploy ingress-nginx:
        
        kubectl apply -f k8s/letscrypt/ingress.yaml
        
Wait a little bit, and then confirm it using _curl_:
    
        curl --insecure -v https://dans.dataverse.tk 2>&1 | awk 'BEGIN { cert=0 } /^\* SSL connection/ { cert=1 } /^\*/ { if (cert) print }'
        
We can see:
![https](readme-imgs/https.png "https")

                
### Kubernetes Dashboard
The following steps are about installing the kubernetes dashboard version v1.1, not the latest version.
    
        kubectl apply -f k8s/dashboard/kubernetes-dashboard.yaml
        kubectl apply -f k8s/dashboard/heapster.yaml
        kubectl apply -f k8s/dashboard/influxdb.yaml
        kubectl apply -f k8s/dashboard/heapster-rbac.yaml
        kubectl apply -f k8s/dashboard/eks-admin-service-account.yaml
        
Retrieve the authentication token, using the following command:   
        
        kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
        
We need to copy the token to the clipboard.
![token](readme-imgs/token.png "token")

We access Dashboard using the following command:

        kubectl proxy

The Dashboard will available at http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/.

Choose the Token option, and then paste the token from the clipboard, and click the Sign In button.

        
### Setting Up Jenkins                