# CHASE-CI-Tutorial
CHASE-CI stands for Cognitive Hardware and Software Ecosystem Community Infrastructure. It's now maintained by UCSD and provides computing resources for researchers. This is really a good news for some computational heavy work. So this tutorial is designed to help beginners to learn how to use it via a simple curriculum. I hope these 'lessons' could help you to have a basic understanding about how everything works and be able to handle most common computational tasks. Feel free to open issues if you have any questions or suggestions about this repo. Enjoy!


# Preparation
Before we really start our lessons, you need to have kubectl tool installed and get access to CHASE-CI clusters. CHASE-CI uses kunernetes to manage and allocate computing resources. Kubectl is the command-line tool which allows you to run commands against Kubernetes clusters. I think it's useful to have a basic understanding about some concepts in kubernetes such as pods and containers. But I don't think it's worthwhile to looking into kubernetes documentation since we're not creating a computing cluster by ourselves. So my suggestion is google the words related to kubernetes when you meet them. I believe you will have a better understanding about everything after going through this tutorial. For now, you just need to know that we use kubectl to run commands against Kubernets clusters which is actually CHASE-CI in our case.

1. Install kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl/. To know more about it, the documentation is available [here](https://kubernetes.io/docs/reference/kubectl/overview/).
2. Gain access to CHASE-CI cluster. Go to http://nautilus.optiputer.net
   - Click on login (upper right corner)
   - Choose UCI (or your institution) from the list and use your credentials. Now you should have user access.
   - Click the "Get Config" link on top right and get your config file.
   - Put the file in your $home/.kube directory.
   - (option 1) Register at https://rocket.nautilus.optiputer.net/ chat app and request the owners there to give you admin access to create a namespace. Recommended as the owners are active on the chat and help you fast, even if you don't need admin access.
   - (option 2) If someone from your lab already has admin access, ask to add as a user to his/her namespaces(s).

# Curriculum
Great! Let's start our simple lessons！ Let's first clone this repo to your local computer since some scripts in this repo are necessary 'course materials'. 

```
git clone https://github.com/KarlXing/CHASE-CI-Tutorial.git  ~/CHASE-CI-Tutorial
cd ~/CHASE-CI-Tutorial
```

## Lesson1: Create, Use and Delete Pods
Let's first create a pod, run 
```
kubectl create -f scripts/l1-pod.yaml
```
You just submitted the request to create a pod which is a computing unit in kubernetes. To give a more intuitive understanding, you can think of a pod as a remote interactive system with a certain amount of computing resources. After submitting the request, you could run
```
kubectl get pods -n carl-uci
```
This command will list all current pods under namespace of carl-uci. Please replace *carl-uci* with your own *namespace*. Then you should be able to see something like below. 

![Pods](imgs/l1-1.png)


We just created a pod named 'aha-pod'. The STATUS is *ContainerCreating* now. It normally take a short time for the creation of the pod. Once it's created, the STATUS will become *Running*. Once the pod is ready, you could run 
```
kubectl exec -it aha-pod /bin/bash
```
This command allows you to get a shell of the running. In the shell, you could run commands that you normally run in terminal in your local machine. For example, in our case, let's give it a simple python task .
```
git clone https://github.com/KarlXing/CHASE-CI-Tutorial.git  ~/CHASE-CI-Tutorial
cd ~/CHASE-CI-Tutorial
python ~/scripts/l1.py
```

So our simple task wrote a file called *l1_output.txt*. We might want to save the output result to our local computer. So run the command below in your local computer  
```
kubectl cp carl-uci/aha-pod:/home/user/CHASE-CI-Tutorial/l1_output.txt ~/Desktop/ 
```
It just copied the output file from the pod to the desktop directory in your local machine.   
Okay, our task is done and the result is saved to our local machine. Then we need to delete it to release the computing resources. Please always delete the pod once it's not needed anymore so that the computing resources could be recollected for other users to use. To delete the pod, run

```
kubectl delete pod aha-pod
```

Great! We just learned how to use several most common and useful kubectl commands to do our task. It's the first step, but also a big one. Wait a minute, our first lesson is not done yet. We need to look into the yaml file of the pod so that we understand how it was created. So our yaml file looks like this: 
```
apiVersion: v1
kind: Pod
metadata:
  name: aha-pod
spec:
  containers:
  - name: container-name
    image: jinwei/imagehome:latest
    command: ["/bin/sh"]
    args: ["sleep", "36500000"]
    imagePullPolicy: Always
    resources:
      limits:
        nvidia.com/gpu: 1
        memory: 2Gi
        cpu: 2
      requests:
        nvidia.com/gpu: 1
        memory: 2Gi
        cpu: 1
  restartPolicy: Never
```
I will only explain several key configurations in this yaml file since I don't make things too complicated for beginners. **kind** specifies the type of instance we requested CHASE-CI to create. Here, it was a pod. We would see other **kind** in later lessons. The **name** under metadata stands for the pod name. **name** under containers is the name for a container. Pod is the smallest computing unit for kubernetes and a pod could have multiple containers. For pod and container, one analogy that I can come up with is NSF funding application. So NSF committee is kubernetes and pods are research teams that ask the committee for money (computing resouces). The committee check the application based on the proposal of a team so team is a basic unit in funding application. A team (pod) could have one or more team members (containers). This metaphor could be inaccurate since I have limited NSF experience :) Correct me if I'm wrong. Another import configureation is **image**. So *jinwei/imagehome:latest* means kubernetes would pull the image from https://hub.docker.com/repository/docker/jinwei/imagehome to create the container. You might want to read an introduction article about docker if you're not familiar with that. 

**command** and **args** define the command and its arguments to run once the pod was created. In our example, we run a sleep command in a shell. We will see more usages of them later.

The last thing is **resources**. It's very self-explained. **requests** means how much resoruces that you request. Those resources will allocated to you only and would not be shared with others. If the resources that you used goes beyond your request and kubernetes do have some idle resources, it may give you more resources dynamically, but will never goes beyond the **limit**. 

So our first lesson is done! I hope you enjoy this :)

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


