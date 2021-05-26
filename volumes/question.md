Create a **Persistent Volume** called `log-volume`. It should make use
of a **Storage Class** with name `manual`. It should use **RWX** as
the access mode and have a size of `1Gi`. The volume should use the 
hostPath `/opt/volume/nginx`.

Next, create a **Persistent Volume Claim** called `log-claim` requesting 
a minimum of 200Mi of storage. This PVC should bind to `log-volume`.

Mount this in a **Pod** called `logger` at the location `/var/www/nginx`. This
pod should use the image `nginx:alpine`.

Solution:
We are talking here about a Pod, that mounts a volume the indirect way, via 
pvcs. 

**How to create such a storage class?**

My guess is using one of type `local`. It doesn't need provisioner and hence it
does not support dynamic provisioning:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: manual
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```
Ohhhhh fuck. Not sure this was necessary, see storageClassName below" :')
Previous step isn't required. Manual StorageClass is somehow there for you :)

**How to bind PVC and PV?**

1. Create first the PV:
   ```yaml 
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: log-volume
     labels:
       type: local
   spec:
     storageClassName: manual
     capacity:
       storage: 1Gi
     accessModes:
       - ReadWriteMany
     hostPath:
       path: "/opt/volume/nginx"
   ```
   Note i: RWX means ReadWriteMany (accessMode)
   Note ii: In the docs they tell you to create the folder `/opt/volume/nginx`
   beforehand but I didn't do it and worked anyway.
2. Create the PVC that will be bind with our PV:
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: log-claim
   spec:
     storageClassName: manual
     accessModes:
     - ReadWriteMany
     resources:
       requests:
         storage: 200Mi
   ```
**How to create a Pod pointing to the PVC?**

Create the pod with a volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logger
spec:
  volumes:
    - name: logs
      persistentVolumeClaim:
        claimName: log-claim
  containers:
    - name: nginx
      image: nginx:alpine
      volumeMounts:
        - mountPath: "/var/www/nginx"
          name: logs
```



