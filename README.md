# KUBERNETES HELM POC
With the release of helm3 , tiller is no longer a part of helm. As a result it is now much simpler and easier to use.

Assuming helm is already installed.

- A Chart is a Helm package. It contains all of the resource definitions necessary to run an application, tool, or service inside of a Kubernetes cluster. Think of it like the Kubernetes equivalent of a Homebrew formula, an Apt dpkg, or a Yum RPM file.

- Helm installs charts into Kubernetes, creating a new release for each installation. And to find new charts, you can search Helm chart repositories.

### 'helm search': Finding Charts

Helm comes with a powerful search command. It can be used to search two different types of source:

- `helm search` hub searches the Artifact Hub, which lists helm charts from dozens of different repositories.
- ` helm search repo` searches the repositories that you have added to your local helm client (with helm repo add). This search is done over local data, and no public network connection is needed.
You can find publicly available charts by running helm search hub:

`$ helm search hub wordpress`
URL                                                 CHART VERSION APP VERSION DESCRIPTION
https://hub.helm.sh/charts/bitnami/wordpress        7.6.7         5.2.4       Web publishing platform for building blogs and ...
https://hub.helm.sh/charts/presslabs/wordpress-...  v0.6.3        v0.6.3      Presslabs WordPress Operator Helm Chart
https://hub.helm.sh/charts/presslabs/wordpress-...  v0.7.1        v0.7.1      A Helm chart for deploying a WordPress site o

### 'helm install': Installing a Package

To install a new package, use the helm install command. At its simplest, it takes two arguments: A release name that you pick, and the name of the chart you want to install.

`$ helm install happy-panda bitnami/wordpress`
NAME: happy-panda
LAST DEPLOYED: Tue Jan 26 10:27:17 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    happy-panda-wordpress.default.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w happy-panda-wordpress'

   export SERVICE_IP=$(kubectl get svc --namespace default happy-panda-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default happy-panda-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)

---
Now the wordpress chart is installed. Note that installing a chart creates a new release object. The release above is named happy-panda. (If you want Helm to generate a name for you, leave off the release name and use --generate-name.)

To keep track of a release's state, or to re-read configuration information, you can use
`helm status`

### Customizing the Chart Before Installing

To see what options are configurable on a chart, use `helm show values`:

`$ helm show values bitnami/wordpress`

You can then override any of these settings in a YAML formatted file, and then pass that file during installation.

`$ echo '{mariadb.auth.database: user0db, mariadb.auth.username: user0}' > values.yaml
$ helm install -f values.yaml bitnami/wordpress --generate-name`

###  'helm upgrade' and 'helm rollback': Upgrading a Release, and Recovering on Failure
When a new version of a chart is released, or when you want to change the configuration of your release, you can use the `helm upgrade` command.

### Helpful Options for Install/Upgrade/Rollback

There are several other helpful options you can specify for customizing the behavior of Helm during an install/upgrade/rollback. Please note that this is not a full list of cli flags. To see a description of all flags, just run `helm <command> --help`.

 --timeout: A Go duration value to wait for Kubernetes commands to complete. This defaults to 5m0s.
--wait: Waits until all Pods are in a ready state, PVCs are bound, Deployments have minimum (Desired minus maxUnavailable) Pods in ready state and Services have an IP address (and Ingress if a LoadBalancer) before marking the release as successful. It will wait for as long as the --timeout value. If timeout is reached, the release will be marked as FAILED. Note: In scenarios where Deployment has replicas set to 1 and maxUnavailable is not set to 0 as part of rolling update strategy, --wait will return as ready as it has satisfied the minimum Pod in ready condition.
--no-hooks: This skips running hooks for the command
--recreate-pods (only available for upgrade and rollback): This flag will cause all pods to be recreated (with the exception of pods belonging to deployments). (DEPRECATED in Helm 3)

## PERSONAL OBSERVATIONS
Pre
Export minikube ip to var:
`export kip=$(minikube ip)`
`echo $kip`
This is how we'll access our node, using this handy variable.

To init helm
`helm init`

To install Chart in cluster
`helm install --name k8s-helm-poc .`

You must point to relative directory where `Chart.yaml` is situated. Hence `.`. `--name k8s-helm-poc` gives the release a custom name, otherwise is autogenerated.

To view release status and created deployment.
Using helm:

`$ helm status k8s-helm-poc`

