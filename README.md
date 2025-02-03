#STEP1: First, isntall the kind cluster with given conf file. 
#Download config file
curl -Lo kind-config.yaml https://raw.githubusercontent.com/gireesh-nv/mongo_withargo/refs/heads/main/mongoargo_kind.yaml
#install kind cluster
kind create cluster --config kind-config.yaml --retain --image "kindest/node:v1.21.14

#STEP 2: INSTALLING AND CONFIGURING ARGOCD
#After Kind is set up, install Argo CD:
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

#STEP 3: Run the following command to change ArgoCD service to match your Kind port mappings:

kubectl patch svc argocd-server -n argocd -p '{
  "spec": {
    "type": "NodePort",
    "ports": [
      {
        "name": "http",
        "port": 80,
        "targetPort": 8080,
        "nodePort": 30090
      },
      {
        "name": "https",
        "port": 443,
        "targetPort": 8080,
        "nodePort": 30091
      }
    ]
  }
}'

#verify the change
kubectl get svc argocd-server -n argocd

#STEP 4: Now, access ArgoCD UI using your EC2 public IP:
	•	HTTP: http://<EC2-PUBLIC-IP>:8082
	•	HTTPS: https://<EC2-PUBLIC-IP>:8444

#Retrieve argocd admin password
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode


#STEP 5: Install mongodb with HELM
helm repo add mongodb https://mongodb.github.io/helm-charts;
helm install enterprise-operator mongodb/enterprise-operator --namespace mongodb --create-namespace;
kubectl describe deployments mongodb-enterprise-operator -n mongodb;
