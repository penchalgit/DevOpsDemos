#### Setting up Helm on kubernetes cluster


Helm is a tool created to streamline the installation and management of Kubernetes applications. You can think of Helm like the YUM / APT or Homebrew package managers for Kubernetes.

There are few prerequisites required for a successful installation and operation of Helm.
1.A Kubernetes cluster
2.Admin access to install Tiller
3.Locally configured kubectl.

Step 1: Installing Helm client
Helm client runs on your laptop, CI/CD pipelines, etc. The installation of helm client is simplified for you through bash script.
````sh
curl -L https://git.io/get_helm.sh | bash
```` 

Here is the expected installation output:
````
Helm v2.16.1 is available. Changing from version .
Downloading https://get.helm.sh/helm-v2.16.1-linux-amd64.tar.gz
Preparing to install helm and tiller into /usr/local/bin
helm installed into /usr/local/bin/helm
tiller installed into /usr/local/bin/tiller
Run 'helm init' to configure helm.
````
The helm binary package will be installed to /usr/local/bin/ directory.
````
$ helm version
Client: &version.Version{SemVer:"v2.16.1", GitCommit:"bbdfe5e7803a12bbdf97e94cd847859890cf4050", GitTreeState:"clean"}
````
Step 2: Create Tiller service account & Role binding
````
Helm2 has a server component called Tiller.
We need to create service account for Tiller with admin access to the cluster. Create a new file called tiller-serivice-account.yaml.
````
$ vim tiller-account-rbac.yaml
````
paste the below content in tiller-account-rbac.yaml
````sh
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
	
````

From the manifest definition, we have created a ClusterRoleBinding with cluster-admin permissions to the tiller service account.

Create the resources in Kubernetes using the kubectl command:
````
$ kubectl apply -f tiller-account-rbac.yaml
serviceaccount/tiller created
clusterrolebinding.rbac.authorization.k8s.io/tiller created
````
Confirm creation of these objects:
````
$ kubectl get serviceaccount tiller -n kube-system        
NAME     SECRETS   AGE
tiller   1         64s
````
$ kubectl get clusterrolebinding tiller -n kube-system
NAME     AGE
tiller   100s
````
Step 3: Deploy Tiller and Initialize Helm
````
Now initialize Helm using the command below.
````sh
$ helm init --service-account=tiller \
   --stable-repo-url=https://kubernetes-charts.storage.googleapis.com \
   --upgrade \
   --automount-service-account-token=true \
   --replicas=1 \
   --history-max=100 \
   --wait
````   
Below is the output from the helm init command.
````
you see a messgage  -> tiller is installed in your kubernetes cluster

````sh
$ ls ~/.helm 
cache  plugins  repository  starters
````
On the kubernetes end, you should see a new deployment called tiller-deploy.
````
kubectl get deployment  -n kube-system
````  
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server           1/1     1            1           20d
local-path-provisioner   1/1     1            1           20d
coredns                  1/1     1            1           20d
traefik                  1/1     1            1           20d
tiller-deploy            1/1     1            1           63m
````
$ kubectl get deployment tiller-deploy -n kube-system -o wide
````
NAME            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                  SELECTOR
tiller-deploy   1/1     1            1           64m   tiller       gcr.io/kubernetes-helm/tiller:v2.16.1   app=helm,name=tiller
````
