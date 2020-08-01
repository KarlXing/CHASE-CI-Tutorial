## Lesson2: Use Persistent Volumes
So the last lesson is only about computation. What if we have a big dataset in our local computer which is needed for the task? We definitely don't want to use ```kubectl cp ``` command to copy the data from local computer to the pod everytime. So the solution here is creating persistent volumes and mounting the volume to the pod when we create it. For more information about persistent volumes, you could read https://ucsd-prp.gitlab.io/userdocs/storage/ceph-posix/. 

The first step to use the persistent volume is creating one. Let's run
```
cd ~/CHASE-CI-Tutorial
kubectl create -f scripts/l2_pvc.yaml
```
So just submitted the request to create a persistent volume. To check the persistent volumes under our namespace, run 
```
kubectl get pvc -n carl-uci
# remember to replace calr-uci with your own namespace
```
You should see something like below. 
![pvc](imgs/l2-1.png)

So, let's look into the yaml file to see what we have done. 
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: aha-pvc
spec:
  storageClassName: rook-cephfs
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```
In this yaml file, **kind** is *PersistentVolumeClaim* which is self-explained. The **name** of the persistent volume is *aha-pvc*. **StorageClassName** here is *rook-cephfs*. As for the different storage classes, you could refer https://ucsd-prp.gitlab.io/userdocs/storage/ceph-posix/ to see their differences. For normal users, I think it's fine to use *rook-cephfs* or switch to *rook-cephfs-east* if you are now in the east of US. **accessModes** means whether a volume could be mounted to multiple pods at the same time. Here *ReadWriteMany* means YES while *ReadWriteOnce* means No. If you submit a request to mount a volume that supports *ReadWriteOnce* to two pods, then only one pod will be ready and the STATUS of the other pod will always be **Pending**. Finally, **storage** specifies the size of the volume. 

So let's create a pod with *aha-pvc* mounted! Run 
```
kubectl create -f l2_pod.yaml
```
If you check the yaml file, you could two extra parts besides the yaml file from lesson1. 
```
  volumes:
    - name: pv
      persistentVolumeClaim:
        claimName: aha-pvc
```
The first one is **volumes**. It means you tell kubernetes that you want to use *aha-pvc* which is a persistent volume. You also give it a name as *pv*. In container configuration, you could see
```
    volumeMounts:
      - name: pv
        mountPath: /pv
```
It means you mount persistent volume *pv* which is actually *aha-pvc* to the path */pv*. So once the we just requested is ready, run 
```
kubectl exec -it aha-pod-l2 /bin/bash
```
After getting the shell, let's run 
```
cd /pv
mkdir hello
```
So we just created a directory called hello in the persistent volume. If you delete the current pod (```kubectl delete pod aha-pod-l2```) and create it (```kubectl create -f l2_pod.yaml```) again, you would still see this directory under */pv* after getting the shell(```kubectl exec -it aha-pod-l2 /bin/bash```) . Have a try! 