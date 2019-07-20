---
title: Kubernetes Started
categories:  Kubernetes
tags: 
- Kubernetes
---

###  1. 为pod/container配置env参数，用 DEMO_GREETING 命名参数；
* env 的名称要大写，否则 printenv 打印不出来，且不报错

```
apiVersion: v1
kind: Pod 
metadata:
  name: demo-green
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: nginx
    env:
    - name: DEMO_GREETING
      value: 'hello from the environment'

kubectl exec -it demo3 -- sh
printenv | grep DEMO_GREETING
```

###  2. 为pod/container配置env参数，使用 configMap 为 config1；
* 使用了 congfig 的所有参数则不需要为 env 在设置名称；

```
apiVersion: v1
kind: Pod
metadata:
  name: demo-green
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: nginx
    envFrom:
      configMapRef:
      - name: config1
```
###  3. 为pod/container配置env参数，使用 configMap 为 config1 的 key1;
* 使用 configMap 种的某项参数时，需要为 env 设置名称；

```
apiVersion: v1
kind: Pod
metadata:
  name: demo-green
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: nginx
    envFrom:
    - name: env1
      configMapKeyRef:
      - name: config1
        key: key1
```
### 4. 使用环境变量来定义参数
```
apiVersion: v1
kind: Pod
metadata:
  name: demo-green
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: nginx
    env:
    - name: MESSAGE
      value: 'hello world'
    command: ['/bin/echo']
    args: ['$(MESSAGE)']
```

###  5. 为container配置args/command参数；
```
apiVersion: v1
kind: Pod
metadata:
  name: demo-green
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: nginx
    args: ['HOSTNAME']
    commands: ['printenv']
```

### 6. 将 pod 分配给 node
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeSelector:
    nodeKey: nodeValue
  containers:
  - name: envar-demo-container
    image: nginx1
```

###  7. 记录基础日志
* 如果容器崩溃，可以使用 kubectl logs --previous

```
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args: ['/bin/sh', '-c', 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']

kubectl logs counter
```

###  8. 使用 sidecar 记录日志
```
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  volumes:
  - name: my-volume
    emptyDir: {}
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: my-volume
      mountPath: /var/log
  - name: count-agent1
    image: busybox
    args: ['/bin/sh', '-c', 'tail -n+1 -f /var/log/1.log']
    volumeMounts:
    - name: my-volume
      mountPath: /var/log   
  - name: count-agent2
    image: busybox
    args: ['/bin/sh', '-c', 'tail -n+1 -f /var/log/2.log']
    volumeMounts:
    - name: my-volume
      mountPath: /var/log 
```
### 9. 为 pod 设置 ServiceAccount
kubectl create sa sa1

```
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  serviceAccountName: sa1
  containers:
  - name: nginx
    image: nginx

```

### 10. 为 ServiceAccount 添加 ImagePullSecret
* 创建 ImagePullSecret

```
kubectl create secret docker-registry test2 --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL
```

* 添加到 ServiceAccount

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: test2
  namespace: default
secrets:
- name: default-token-uudge
imagePullSecrets:
- name: myregistrykey
```

* 为 pod 配置 serviceAccount

```
apiVersion: v1
kind: Pod
metadata:
  name: demo-green
  labels:
    purpose: demonstrate-envars
spec:
  serviceAccountName: test2
  containers:
  - name: envar-demo-container
    image: nginx
```

### 11. 为 deploy/rs 扩容或缩放
```
kubectl scale deploy/foo --replicas=3
kubectl scale rs/foo --replicas=3
```

### 12. 为 deploy 添加自动伸缩
```
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
```

### 13. 给 node 打污点
```
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1:NoSchedule-
```
### 14. 给 pod 添加容忍
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  tolerations:
  - key: 'key1'
    operator: "Equal"
    value: 'value1'
    effect: 'NoSchedule'
    tolerationSeconds: 3600
  containers:
    - name: myfrontend
      image: nginx
```


### 15. 为 pod 添加持久卷
* 创建文件

```
echo 'hello world' > /mnt/data
```

* 创建 pv

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  storageClassName: normal
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
```

* 创建 pvc

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  storageClassName: normal
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
```

* 为 pod 绑定 pvc

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: pvc
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - name: mypd
        mountPath: "/var/www/html"      
```

### 16. 给 pod 配置存活检查
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
    exec:
      command:
      - cat
      - /tmp/healthy
    initialDelaySeconds: 5
    periodSeconds: 5

```

### 17. 创建 secret
```
-> kubectl create secret generic my-secret --from-literal=a=b
-> echo xxx | base64 -d
```

### 18. 将 secret 通过 volume 绑定在 pod 上
```
apiVersion: v1
kind: Pod
metadata:
  name: secrets-test-pod
spec:
  volume:
  - name: secret-volume
    secret:
    - secretName: my-secret
  containers:
  - image: nginx
    name: test-container
	volumeMounts:
	- name: secret-volume
	  path: /etc/secret/volume
```

### 19. 将 secret 绑定在 env 上
```
apiVersion: v1
kind: Pod
metadata:
  name: env-single-secret
spec:
  containers:
  - name: env1
    image: nginx
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: backend-user
          key: backend-username
```

### 20. 将 secret 所有键值对配置为容器环境变量
```
apiVersion: v1
kind: Pod
metadata:
  name: envfrom-secret
spec:
  containers:
  - name: envvars-test-container
    image: nginx
    envFrom:
    - secretRef:
      name: test-secret
```

### 21. 为 pod 添加 serviceAccountName
```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: build-robot
```


### 22.拉取私有仓库的镜像
```
-> docker login
-> cat ~/.docker/config.json //查看config.json
-> kubectl create secret generic regcred \
    --from-file=.dockerconfigjson=<path/to/.docker/config.json> \
    --type=kubernetes.io/dockerconfigjson
```
```
// 为 pod 添加 imagePullSecrets
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred
```
 
### 23.为 pod 公开端口暴露给集群
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
```


### 24.创建 service

```
-> kubectl expose deployment/my-nginx
-> kubectl get service my-nginx
-> kubectl get endpoint my-nginx

// 或者 直接编辑 yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
```

### 25. 连接 service
```
-> kubectl exec my-nginx-3800858182-jr4a2 -- printenv | grep SERVICE

```

### 26. 暴露 service
* nodePort 或 loadbalance 

```
curl <node-ip>: nodePort
curl <externale-ip>
```

### 27. 有一组Pod，每个Pod都在TCP端口9376上侦听并带有标签"app=MyApp"
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
  	app: MyApp
  ports:
  - protocol: tcp
    port: 80
    targetPort: 9376
```

### 28.不带 selector 的 service,手动将 service 映射到运行它的网络地址和端口
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```
```
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.0.42
    ports:
      - port: 9376
```

### 29. 服务发现
* Kubernetes支持2种寻找服务的主要模式 - 环境变量和DNS。 
I."redis-master"公开TCP端口6379并已分配群集IP地址10.0.0.11的服务会生成以下环境变量

```
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```