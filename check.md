# check

піднімаємо і перевіряємо healthcheck

```
docker compose -f docker-compose.yml up -d --build
```
```
[+] Running 5/5
 ✔ course-app:latest                   Built
 ✔ Network course-repo_default         Created
 ✔ Volume course-repo_appdata          Created
 ✔ Container course-repo-redis-1       Healthy
 ✔ Container course-repo-course-app-1  Started
```

```
docker compose -f docker-compose.yml ps
```
```
NAME                       IMAGE               SERVICE      STATUS                    PORTS
course-repo-course-app-1   course-app:latest   course-app   Up (healthy)              0.0.0.0:8080->8080/tcp
course-repo-redis-1        redis:alpine        redis        Up (healthy)              6379/tcp
```

лічильник до рестарту

```
curl -s http://localhost:8080/api/counter/visits
```
```
{"name":"visits","value":3}
```

рестартуємо і перевіряємо що том appdata зберіг дані

```
docker compose -f docker-compose.yml down
docker compose -f docker-compose.yml up -d
```
```
[+] Running 3/3
 ✔ Network course-repo_default         Created
 ✔ Container course-repo-redis-1       Healthy
 ✔ Container course-repo-course-app-1  Started
```

```
curl -s http://localhost:8080/api/counter/visits
```
```
{"name":"visits","value":3}
```

запускаємо swarm

```
docker swarm init
```
```
Swarm initialized: current node (x1m4u03umkf3ov7jfdu9lm23b) is now a manager.
```

```
docker compose -f docker-compose.yml build
```
```
[+] Building 1/1
 ✔ course-app:latest  Built
```

деплоїмо stack

```
docker stack deploy -c docker-compose.yml coursestack
```
```
Ignoring unsupported options: build
Creating network coursestack_default
Creating service coursestack_course-app
Creating service coursestack_redis
```

```
docker stack services coursestack
```
```
ID             NAME                     MODE         REPLICAS   IMAGE               PORTS
mj4c30vm3fgs   coursestack_course-app   replicated   1/1        course-app:latest   *:8080->8080/tcp
bpk45bphnnvy   coursestack_redis        replicated   1/1        redis:alpine
```

перша спроба course-app впала бо redis ще не був готовий, swarm рестартував її сам

```
docker stack ps coursestack
```
```
NAME                           IMAGE               DESIRED STATE   CURRENT STATE            ERROR
coursestack_course-app.1       course-app:latest   Running         Running 34 seconds ago
 \_ coursestack_course-app.1   course-app:latest   Shutdown        Failed 44 seconds ago    "task: non-zero exit (3)"
coursestack_redis.1            redis:alpine        Running         Running 40 seconds ago
```

```
curl -i http://localhost:8080/healthz
```
```
HTTP/1.1 200 OK
content-type: application/json

{"status":"ok"}
```

прибираємо

```
docker stack rm coursestack
docker swarm leave --force
```
