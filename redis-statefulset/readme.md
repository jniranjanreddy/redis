## How to deploy redis in K8s as statefulset
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

```
