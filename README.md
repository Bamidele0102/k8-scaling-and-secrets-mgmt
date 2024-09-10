# k8s-scaling-and-secrets-mgmt

Exercise 1: Build and deploy an application to Kubernetes

Step 1: Build the Docker image
docker build . -t us.icr.io/$MY_NAMESPACE/myapp:v1 

Step 2: Push and list the image
docker push us.icr.io/$MY_NAMESPACE/myapp:v1

Step 3: Deploy your application
kubectl apply -f deployment.yaml
kubectl get pods

Step 4: View the application output
kubectl port-forward deployment.apps/myapp 3000:3000 

- Create a ClusterIP service for exposing the application to the internet:
kubectl expose deployment/myapp


Exercise 2: Implement Vertical Pod Autoscaler (VPA)

Step 1: Create a VPA configuration
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myvpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"  # VPA will automatically update the resource requests and limits
```
Step 2: Apply the VPA
kubectl apply -f vpa.yaml

Step 3: Retrieve the details of the VPA
kubectl get vpa

kubectl describe vpa myvpa

Exercise 3: Implement Horizontal Pod Autoscaler (HPA)

Step 1: Create an HPA configuration
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: myhpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 1         # Minimum number of replicas
  maxReplicas: 10        # Maximum number of replicas
  targetCPUUtilizationPercentage: 5  # Target CPU utilization for scaling
```

Step 2: Configure the HPA
kubectl apply -f hpa.yaml

Step 3: Verify the HPA
kubectl get hpa myhpa

Step 4: Start the Kubernetes proxy
kubectl proxy

Step 5: Spam and increase the load on the app
for i in `seq 100000`; do curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/myapp/proxy; done

Step 6: Observe the effect of autoscaling
kubectl get hpa myhpa --watch

Step 7: Observe the details of the HPA
kubectl get hpa myhpa


Exercise 4: Create a Secret and update the deployment

Step 1: Create a Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret
type: Opaque
data:
  username: bXl1c2VybmFtZQ==
  password: bXlwYXNzd29yZA==
```

Step 2: Update the deployment to utilize the secret
Add the following lines at the end of deployment.yaml:
```yaml
        env:
        - name: MYAPP_USERNAME
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: username
        - name: MYAPP_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: password
```

Step 3: Apply the secret and deployment
kubectl apply -f secret.yaml
Apply the updated deployment using the following command:
kubectl apply -f deployment.yaml

Step 4: Verify the secret and deployment
kubectl get secret
Run the following command to show the status of the deployment, including information about replicas and available replicas
kubectl get deployment
