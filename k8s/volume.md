# Volume
* 쿠버네티스는 상태가 없는 stateless 앱 컨테이너를 사용하기 때문에  
  Pod안에 있는 데이터는 Pod을 재기동하고 나면 내부에 저장 된 데이터가 손실된다.  
* 볼륨은 쿠버네티스 Pod에 종속되는 디스크
* 컨테이너 단위가 아니라 Pod 단위이기 때문에 해당 Pod에 속한 여러 컨테이너가 공유해서 사용할 수 있음

참고 링크: [Volume 설명](https://kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes)

## 볼륨 플러그인
* 종류가 굉장히 다양함
* AWS EBS, AzureDisk, GCE 같은 클라우드 환경을 사용하는 볼륨
* pod가 돌아가는 노드의 디스크를 볼륨으로 사용

## 로컬 디스크를 볼륨으로 사용할 수 있는 플러그인
### emptyDir 
Pod가 실행되는 호스트의 디스크를 임시로 컨테이너 볼륨에 할당해서 사용  
--> 파드가 사라지면 볼륨에 있던 데이터도 함께 사라진다.  

생명주기가 Pod를 따르기 때문에 Pod가 컨테이너 재시작과 상관 없지만 Pod 재시작하면 사라짐 

### hostPath
Pod가 실행된 호스트가 파일이나 디렉토리를 파드에 마운트함  
실제 디렉터리를 마운트하기 때문에 파드가 재시작 되더라도 데이터가 남음  

### PV / PVC
* Persistent Volume과 Persistent Volume Claim
* PV: 클러스터 내 자원으로 다뤄지는 볼륨
* PVC: PV가 필요한 pod에서 얼만큼, 어떤 모드로 사용할지 등을 설정
![PV/PVC 구조 설명](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FHxFFq%2FbtqFQ9blCMw%2FWAlm9gjafidbJjF2BCG8bK%2Fimg.png)

#### PV 생성 설정 템플릿
[PV 설명 링크](https://kubernetes.io/docs/concepts/storage/persistent-volumes)
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-sample
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  storageClassName: manual
  persistentVolumeReclaimPolicy: Delete
  hostPath:
    path: /tmp/k8s-pv
```

#### PVC 생성 설정 템플릿
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-sample
spec:
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
```
PVC의 Storage는 PV보다 작게 설정해야 Pending 상태가 되지 않는다.

#### Pod에서 PVC를 Volume으로 사용하기
Deployment 생성 파일에서 volume을 pvc로 설정해준다. 
<pre>
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pvc-deploy-app
  labels:
    app: pvc-deploy-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pvc-deploy-app
  template:
    metadata:
      labels:
        app: pvc-deploy-app
    spec:
      containers:
      - name: pvc-deploy-app
        image: arisu1000/simple-container-app:latest
        ports:
        - containerPort: 8080
        imagePullPolicy: Always
        volumeMounts:
        - mountPath: "/tmp"
          name: myvolume
      <b>
      volumes:
      - name: myvolume
        persistentVolumeClaim:
          claimName: pvc-hostpath
      </b>
</pre>

##### PVC에서 hostpath 설정하는 것과 deployment에서 바로 hostpath 설정하는 것의 차이
* hostpath를 그냥 띄워서 사용하는 건 워커노드 하나를 사용하는 경우와 같이 간단한 경우에 쓴다.
  for single node testing only; WILL NOT WORK in a multi-node cluster; consider using local volume instead
