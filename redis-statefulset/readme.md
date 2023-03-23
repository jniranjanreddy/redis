## How to deploy redis in K8s as statefulset
## Source: https://www.containiq.com/post/deploy-redis-cluster-on-kubernetes
```
kubectl create ns redis
kubectl get ns

## StorageClass
cat kubectl sc.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete

kubectl apply -f sc.yaml
kubectl get sc
```
## Create storage class for persistant storage, creating 3 volumes.
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv1
spec:
  storageClassName: local-storage
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/storage/data1"
----------------------------------  
    
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv2
spec:
  storageClassName: local-storage
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/storage/data2"
----------------------------------------
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv3
spec:
  storageClassName: local-storage
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/storage/data3"

# Apply pv.yaml
kubectl apply -f pv.yaml
kubectl get pv
```
## Creating ConfigMap 
```
ConfigMap file can be found under parent Directory.
kubectl apply -n redis -f redis-config.yaml
kubectl get configmap -n redis
```
## Create StateFulSet
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: redis
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      initContainers:
      - name: config
        image: redis:6.2.3-alpine
        command: [ "sh", "-c" ]
        args:
          - |
            cp /tmp/redis/redis.conf /etc/redis/redis.conf
            
            echo "finding master..."
            MASTER_FDQN=`hostname  -f | sed -e 's/redis-[0-9]\./redis-0./'`
            if [ "$(redis-cli -h sentinel -p 5000 ping)" != "PONG" ]; then
              echo "master not found, defaulting to redis-0"

              if [ "$(hostname)" == "redis-0" ]; then
                echo "this is redis-0, not updating config..."
              else
                echo "updating redis.conf..."
                echo "slaveof $MASTER_FDQN 6379" >> /etc/redis/redis.conf
              fi
            else
              echo "sentinel found, finding master"
              MASTER="$(redis-cli -h sentinel -p 5000 sentinel get-master-addr-by-name mymaster | grep -E '(^redis-\d{1,})|([0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3})')"
              echo "master found : $MASTER, updating redis.conf"
              echo "slaveof $MASTER 6379" >> /etc/redis/redis.conf
            fi
        volumeMounts:
        - name: redis-config
          mountPath: /etc/redis/
        - name: config
          mountPath: /tmp/redis/
      containers:
      - name: redis
        image: redis:6.2.3-alpine
        command: ["redis-server"]
        args: ["/etc/redis/redis.conf"]
        ports:
        - containerPort: 6379
          name: redis
        volumeMounts:
        - name: data
          mountPath: /data
        - name: redis-config
          mountPath: /etc/redis/
      volumes:
      - name: redis-config
        emptyDir: {}
      - name: config
        configMap:
          name: redis-config
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "local-storage"
      resources:
        requests:
          storage: 500Mi

kubectl apply -n redis -f redis-statefulset.yaml
kubectl get pods -n redis
```
## Create service
```
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  clusterIP: None
  ports:
  - port: 6379
    targetPort: 6379
    name: redis
  selector:
    app: redis

kubectl apply -n redis -f redis-service.yaml
kubectl get service -n redis

kubectl -n redis logs redis-0
kubectl -n redis describe pod redis-0

```
## Check Replication.
```
Get the replication information using the following command:


kubectl -n redis exec -it redis-0 -- sh
redis-cli 
auth a-very-complex-password-here
info replication

This screenshot shows how many slaves are connected with the master, as well as the slave’s IP address and other information.

Likewise, you can check the slave pod’s log and see the successful connection between master and slave:
```
![image](https://user-images.githubusercontent.com/83489863/227305662-70a6cec8-5dec-45fb-adc5-6090a0586930.png)

