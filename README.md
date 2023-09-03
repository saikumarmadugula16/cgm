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
Multipass (for Ubuntu VM Creation)
K0sctl (for k0s cluster creation)

VM provisioning:
**SSH Key Generation:**
- Generate an SSH key on your local machine.

**Cloud-Init File Setup:**
- Create a cloud-init.yaml file.
- Copy the public SSH key into the cloud-init.yaml file to enable login to created VMs.

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





**Install MetalLB:**
- Apply MetalLB manifest: `kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml`
- Create MetalLB configuration and apply it: `kubectl apply -f .\metallb-config.yaml`
In metallb L2 advisement config file provide iprange.
- Test by applying a web server: `kubectl apply -f web-server.yaml`
- Verify the Service: `kubectl get svc`



**Install Nginx Ingress Controller:**
- Apply Nginx Ingress Controller manifest: `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml`
- Check Pods and Services: `kubectl get pods -n ingress-nginx`, `kubectl get services -n ingress-nginx`
- Set the default Ingress class: `kubectl -n ingress-nginx annotate ingressclasses nginx ingressclass.kubernetes.io/is-default-class="true"`



**Install ArgoCD:**
- Create ArgoCD namespace: `kubectl create namespace argocd`
- Apply ArgoCD install manifest: `kubectl apply -n argocd -f argocd-install.yaml`
- Apply ArgoCD network ingress: `kubectl apply -n argocd -f argo-ingress.yaml’
- Patch ArgoCD Server service to use NodePort: `kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'`
- Get ArgoCD admin password: `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`
- Install Helm using Scoop: `scoop install helm`
- Add ArgoCD Helm repository: `helm repo add argo https://argoproj.github.io/argo-helm`
- Update Helm repositories: `helm repo update`
- Install ArgoCD using Helm: `helm install argocd argo/argo-cd --namespace argocd`

**ArgoCD Access:**
- Access the ArgoCD UI in your browser: `<worker-external-ip>:<node-port>`
- Example: http://172.26.238.123:30027/
- Login with the provided username and password.
- Default username is ‘admin’	

**Install MySQL via ArgoCD UI:**
- Add a Helm repo in the ArgoCD configuration (Bitnami Helm charts repository).
- Create a MySQL application in ArgoCD, specifying the name, project, Helm chart (mysql), version, cluster, and namespace.
- Choose manual synchronization.
- Initiate a synchronization to deploy MySQL.
- Monitor the application status.


Remaining Task I will work in upcoming days. 
