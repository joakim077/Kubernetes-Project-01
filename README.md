# Kubernetes-Project-01
Deploying an app to EKS cluster and Creating a CICD pipeline

## Create image and push to dockerHub
` docker login -u your_username `                  
` docker build --no-cache --platform=linux/amd64 -t myapp . `                         
` docker tag your_image your_username/your_repo_name `                                        
` docker push your_username/your_repo_name `

## Create secret 
```
kubectl create secret generic my-postgresql-credentials --from-literal=password='new_password'  --from-literal=username='goals_user'  --dry-run=client -o yaml | kubectl apply -f -
```

## Creating SC in EKS
```
Add on AWS EBS CSI Drive 
create Storage class
Attach Policy to node role - AmazonEBSCSIDriverkubectl

apply -f storageclass-gp2.yaml
```

## Install cloudnative PG
```
kubectl apply --server-side -f \
  https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/release-1.23/releases/cnpg-1.23.1.yaml
```
## create Database cluster
`kubectl apply -f postgres-cluster.yaml`


## Exec into pod to create table

```
kubectl exec -it my-postgresql-1 -- psql -U postgres -c "ALTER USER goals_user WITH PASSWORD 'new_password';"

kubectl exec -it my-postgresql-1 -- bash

PGPASSWORD='new_password' psql -h 127.0.0.1 -U goals_user -d goals_database -c "
CREATE TABLE goals (
    id SERIAL PRIMARY KEY,
    goal_name VARCHAR(255) NOT NULL
);
"
```

## Deploy app and Service
kubectl apply -f deploy.yaml

kubectl apply -f service.yaml



===============================================
## Install CERT MANAGER
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.5/cert-manager.yaml

## Install nginx ingress controller 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/cloud/deploy.yaml

An ELB will be created and you will get either IP or DNS 
add A (IP) or CNAME (DNS) record for you domain 

kubectl apply -f cluster_issuer.yaml                                                          
kubectl apply -f certificate                                                                  
kubectl apply -f ingress.yaml

## Install Metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl apply -f hpa.yaml

Install Grafan K6

k6s run load.js

===============================================

# GithubActions and ArgoCD

## Steps 
Create .github/workflows folder                                                                                                          
  Create a file build-push-image.yaml  
  
Create a jinja template /tmpl/deploy.j2 

Create deployment file - /deploy/deploy.yaml   

Create GitHub Actions secret - DOCKERHUB_USERNAME and DOCKERHUB_PASSWORD                                                                 
Make sure your actions have push access as well.  

## Install ArgoCd
```
kubectl create namespace argocd                                                                                                           
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml                               
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'                                                         
kubectl get secret -n argocd argocd-initial-admin-secret -oyaml                                                                           

```