GIVES : 


    LAST DEPLOYED: Sun OCT 18 23:23:22 2021
    NAMESPACE: default
    STATUS: DEPLOYED
    
    RESOURCES:
    ==> v1beta2/Deployment
    NAME          DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
    k8s-helm-poc  1        1        1           1          19s
	
You may also view it using kubectl:

`kubectl describe deployment k8s-helm-poc`

View pods:

`$ kubectl get pods -l app=k8s-helm-poc`



    NAME                            READY     STATUS    RESTARTS   AGE
    k8s-helm-poc-598bbdccc9-gn9vk   1/1       Running   0          5m
	
Upgrading replicate count and upgrade using helm
Change replicateCount of deploy to 4.

And issue following command:

`helm upgrade k8s-helm-poc .`
This will create a new release as seen below.



    $ helm ls
    NAME          REVISION  UPDATED                   STATUS    CHART               NAMESPACE
    k8s-helm-poc  2         Sun OCT 18 23:30:57 2021  DEPLOYED  k8s-helm-poc-0.0.1  default
	
Kubectl will confirm increase of replicas.



    $ kubectl get deploy
    NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    k8s-helm-poc   4         4         4            4           9m
	

To install on different namespace:
`$ helm --namespace=development install --name k8s-helm-poc . $ helm --namespace=test install --name k8s-helm-poc .`

------------


## USING HELM CHART WITH TERRAFORM ON AWS EKS
####Pre-requisites
- Terraform installed
- AWS cli installed on a host to connect to the cluster
- AWS credentials configured
- kubectl installed on a host to deploy to the cluster

####Deployment Instructions
- Install Terraform
- Clone this repository
- Edit the jenkins-values.yaml to match your values
- Edit or remove the set_sensitive arguments in helm_release.tf if you would like to
- Run a terraform init to grab providers and modules
- Run aws_configure and establish your credentials
- Run a terraform_apply and wait 10 - 15 minutes. Note: If it fails for HTTP timeout while waiting to apply the Helm chart, retry terraform_apply
>  Sometimes it takes the worker nodes a little longer to comeup and we will get a HTTP timeout as terraform is waiting for them to comeup. So if we do get this error,
simply run `terraform apply again` and it will rerun just the jenkins helm chart ( can be any chart) using the helm provider and apply that.

- Run aws eks --region ap-south-1 update-kubeconfig --name dev-cluster to add the cluster context to your kubeconfig
- Run kubectl get pods and kubectl get svc to ensure Jenkins deployed as expected
- Run kubectl patch svc jenkins -p '{"spec": {"type": "LoadBalancer"}}' to change the Service to type LoadBalancer
- Run kubectl get svc again to grab the AWS created DNS address

Go to your browser and navigate to http://<dns-address>:8080 Note: This may take 3 - 5 minutes to resolve while waiting for Jenkins to fully initialize.
Log in with the credentials you set in the jenkins-values.yaml or the helm_release.tf


#### For our helm_release.tf file

our provider is helm, and in that we have our k8s configured.
There are various ways of authentication,
https://registry.terraform.io/modules/terraform-module/release/helm/latest?tab=inputs

For our implementation , we used an exec plugin where we get an authentication token.

for the "host"
the host in eks-cluster.tf needs to match the host in helm provider.

For the "resource"
each helm chart is an new resource ,
the name is helm repo name,repo is the repository we are getting the resource from.
eg. `helm repo add jenkins < url of the repo> `
and then 
`helm search repo jenkins`

to search the repo you just added.

Valid arguments are really necessary .

for values, we have defined a clear path ( not mandatory)
and the secrets are set sensitive too.

## BENIFITS OF USING HELM CHARTS

### First: Deployment Speed

### Second: Helm chart on Kubernetes for application configuration

significant benefit of deploying infrastructure using Helm is the use of prebuilt configurations. You are no longer building your configuration from scratch and diving into docs to get a sample application running — most Helm chart values.yaml files will not only be created with starting users' best interest but will also typically include commented out features that you may be interested in as an advanced user. With that being said, I do caution that you truly understand a service and have read up on its documentation if you intend to roll it into a production environment.

### Third: Application testing

Engineers have spent hours building out these great Helm charts, and we can reap what they have sown. Like any engineer, they expect failure and design with that in mind. In many Helm chart repo’s you will find the beautiful addition of tests. These tests can range from proper load testing in your deployment to simple tests for your configuration and making sure your services are running properly.

#### SOME BEST PRACTICES FOR USING HELM CHARTS

https://codersociety.com/blog/articles/helm-best-practices
