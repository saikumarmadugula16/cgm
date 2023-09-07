CGM_Task
1. Install 2 VM’s with Ubuntu
2. Install k0s controller on the first node, on the second node install k0s worker
3. Install localopenebs storage via k0s config
4. Check that all necessary services are installed, like coredns,metricsserver, storage
5. Install metalLB with L2 network advisement
6. Install ArgoCD within the cluster
7. Install nginx ingress controller
8. Expose ArgoCD-Server via ingress, use passthrough SSL (please check the ArgoCD manual)
9. Find out the admin credentials for ArgoCD
10. Create an argo application via ui, please install mysql in the default namespace
11. Create a Namespace payroll
a. Install Wordpress within via ArgoCD
b. Connect Wordpress with the newly created database
12. Expose Wordpress application via Ingress with path and hostname or your choice


Tasks Notes:
Tools list:
Multipass (for Ubuntu VM Creation) - 1.12.2+win
K0sctl (for k0s cluster creation) - v0.15.5
Helm v3.12.3

VM provisioning:
**SSH Key Generation:**
- Generate an SSH key on your local machine.

**Cloud-Init File Setup:**
- Create a cloud-init.yaml file.
- Copy the public SSH key into the cloud-init.yaml file to establish connection to created VMs from local VSCode terminal.

**Create Controller and Worker VMs:**
- Use `multipass` to launch the controller and worker VMs:
  - `multipass launch -c 2 -m 2G -d 10G -n controller --cloud-init .\cloud-init.yaml`
  - `multipass launch -c 2 -m 2G -d 10G -n worker --cloud-init .\cloud-init.yaml`

**Install k0sctl:**
- Download k0sctl for Windows from the provided link.
https://github.com/k0sproject/k0sctl/releases/download/v0.15.5/k0sctl-win-x64.exe

- Initialize k0sctl: `k0sctl init --k0s > k0s.yaml`
- Edit k0s.yaml to add IP addresses for the controller and worker.
- Apply the k0s configuration: `k0sctl apply --config k0s.yaml`
- Get the kubeconfig: `k0sctl kubeconfig --config .\k0s.yaml`
- Optionally, save the kubeconfig to the default location: `k0sctl kubeconfig --config .\k0s.yaml > ~/.kube/config`

**Check K0s Pods are present**
kubectl get pods -A

**Install MetalLB:**
Before Installing Metallb make this changes 
kubectl edit configmap -n kube-system kube-proxy
change Iptabels to ipvs and strictARP: true
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true

- Apply MetalLB manifest: `kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml`
- Create MetalLB configuration and apply it: `kubectl apply -f .\metallb-config.yaml`
In metallb L2 advisement config file provide iprange.

- check Ipddress pool
kubectl -n metallb-system get IPAddressPool

- check l2 L2Advertisement
kubectl get l2advertisement -n metallb-system



**Install Nginx Ingress Controller:**
- Apply Nginx Ingress Controller manifest: `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml`
- Check Pods and Services: `kubectl get pods -n ingress-nginx`, `kubectl get services -n ingress-nginx`
 #optional - kubectl edit svc ingress-nginx-controller -n ingress-nginx 
  change NodePort to LoadBalancer however making this change mandatory for production applications. 

#optional- Set the default Ingress class: `kubectl -n ingress-nginx annotate ingressclasses nginx ingressclass.kubernetes.io/is-default-class="true"` 
This can be useful if we have morethan one ingressclasses



**Install ArgoCD:**
- Create ArgoCD namespace: `kubectl create namespace argocd`
- Install Helm using Scoop: `scoop install helm`
- Add ArgoCD Helm repository: `helm repo add argo https://argoproj.github.io/argo-helm`
- Update Helm repositories: `helm repo update`
- Install ArgoCD using Helm: `helm install argocd argo/argo-cd --namespace argocd`
- Get ArgoCD admin password: `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`
  
Expose ArgoCD-Server via ingress, use passthrough SSL
Kubectl apply -f argo-ingress.yaml -n argocd
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
Add ingress hostname in C:\Windows\System32\drivers\etc\hosts
kubectl -n argocd port-forward pod/argocd-server-6b5ffb984b-75xq7 --address 0.0.0.0 80:80 443:443
Getting Issues......
So, I followed a traditional method.
- Disable Certificate check: kubectl -n argocd edit deployments.app argocd-server
add --insecure to containers spec
- Patch ArgoCD Server service to use NodePort: `kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'`

**ArgoCD Access:**
- Access the ArgoCD UI in your browser: `<worker-external-ip>:<node-port>`

- Login with the provided username and password.
- Default username is ‘admin’	

**Install MySQL and Wordpress via ArgoCD UI:**

MySQL:
Using deployment and service files to install Mysql applications. 
First Install mysql in the default namespace by using git repo: https://github.com/saikumarmadugula16/cgm.git
Path: mysqltest

The above mentioned repo consists of deployment and service files. Once Application creation completed get the NodePort port details.


Wordpress:

First, copy and place the  <worker-ip>:<mysql-service-nodeport> into wordpress deployment env, name: WORDPRESS_DB_HOST.

Using deployment,  service and ingress files to install Wordpress applications. 

Kubectl create ns payroll (or)  can use auto create namespace option in ArgoCD UI.

Install wordpress application in the payroll namespace by using git repo: 
https://github.com/saikumarmadugula16/cgm.git
Path: wordpresstest


C:\Windows\System32\drivers\etc\hosts local server. Then applied port-forward to access the application by using host.
Optional: kubectl patch svc wordpress-service -n payroll -p '{"spec": {"type": "LoadBalancer"}}'
kubectl -n payroll port-forward pod/wordpress-deployment-7b8b5d854f-sdpdt --address 0.0.0.0 80:80 443:443.


				—------ END —---

References:
https://multipass.run/install
https://docs.k0sproject.io/v1.23.6+k0s.2/
https://argo-cd.readthedocs.io/en/stable/operator-manual/installation/
https://discuss.kubernetes.io/t/error-establishing-a-database-connection-error-kubernates-wordpress-mysql/15407
https://github.com/sugreevudu/kubernetes/tree/master/wordpre-phpmysql-mysql-deployments
