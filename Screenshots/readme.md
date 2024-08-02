# kubernetes-playground
## Create an nginx deployment in k8s with only 1 replica. 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

``` kubectl apply -f nginx.yaml ```

## Scale the deployment to 3 replicas

```kubectl scale deployment nginx-deployment --replicas=3```

## Exposing teh deployment to port 80

```kubectl expose deployment nginx-deployment --port=80 --target-port=80 --name=nginx-service --type=ClusterIP```

## Getting info about deployments 

```kubectl get deployments```
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
kubia              1/1     1            1           114m
nginx-deployment   3/3     3            3           11m
```

```kubectl get pods -o wide```

```
NAME                                READY   STATUS    RESTARTS   AGE    IP            NODE     NOMINATED NODE   READINESS GATES
kubia-587b7cd6b7-vsh2p              1/1     Running   0          114m   10.42.0.99    pop-os   <none>           <none>
nginx-deployment-7c79c4bf97-5l4pg   1/1     Running   0          11m    10.42.0.102   pop-os   <none>           <none>
nginx-deployment-7c79c4bf97-7n8v2   1/1     Running   0          11m    10.42.0.101   pop-os   <none>           <none>
nginx-deployment-7c79c4bf97-m9kbd   1/1     Running   0          11m    10.42.0.100   pop-os   <none>           <none>
```

```kubectl get service nginx-service```
```
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   10.43.240.244   <none>        80/TCP    1
```

## Curling the pod from inside another pod 

```kubectl run curlpod --image=radial/busyboxplus:curl -i --tty --rm --restart=Never -- /bin/sh```

```
If you don't see a command prompt, try pressing enter.
[ root@curlpod:/ ]$ curl 10.43.240.244
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[ root@curlpod:/ ]$ exit
pod "curlpod" deleted
```

## delete pod 

```kubectl delete pod -l app=nginx```

```
pod "nginx-deployment-7c79c4bf97-5l4pg" deleted
pod "nginx-deployment-7c79c4bf97-7n8v2" deleted
pod "nginx-deployment-7c79c4bf97-m9kbd" deleted
```

## trying to kill the process 

```kubectl exec pod/nginx-deployment-7c79c4bf97-p67tg -- pkill nginx```

```
error: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "9cfbcfb1705b93d0f92aa238b65376035d3fcd38e0c460ee84c19e71c4e0a72a": OCI runtime exec failed: exec failed: unable to start container process: exec: "pkill": executable file not found in $PATH: unknown
```
