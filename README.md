Step 1: Deploy a Basic Nginx Pod
Create a Pod manifest for Nginx:
```yaml
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
```
Step 2: Introduce an Init Container
Update the Pod manifest to include an init container:
```yaml
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
```
Step 3: Add a Liveness Probe
Update the Pod manifest to include a liveness probe:
```yaml
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
```
Step 4: Add a HostPath Volume for Logs
```yaml
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
```
