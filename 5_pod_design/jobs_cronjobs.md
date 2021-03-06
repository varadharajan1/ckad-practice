20% - Pod design 
* Understand how to use labels, selectors and Annotations
* Understand deployments and how to perform rolling updates
* understand deployments and how to perform rollbacks 
* understand jobs and cronjobs
#### create a job that runs the command uname 
```bash
k create job busyb --image=busybox  -- uname
k log <podname>
```
####  Cretae a cronjob that runs the command uname
####  quickly refer to this link and play around -  [crontab tool](https://crontab.guru/)
```bash
k create cronjob busyb --image=busybox --schedule="*/1 * * * *" -- uname
k log po -w  # pods will be created every one min 
```
#### Create a job that should terminate the pod created by job in 30 seconds if it doesn't execute in that timeframe
```
k create job busyb --image=busybox --dry-run -o yaml -- /bin/sh -c 'uname; sleep 3600' > job.yaml
k explain job.spec # this should give you all the fields under the spec job.spec.activeDeadlineSeconds is what you should be using
nano job.yaml

```
```
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busyb
  name: busyb
spec:
  activeDeadlineSeconds: 30 # If it pod doesn't execute in 30 seconds terminate it
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busyb
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - uname; sleep 3600
        image: busybox
        name: busyb
        resources: {}
      restartPolicy: OnFailure
status: {}
```
```bash
k get po -w # pod should be terminated after sometime 
k describe job busyb # look at the events section  
k delete job busyb
```
#### Create a job that runs 5 times and runs the command uname 
```
k create job busyb --image=busybox --dry-run -o yaml  -- /bin/sh -c 'uname; sleep 3600' > job.yaml
k explain job.spec # this should give you all the fields under the spec job.spec.completions is what you should be using
nano job.yaml

```
```
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busyb
  name: busyb
spec:
  activeDeadlineSeconds: 30 # If it pod doesn't execute in 30 seconds terminate it
  completions: 5
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busyb
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - uname
        image: busybox
        name: busyb
        resources: {}
      restartPolicy: OnFailure
status: {}
```
```bash
k get po -w # 5 pods are created  
k get job busyb # look at no of completions  
k delete job busyb
```
#### run the above job in parallel using 3 threads and does completion 9 times 
```
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busyb
  name: busyb
spec:
  activeDeadlineSeconds: 30 # If it pod doesn't execute in 30 seconds terminate it
  completions: 9
  parallelism: 3
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busyb
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - uname
        image: busybox
        name: busyb
        resources: {}
      restartPolicy: OnFailure
status: {}
```
```bash
k get po -w # 9 pods are created (will notice running parallel ) 
k get job busyb # look at no of completions  
k delete job busyb
```
