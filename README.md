# CHASE-CI-Tutorial
CHASE-CI stands for Cognitive Hardware and Software Ecosystem Community Infrastructure. It's now maintained by UCSD and provides computing resources for researchers. This is really a good news for some computational heavy work. So this tutorial is designed to help beginners to learn how to use it via a simple curriculum. I hope these 'lessons' could help you to have a basic understanding about how everything works and be able to handle most common computational tasks. Feel free to open issues if you have any questions or suggestions about this repo. If you think this repo is helpful, please consider to give it a star. Thanks!


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
I'd like to divide the tutorial into intermediate levels and advanced levels. After going through the intermediate ones, you should have an understanding about how CHASE-CI works and be able to do most of your work. Advanced ones could help you to do the work more efficiently and could also be necessary for some certain tasks. 

* Lesson1-Intermediate: Create, Use and Delete Pods
* Lesson2-Intermediate: Use Persistent Volumes
* Lesson3-Intermediate: Use Jobs
* Lesson4-Advanced: Big Data Storage (coming soon)
* Lesson5-Adanced: Customized Docker Images (coming soon)


Great! Before we start, let's first clone this repo to your local computer since some scripts in this repo are necessary 'course materials'. 

```
git clone https://github.com/KarlXing/CHASE-CI-Tutorial.git  ~/CHASE-CI-Tutorial
cd ~/CHASE-CI-Tutorial
```

Now you could go through the lessons in this repo! 




