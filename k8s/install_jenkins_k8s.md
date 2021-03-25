# 쿠버네티스 환경에서 젠킨스 설치  
### 전제조건: 쿠버네티스 환경(로컬이기 때문에 minikube) 설치 필요  

구성하고 싶은 구조  
![k8s 위의 젠킨스 구조](https://user-images.githubusercontent.com/33820372/112235082-46b6ff80-8c81-11eb-8be4-0a1806448085.png)  

Jenkins master는 docker이미지로 설치하고,  
Jenkins slave는 구성 후에 jnlp로 통신한다.  

### jenkins master 설치  

#### namespace 생성  
jenkins namespace 생성  
``` bash
$ kubectl create namespace jenkins
namespace/jenkins created
```

#### persistent volume / persistent volume claim 설정  
jenkins-pv.yaml 설정  
persistent volume으로 컨테이너 동작을 멈춰도 사라지지 않는 볼륨 구성  
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-pv
  namespace: jenkins
spec:
  storageClassName: manual
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 5Gi
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/jenkins
```

jenkins-pv.yaml 파일 노드에 적용  
```
$ kubectl apply -f jenkins-pv.yaml --namespace=jenkins
persistentvolume/jenkins-pv created
```

jenkins-pvc.yaml  
pod에 필요한 pv를 요청하는 pvc yaml 설정 파일 생성  
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkins-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
```

jenkins-pvc.yaml 파일 노드에 적용
```
$ kubectl apply -f jenkins-pvc.yaml --namespace=jenkins
persistentvolumeclaim/jenkins-pvc created
```

#### jenkins.yaml 파일 설정 및 적용  
jenkins deployment 생성을 위한 설정 파일  
pvc도 jenkins-pvc로 설정, replica 수는 1개로 설정 
내부 ip로는 8080 포트로 접근, jnlp는 50000 포트로 접근  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        ports:
          - name: http-port
            containerPort: 8080
          - name: jnlp-port
            containerPort: 50000
        volumeMounts:
          - name: jenkins-vol
            mountPath: /var/jenkins_home
      volumes:
        - name: jenkins-vol
          persistentVolumeClaim:
            claimName: jenkins-pvc
```
해당 파일을 jenkins 네임스페이스에서 생성하면 
docker hub에서 이미지를 불러와서 pod가 생김 

#### jenkins service 설정 
Pod의 네트워크 설정 
```
apiVersion: v1
kind: Service
metadata:
  name: jenkins
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 32000
  selector:
    app: jenkins

---

apiVersion: v1
kind: Service
metadata:
  name: jenkins-jnlp
spec:
  type: ClusterIP
  ports:
    - port: 50000
      targetPort: 50000
  selector:
    app: jenkins
```
nodeip:30000번으로 접속하면 젠킨스 초기 설정 화면이 생성됨.  

---  

#### 에러: pod이 crashloopbackoff 상태로 jenkins 접속이 안됨  
```
$ kubectl get pods --all-namespaces
NAMESPACE              NAME                                        READY   STATUS             RESTARTS   AGE
jenkins                jenkins-6dc59b988b-pf9m5                    0/1     CrashLoopBackOff   19         75m
kube-system            coredns-74ff55c5b-zbvl5                     1/1     Running            6          6d3h
kube-system            etcd-minikube                               1/1     Running            5          6d3h
kube-system            kube-apiserver-minikube                     1/1     Running            6          6d3h
kube-system            kube-controller-manager-minikube            1/1     Running            6          6d3h
kube-system            kube-proxy-x29pv                            1/1     Running            5          6d3h
kube-system            kube-scheduler-minikube                     1/1     Running            5          6d3h
kube-system            storage-provisioner                         1/1     Running            15         6d3h
```

로그 찍어보기 시도하면 아래와 같이 permission 없어서 안된다고 뜸 
```
$ kubectl -n ns-jenkins logs jenkins-655f99d864-kkfk5
touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied
Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
```

위에서 말한 volume permission은 docker의 minikube volume에 대한 권한이다.
docker 설정에서 minikube volume의 mount path를 알아내서 권한을 모두 열어준다.
```
$ sudo chown -R 1000  /var/lib/docker/volumes/minikube/_data
```

다시 pod 확인해보니 실행 가능 상태(pod 설정 상태가 컨테이너가 에러로 종료되면 계속해서 다시 시작하게 하고 있음)
```
$ kubectl get all -n ns-jenkins 
NAME                           READY   STATUS    RESTARTS   AGE
pod/jenkins-655f99d864-kkfk5   1/1     Running   22         67m

NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/jenkins        NodePort    10.105.193.242   <none>        8080:32000/TCP   67m
service/jenkins-jnlp   ClusterIP   10.107.249.94    <none>        50000/TCP        67m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/jenkins   1/1     1            1           67m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/jenkins-655f99d864   1         1         1       67m
```

#### 젠킨스 초기 설정 및 플러그인 설치
![image](https://user-images.githubusercontent.com/33820372/112425464-75180600-8d79-11eb-8346-77216a6c7c20.png)
```
$ kubectl -n ns-jenkins logs jenkins-655f99d864-kkfk5
```
pod에서 컨테이너 생성할 때 초기 비밀번호가 있음

비밀번호 치고, 플러그인 설치하고 계정설정하면 초기 화면으로 접속가능
pv,pvc 설정을 했기 때문에 node 재시작해도 설정 정보 남아있음
