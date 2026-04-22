# check

```
docker compose -f docker-compose.yml up -d --build
```
```
[+] Running 5/5
 ✔ course-repo-course-app              Built
 ✔ Network course-repo_default         Created
 ✔ Volume course-repo_redis_data       Created
 ✔ Container course-repo-redis-1       Healthy
 ✔ Container course-repo-course-app-1  Started
```

```
docker compose -f docker-compose.yml ps
```
```
NAME                       IMAGE                    SERVICE      STATUS                    PORTS
course-repo-course-app-1   course-repo-course-app   course-app   Up                        0.0.0.0:8080->8080/tcp
course-repo-redis-1        redis:alpine             redis        Up (healthy)              6379/tcp
```

```
curl -i http://localhost:8080
```
```
HTTP/1.1 200 OK
server: uvicorn
content-type: text/html; charset=utf-8

<html>... Course App UI ...</html>
```

```
curl -i http://localhost:8080/healthz
```
```
HTTP/1.1 200 OK
server: uvicorn
content-type: application/json

{"status":"ok"}
```

```
docker compose -f docker-compose.yml exec course-app env | grep APP_
```
```
APP_REDIS_URL=redis://redis:6379/0
APP_STORE=redis
```

```
docker compose -f docker-compose.yml down
```
