# check

логінимось і пушимо образ у Docker Hub

```
docker login
```
```
Login Succeeded
```

```
docker tag course-app:latest rubanik11/course-app:latest
```
```
```

```
docker push rubanik11/course-app:latest
```
```
The push refers to repository [docker.io/rubanik11/course-app]
9e8ce6328d2a: Pushed
... (інші шари — Layer already exists)
latest: digest: sha256:9373f872080a74914339610fbfcda4f15a4f4bce1de5d394e7b8b59c4f399a98 size: 856
```

перевіряємо кластер

```
kubectl get nodes
```
```
NAME             STATUS   ROLES           AGE   VERSION
docker-desktop   Ready    control-plane   32h   v1.34.1
```

накатуємо deployment і service

```
kubectl apply -f deployment.yaml
```
```
deployment.apps/course-app created
```

```
kubectl apply -f service.yaml
```
```
service/course-app created
```

```
kubectl get deploy
```
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
course-app   2/2     2            2           22s
```

```
kubectl get pods
```
```
NAME                          READY   STATUS    RESTARTS   AGE
course-app-64cff7b856-4tvsz   1/1     Running   0          22s
course-app-64cff7b856-lqvnc   1/1     Running   0          22s
```

```
kubectl get svc
```
```
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
course-app   NodePort    10.103.170.110   <none>        8080:30080/TCP   22s
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          32h
```

```
curl -i http://localhost:30080/healthz
```
```
HTTP/1.1 200 OK
content-type: application/json

{"status":"ok"}
```

міняємо replicas з 2 на 3 і дивимось rollout

```
kubectl apply -f deployment.yaml
```
```
deployment.apps/course-app configured
```

```
kubectl rollout status deployment/course-app
```
```
deployment "course-app" successfully rolled out
```

```
kubectl get pods
```
```
NAME                          READY   STATUS    RESTARTS   AGE
course-app-64cff7b856-4tvsz   1/1     Running   0          5m14s
course-app-64cff7b856-lqvnc   1/1     Running   0          5m14s
course-app-64cff7b856-nvrjn   1/1     Running   0          89s
```
