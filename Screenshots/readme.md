Step 1: Deploy a Basic Nginx Pod
Create a Pod manifest for Nginx:
yaml
Copy code
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
Apply the manifest to your cluster:
bash
Copy code
kubectl apply -f nginx-pod.yaml
Verify the Pod is running:
bash
Copy code
kubectl get pods
kubectl port-forward pod/nginx-pod 8080:80
Access the Nginx default page:

Open a browser and navigate to http://localhost:8080 to see the default Nginx page.
Screenshot: Take a screenshot showing the Nginx default page.

Step 2: Introduce an Init Container
Update the Pod manifest to include an init container:
yaml
Copy code
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}

  initContainers:
  - name: init-fetch-index
    image: curlimages/curl:latest
    command: ['sh', '-c', 'curl -o /mnt/index.html http://example.com/index.html']
    volumeMounts:
    - name: shared-data
      mountPath: /mnt

  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
Apply the updated manifest:
bash
Copy code
kubectl apply -f nginx-pod.yaml
Verify the Pod is running:
bash
Copy code
kubectl get pods
kubectl logs pod/nginx-pod -c init-fetch-index
Port-forward and check the custom page:
bash
Copy code
kubectl port-forward pod/nginx-pod 8080:80
Open a browser and navigate to http://localhost:8080 to see the custom index.html served by Nginx.
Screenshot: Take a screenshot showing the custom Nginx page.
Step 3: Add a Liveness Probe
Update the Pod manifest to include a liveness probe:
yaml
Copy code
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}

  initContainers:
  - name: init-fetch-index
    image: curlimages/curl:latest
    command: ['sh', '-c', 'curl -o /mnt/index.html http://example.com/index.html']
    volumeMounts:
    - name: shared-data
      mountPath: /mnt

  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
Apply the updated manifest:
bash
Copy code
kubectl apply -f nginx-pod.yaml
Verify the liveness probe:
bash
Copy code
kubectl describe pod/nginx-pod
kubectl get pods
Test liveness probe:

Modify the Nginx configuration to serve a /healthz endpoint if needed, and verify that the liveness probe restarts the container if it fails.
Screenshot: Take a screenshot showing the Pod's status and liveness probe in action.

Step 4: Add a HostPath Volume for Logs
Update the Pod manifest to include a hostPath volume:
yaml
Copy code
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  - name: nginx-logs
    hostPath:
      path: /var/log/nginx
      type: DirectoryOrCreate

  initContainers:
  - name: init-fetch-index
    image: curlimages/curl:latest
    command: ['sh', '-c', 'curl -o /mnt/index.html http://example.com/index.html']
    volumeMounts:
    - name: shared-data
      mountPath: /mnt

  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
    - name: nginx-logs
      mountPath: /var/log/nginx
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
Apply the updated manifest:
bash
Copy code
kubectl apply -f nginx-pod.yaml
Verify the logs are being written to the hostPath volume:
bash
Copy code
kubectl exec -it nginx-pod -- cat /var/log/nginx/access.log
Check the hostPath on the node:

SSH into the node if necessary and verify that logs are being stored at /var/log/nginx.
Screenshot: Take a screenshot showing the logs being written to the hostPath volume.

Final Steps
Clean up:

After verifying that everything works, you can delete the resources with:
bash
Copy code
kubectl delete pod nginx-pod
Document & Review:

Review your screenshots and ensure all steps are correctly documented.
