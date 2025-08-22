pod scaling in Kubernetes :
============================

---->. two types of scalling

        1. vertical scalling
        2. horizontal scalling


1 . Vertical scaling
------------------------------------

Vertical scaling in Kubernetes means increasing or decreasing the CPU and memory resources assigned to a single pod, instead of adding or removing pods. It makes a pod more powerful by giving it more resources.

Horizontal Pod Autoscaler :
---------------------------------------------------

    Horizontal Pod Autoscaler --> k8s object

 >> Horizontal scaling means increasing or decreasing the number of pod replicas to handle changes in application load.



--> increase/dec the pods manually is called a manual scaling[replicas 2 to 3, 3 to 4]

    HorizontalPodAutoscaler --> k8s object



  IQ :  what is the diff b/w pod auto scaling and aws auto scaling? 

    | Feature             | **Horizontal Scaling**                        | **Vertical Scaling**                                  |
| ------------------- | --------------------------------------------- | ----------------------------------------------------- |
| **What it does**    | Adds or removes **pod replicas**              | Increases or decreases **CPU/memory of a single pod** |
| **Use case**        | Handle **more traffic** by spreading the load | Make a pod handle **heavier tasks** on its own        |
| **Pod count**       | Changes (more or fewer pods)                  | Stays the same (only 1 pod, just stronger)            |
| **Scaling method**  | Replication (e.g. `replicas: 3`)              | Resource adjustment (e.g. `cpu: 500m → 1000m`)        |
| **Example**         | Going from 2 to 5 pods                        | Giving 1 pod more RAM or CPU                          |
| **Restart needed?** | Not usually                                   | Often restarts the pod                                |
| **Autoscaler used** | **Horizontal Pod Autoscaler (HPA)**           | **Vertical Pod Autoscaler (VPA)**                     |









    --> two types of scalings?

     1. vertical
     2. Horizontal

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80




kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml



metric server set-up
====================

--> search metric server yaml for hpa

--> open the GitHub link

--> kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

--> kubectl get all -n kube-system

--> But pod is not coming up due to certificate issue.

--> kubectl edit deploy metrics-server -n kube-system

--> 

spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls  ---> add this line


=========================================================================

#deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpadeployment
  labels:
    name: hpadeployment
spec:
  replicas: 1
  selector:
    matchLabels:
      name: hpapod
  template:
    metadata:
      labels:
        name: hpapod
    spec:
      containers:
      - name: hpacontainer
        image: k8s.gcr.io/hpa-example
        ports:
        - name: http
          containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "64Mi"
          limits:
            cpu: "100m"
            memory: "128Mi"
---
#HPA

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpadeploymentautoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpadeployment
  minReplicas: 2
  maxReplicas: 4
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 30
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80

---

#service

apiVersion: v1
kind: Service
metadata:
  name: hpaclusterservice
  labels:
    name: hpaservice
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    name: hpapod
  type: ClusterIP


========================================================


# ==== Execute below commands to increase load====

Now, we will see how the auto scaler reacts to increased load. We will start a container, and send an infinite loop of queries to the php-apache service .
# Create A Temp Pod in interactive mod to access app using service name

  $ kubectl run -i --tty load-generator --rm --image=busybox /bin/sh	
# Execute below command in Temp Pod
 
  $ while true;
   do wget -q -O- http://hpaclusterservice;
   done	

Open kubectl terminal in another tab and watch kubectl get pods or kubect get hpa to see how the auto scaler reacts to increased load.

  $ watch kubectl get hpa


