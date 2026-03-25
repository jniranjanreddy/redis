# redis
## redis-cli -h 192.168.9.51 -p 6379 ping
PONG

# sudo /opt/redislabs/bin/redis-cli -p 12000
```
127.0.0.1:16653> set key1 123
OK
127.0.0.1:16653> get key1
"123"

root@dev /myworkspace/nest/project-name # /var/lib/docker/overlay2/45ffef0c87e5b424cdac7b820430a8b5677d30c75e2240e5ebe1e63cc8ae34fd/diff/usr/local/bin/redis-cli -p 8000
127.0.0.1:8000>
127.0.0.1:8000> set key1 123
OK
127.0.0.1:8000> get key1
"123"
127.0.0.1:8000>

```
```

kubectl create secret generic redis-auth -n backend --from-literal=password='abc123'



kubectl patch deployment redis-deployment -n backend --type=strategic -p "$(cat <<'EOF'
spec:
  template:
    spec:
      containers:
      - name: redis
        command:
        - redis-server
        args:
        - --appendonly
        - "yes"
        - --requirepass
        - $(REDIS_PASSWORD)
        env:
        - name: REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: redis-auth
              key: password
        - name: REDISCLI_AUTH
          valueFrom:
            secretKeyRef:
              name: redis-auth
              key: password
EOF
)"


kubectl exec -n backend deploy/redis-deployment -- env REDISCLI_AUTH='abc123' redis-cli ping
```
```
1. Find the Service (host name)
Redis is reached through a Service, not the Deployment name.

kubectl get svc -n backend | grep -i redis
Note the Service name (example: redis, redis-deployment, redis-service).


2. Connection string shape
In-cluster (apps in any namespace):

redis://:YOUR_PASSWORD@SERVICE_NAME.backend.svc.cluster.local:6379/0
Same namespace (backend) you can shorten:

redis://:YOUR_PASSWORD@SERVICE_NAME:6379/0
No password (if you remove auth): redis://SERVICE_NAME.backend.svc.cluster.local:6379/0
TLS: only if you terminate TLS on Redis; then often rediss://... (your setup is probably plain redis://).
3. Example
If the service is redis and password is secret:

redis://:secret@redis.backend.svc.cluster.local:6379/0
4. From your machine (not in-cluster)
Use port-forward and point clients at localhost:

kubectl port-forward -n backend svc/SERVICE_NAME 6379:6379
Then:


redis://:YOUR_PASSWORD@127.0.0.1:6379/0
```

