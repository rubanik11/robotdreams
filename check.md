# check

прибираємо старі ресурси з минулого ДЗ

```
kubectl delete deployment course-app --ignore-not-found
kubectl delete service course-app --ignore-not-found
kubectl delete configmap course-app-config --ignore-not-found
```
```
deployment.apps "course-app" deleted from default namespace
service "course-app" deleted from default namespace
```

деплоїмо ConfigMap, Deployment з 10 реплік і Service

```
kubectl apply -f configmap.yaml
```
```
configmap/course-app-config created
```

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

перевіряємо що всі 10 реплік піднялись

```
kubectl get deploy
```
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
course-app   10/10   10           10          58s
```

```
kubectl get pods
```
```
NAME                         READY   STATUS    RESTARTS   AGE
course-app-6c966d887-5vjc7   1/1     Running   0          58s
course-app-6c966d887-cptpb   1/1     Running   0          58s
course-app-6c966d887-crrr6   1/1     Running   0          58s
course-app-6c966d887-flmq2   1/1     Running   0          58s
course-app-6c966d887-h27hd   1/1     Running   0          58s
course-app-6c966d887-kvxj5   1/1     Running   0          58s
course-app-6c966d887-p497w   1/1     Running   0          58s
course-app-6c966d887-qzf47   1/1     Running   0          58s
course-app-6c966d887-t6g6p   1/1     Running   0          58s
course-app-6c966d887-whtfg   1/1     Running   0          58s
```

міняємо значення в ConfigMap (APP_GREETING на v2) і дивимось що з подами — старі поди мають лишитись з v1

```
kubectl edit configmap course-app-config
```
```
configmap/course-app-config edited
```

```
kubectl get pods
```
```
NAME                         READY   STATUS    RESTARTS   AGE
course-app-6c966d887-5vjc7   1/1     Running   0          29m
course-app-6c966d887-cptpb   1/1     Running   0          29m
course-app-6c966d887-crrr6   1/1     Running   0          29m
course-app-6c966d887-flmq2   1/1     Running   0          29m
course-app-6c966d887-h27hd   1/1     Running   0          29m
course-app-6c966d887-kvxj5   1/1     Running   0          29m
course-app-6c966d887-p497w   1/1     Running   0          29m
course-app-6c966d887-qzf47   1/1     Running   0          29m
course-app-6c966d887-t6g6p   1/1     Running   0          29m
course-app-6c966d887-whtfg   1/1     Running   0          29m
```

```
kubectl exec course-app-6c966d887-5vjc7 -- printenv | grep APP_GREETING
```
```
APP_GREETING=hello-from-configmap-v1
```

рестартимо deployment щоб поди підхопили нові значення

```
kubectl rollout restart deployment/course-app
```
```
deployment.apps/course-app restarted
```

```
kubectl rollout status deployment/course-app
```
```
Waiting for deployment "course-app" rollout to finish: 4 out of 10 new replicas have been updated...
... (поступово 5, 6, 7, 8, 9 з 10)
Waiting for deployment "course-app" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "course-app" rollout to finish: 9 of 10 updated replicas are available...
deployment "course-app" successfully rolled out
```

```
kubectl exec course-app-6546f694fc-7x2x6 -- printenv | grep APP_GREETING
```
```
APP_GREETING=hello-from-configmap-v2
```

```
kubectl get pods
```
```
NAME                          READY   STATUS    RESTARTS   AGE
course-app-6546f694fc-7x2x6   1/1     Running   0          114s
course-app-6546f694fc-9dkc7   1/1     Running   0          2m6s
course-app-6546f694fc-fpt4f   1/1     Running   0          102s
course-app-6546f694fc-g297h   1/1     Running   0          2m18s
course-app-6546f694fc-hs5dt   1/1     Running   0          2m6s
course-app-6546f694fc-kcc8d   1/1     Running   0          97s
course-app-6546f694fc-krktw   1/1     Running   0          90s
course-app-6546f694fc-m9nrk   1/1     Running   0          2m18s
course-app-6546f694fc-qgcdr   1/1     Running   0          104s
course-app-6546f694fc-xwrgh   1/1     Running   0          115s
```

оновлюємо образ контейнера — пушимо :v2 у Docker Hub і застосовуємо

```
docker tag course-app:latest rubanik11/course-app:v2
```
```
```

```
docker push rubanik11/course-app:v2
```
```
The push refers to repository [docker.io/rubanik11/course-app]
... (всі шари — Layer already exists)
v2: digest: sha256:9373f872080a74914339610fbfcda4f15a4f4bce1de5d394e7b8b59c4f399a98 size: 856
```

```
kubectl set image deployment/course-app course-app=rubanik11/course-app:v2
```
```
deployment.apps/course-app image updated
```

```
kubectl rollout status deployment/course-app
```
```
Waiting for deployment "course-app" rollout to finish: 4 out of 10 new replicas have been updated...
... (поступово 5, 6, 7, 8, 9 з 10)
Waiting for deployment "course-app" rollout to finish: 1 old replicas are pending termination...
deployment "course-app" successfully rolled out
```

```
kubectl get pods
```
```
NAME                          READY   STATUS    RESTARTS   AGE
course-app-544fd68cbb-444rl   1/1     Running   0          72s
course-app-544fd68cbb-47n89   1/1     Running   0          70s
course-app-544fd68cbb-54tv4   1/1     Running   0          46s
course-app-544fd68cbb-75hzx   1/1     Running   0          80s
course-app-544fd68cbb-cm2hf   1/1     Running   0          83s
course-app-544fd68cbb-l5clz   1/1     Running   0          93s
course-app-544fd68cbb-mhhzk   1/1     Running   0          93s
course-app-544fd68cbb-n87tf   1/1     Running   0          50s
course-app-544fd68cbb-qmbk2   1/1     Running   0          60s
course-app-544fd68cbb-sn8cn   1/1     Running   0          57s
```

тестуємо швидкий RollingUpdate з maxSurge=3, maxUnavailable=0 (нульовий простій, новий rollout помітно швидший)

```
kubectl patch deployment course-app -p '{"spec":{"strategy":{"rollingUpdate":{"maxSurge":3,"maxUnavailable":0}}}}'
```
```
deployment.apps/course-app patched
```

```
kubectl set image deployment/course-app course-app=rubanik11/course-app:latest
```
```
deployment.apps/course-app image updated
```

```
kubectl rollout status deployment/course-app
```
```
Waiting for deployment "course-app" rollout to finish: 1 old replicas are pending termination...
deployment "course-app" successfully rolled out
```

```
kubectl get pods
```
```
NAME                          READY   STATUS    RESTARTS   AGE
course-app-6546f694fc-2lqvl   1/1     Running   0          59s
course-app-6546f694fc-2m2wf   1/1     Running   0          49s
course-app-6546f694fc-67fxp   1/1     Running   0          34s
course-app-6546f694fc-jzm98   1/1     Running   0          59s
course-app-6546f694fc-kp8r8   1/1     Running   0          46s
course-app-6546f694fc-lcsr7   1/1     Running   0          29s
course-app-6546f694fc-p9t2k   1/1     Running   0          41s
course-app-6546f694fc-qgkwt   1/1     Running   0          46s
course-app-6546f694fc-r4sps   1/1     Running   0          32s
course-app-6546f694fc-rz9ld   1/1     Running   0          59s
```

змінюємо стратегію на Recreate і знов оновлюємо образ — всі старі поди гасяться одночасно, потім піднімаються нові

```
kubectl patch deployment course-app -p '{"spec":{"strategy":{"type":"Recreate","rollingUpdate":null}}}'
```
```
deployment.apps/course-app patched
```

```
kubectl set image deployment/course-app course-app=rubanik11/course-app:v2
```
```
deployment.apps/course-app image updated
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
course-app-544fd68cbb-94kr5   1/1     Running   0          115s
course-app-544fd68cbb-9v8bl   1/1     Running   0          115s
course-app-544fd68cbb-bhpgl   1/1     Running   0          115s
course-app-544fd68cbb-df8wx   1/1     Running   0          115s
course-app-544fd68cbb-g92g8   1/1     Running   0          115s
course-app-544fd68cbb-h6bh7   1/1     Running   0          115s
course-app-544fd68cbb-hjqnm   1/1     Running   0          115s
course-app-544fd68cbb-hzdgf   1/1     Running   0          115s
course-app-544fd68cbb-jlj26   1/1     Running   0          115s
course-app-544fd68cbb-r5hvc   1/1     Running   0          115s
```

всі поди мають однаковий AGE (115s) — це характерна ознака Recreate: одночасний старт. У rolling AGE був неоднаковий (29s, 32s, 34s, 41s ...).

прибираємо

```
kubectl delete -f .
```
```
configmap "course-app-config" deleted from default namespace
deployment.apps "course-app" deleted from default namespace
service "course-app" deleted from default namespace
```
