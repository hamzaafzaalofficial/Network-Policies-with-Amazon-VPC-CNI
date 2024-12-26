# Networking Policy Development within EKS  

## Objective  
Develop a networking policy within EKS by deploying multi-tier applications and implementing network policies using Amazon VPC CNI.  

## Prerequisites  
1. **AWS Account**: Ensure you have access to an AWS account.  
2. **AWS CLI**: Installed and configured with access keys.  
3. **kubectl Tool**: Installed to manage EKS clusters.  
4. **EKS Cluster**: Running with nodes launched.  
5. **Amazon VPC CNI**: Ensure it's installed in your EKS cluster. By default, EKS uses the Amazon VPC CNI.  

## Tasks Overview  
1. Deploy multi-tier applications on EKS.  
2. Implement network policies using Amazon VPC CNI.  
3. Test network segmentations.  

## Documentation  
- Basics of Kubernetes networking.  
- Amazon VPC CNI overview.  
- Crafting network policies.  

## Tasks  

### 1. Deploy Multi-Tier Applications on EKS  

For demonstration purposes, let's deploy a basic frontend and backend. Run the following commands to set up your environment:  

Make the deployment script executable and run it:  

```bash  
chmod +x deployments.sh  
bash deployments.sh  
```  

Create a namespace:  

```bash  
kubectl create namespace multi-tier-app  
```  

#### Deploy Backend  

```bash  
kubectl apply -f - <<EOF  
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: backend-deployment  
  namespace: multi-tier-app  
spec:  
  replicas: 2  
  selector:  
    matchLabels:  
      app: backend  
  template:  
    metadata:  
      labels:  
        app: backend  
    spec:  
      containers:  
      - name: backend  
        image: nginx  
EOF  
```  

#### Deploy Frontend  

```bash  
kubectl apply -f - <<EOF  
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: frontend-deployment  
  namespace: multi-tier-app  
spec:  
  replicas: 2  
  selector:  
    matchLabels:  
      app: frontend  
  template:  
    metadata:  
      labels:  
        app: frontend  
    spec:  
      containers:  
      - name: frontend  
        image: nginx  
EOF  
```  

#### Create Backend Service  

```bash  
kubectl apply -f - <<EOF  
apiVersion: v1  
kind: Service  
metadata:  
  name: backend-deployment  
  namespace: multi-tier-app  
spec:  
  selector:  
    app: backend  
  ports:  
  - protocol: TCP  
    port: 80  
    targetPort: 80  
EOF  
```  

### 2. Implement Network Policies Using Amazon VPC CNI  

The following policy allows only the frontend to communicate with the backend:  

```bash  
kubectl apply -f - <<EOF  
apiVersion: networking.k8s.io/v1  
kind: NetworkPolicy  
metadata:  
  name: backend-network-policy  
  namespace: multi-tier-app  
spec:  
  podSelector:  
    matchLabels:  
      app: backend  
  policyTypes:  
  - Ingress  
  ingress:  
  - from:  
    - podSelector:  
        matchLabels:  
          app: frontend  
EOF  
```  

### 3. Test Network Segmentations  

Now, try communicating from the frontend pod to the backend pod.  

#### Get Frontend Pod Name  

```bash  
FRONTEND_POD=$(kubectl get pods -n multi-tier-app -l app=frontend -o jsonpath='{.items[0].metadata.name}')  
```  

#### Try Reaching Backend from Frontend  

```bash  
kubectl exec -n multi-tier-app $FRONTEND_POD -- curl backend-deployment  
```  

### 4. Verify the Network Policy  

Use the following command to check if the network policy has been applied:  

```bash  
kubectl get networkpolicy -n multi-tier-app  
```  

You should see `backend-network-policy` in the list, which means the policy is applied successfully.
