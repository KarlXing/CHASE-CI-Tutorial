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