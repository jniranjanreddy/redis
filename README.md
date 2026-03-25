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
