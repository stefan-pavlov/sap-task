# Update system
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y curl wget git net-tools vim

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

# Install helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

# Install minikube
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

# Start Minikube
minikube start --driver=docker

# deploy ingress controller with helm
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace

# Setup GitHub Runner

# Install it as a service
sudo ./svc.sh install
sudo ./svc.sh start
sudo ./svc.sh enable

# Configure GitHub Secrets
Add the secrets to the repository

DOCKER_USERNAME
DOCKER_PASSWORD
SONAR_TOKEN

#Install Metrics Server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

#Patch the Deployment
kubectl patch deployment metrics-server -n kube-system \
  --type='json' -p='[
    {
      "op": "add",
      "path": "/spec/template/spec/containers/0/args/-",
      "value": "--kubelet-insecure-tls"
    }
  ]'

#test metric server:
kubectl get deployment metrics-server -n kube-system
kubectl top pods
watch kubectl get hpa

#Install Cert-Manager:
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
  
#Create a Self-Signed ClusterIssuer
file in the repo: selfsigned-issuer.yaml and selfsigned-cert.yaml


# Add the repo to Sonar
make it as "With GitHub Actions"
take the info from information tab

# Deploy
Push to the main branch and Actions will:

Build and push the Docker image
Deploy to Minikube
