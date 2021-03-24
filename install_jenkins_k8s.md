# 쿠버네티스 환경에서 젠킨스 설치
### 전제조건: 쿠버네티스 환경(로컬이기 때문에 minikube) 설치 필요

구성하고 싶은 구조
![k8s 위의 젠킨스 구조](https://user-images.githubusercontent.com/33820372/112235082-46b6ff80-8c81-11eb-8be4-0a1806448085.png)

Jenkins master는 docker이미지로 설치하고,
Jenkins slave는 구성 후에 jnlp로 통신한다.

### jenkins master 설치
jenkins namespace 생성
``` bash
$ kubectl create namespace jenkins
namespace/jenkins created
```

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
