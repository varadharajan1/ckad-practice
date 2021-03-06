20% - Pod design 
* Understand how to use labels, selectors and Annotations
* Understand deployments and how to perform rolling updates
* understand deployments and how to perform rollbacks 
* understand jobs and cronjobs
#### Create a nginx deployment with 2 relicas and expose port 80 
```bash
k run mydeploy --image=nginx --serviceaccount=ckad --replicas=2 --port=80
k get deploy 
```
#### Get rollout history of mydeploy 
```bash 
k rollout history deploy/mydeploy
```
#### get rollout status of mydeploy
```bash 
k rollout status deploy/mydeploy 
```
#### change the image of mydeploy to 1.19.1 (bad image)
```bash
k set image deploy mydeploy mydeploy=nginx:1.19.1 
k get po -w
k rollout history deploy mydeploy 
```
#### change the image of mydeploy to 1.17.1 (bad image)
```bash
k set image deploy mydeploy mydeploy=nginx:1.17.1 
k get po -w
k rollout history deploy mydeploy 
```
#### check the rollout history of the deployment mydeloy 
```bash
k rollout history deploy/mydeploy  
```
#### rollout the deployment to previous version 
```bash
k rollout undo deploy/mydeploy 
```
#### rollout the deployment to version=1 
```bash
k rollout undo deploy/mydeploy --to-revision=1
```
#### change the service account of a deployment 
```bash
k set serviceaccount deploy mydeploy ckad  
```
#### autoscale deployment with min pods 3 and max 5 targetting CPU utilization 75%
```bash
k autoscale deploy mydeploy --min=3 --max=5 --cpu-percent=75
```
#### pause the deployment 
```bash
k rollout pause deploy mydeploy 
```
#### update the deploy image when paused 
```bash
k set image deploy mydeploy mydeploy=nginx:1.7.1
k rollout history deploy mydeploy # there wont be any new revision as rollout is paused
```
#### resume the rollout and see the rollout history
```bash
k rollout resume deploy mydeploy
k rollout history deploy mydeploy
```
#### change maxSurge and minSurge on the deployment 
```
k edit deploy mydeploy 
```
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "6"
  creationTimestamp: "2020-01-31T05:25:05Z"
  generation: 7
  labels:
    run: mydeploy
  name: mydeploy
  namespace: default
  resourceVersion: "2783"
  selfLink: /apis/extensions/v1beta1/namespaces/default/deployments/mydeploy
  uid: 09fb9568-43ea-11ea-8f17-0242ac11001a
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      run: mydeploy
  strategy:
    rollingUpdate:
      maxSurge: 20% # change maxSurge here to 20%
      maxUnavailable: 20% # and max unavailable to 20%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: mydeploy
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: mydeploy
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: ckad
      serviceAccountName: ckad
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: "2020-01-31T05:32:05Z"
    lastUpdateTime: "2020-01-31T05:32:05Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2020-01-31T05:25:05Z"
    lastUpdateTime: "2020-01-31T05:32:05Z"
    message: ReplicaSet "mydeploy-7c8ddcb787" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 7
  readyReplicas: 2
  replicas: 2
  updatedReplicas: 2
```
