# 쿠버네티스 환경에서 젠킨스 마스터 슬레이브 설정
참고 링크: https://medium.com/swlh/quick-and-simple-how-to-setup-jenkins-distributed-master-slave-build-on-kubernetes-37f3d76aae7d

## 젠킨스 마스터
### 플러그인 설치
* Kubernetes plugin  
* Docker plugin  
* Git plugin  
* Node and Label Parameter Plugin (The node and label parameter plugin allows to dynamically select the node on which a job should be executed.)  
![image](https://user-images.githubusercontent.com/33820372/112439171-22941500-8d8c-11eb-9046-beef0eefc015.png)
---
#### 플러그인 설치시 오류가 난다면  
##### ssl 통신 오류  
"Dashboard > Update Center > 고급 > 업데이트 사이트"에 있는 사이트경로를 https에서 http로 바꾸기.  
![image](https://user-images.githubusercontent.com/33820372/112439102-0d1eeb00-8d8c-11eb-99f0-cab426fa36a8.png)   
##### Failed to download from ~ 오류  
1. 해당 사이트에 직접 들어가서 .jpi 혹은 .hpi 파일을 받아온다.   
2. 아래 명령어로 플러그인 파일을 파드 안의 플러그인 폴더에 밀어 넣는다.  
3. 젠킨스를 다시 재기동한다.  
```
kubectl cp .hpi파일 네임스페이스/파드이름:/var/jenkins_home/plugins/
```  
---  

#### 젠킨스 마스터 service account를 credential manager에 등록  
credential 만들 때 secret-text로 생성. token의 이름은 마음대로 설정하되, secret에는 위의 명령어 쳐서 나오는 비밀번호 입력하여 credential 생성한다.  
(쿠버네티스 환경에 접근할 수 있는 권한)  
![image](https://user-images.githubusercontent.com/33820372/113526057-5f190980-95f3-11eb-9036-9f30e5db61ed.png)  
```
kubectl get secret $(kubectl get sa SA이름 -n 네임스페이스이름 -o jsonpath={.secrets[0].name}) -n 네임스페이스이름 -o jsonpath={.data.token} | base64 --decode
```  

### 젠킨스 클라우드 구성  
젠킨스 관리 > 노드 관리 > configure clouds 에서 "kubernetes" 선택하여 새로운 클라우드 환경 생성  
* kubernetes API server address 알아내는 명령어  
```
kubectl config view --minify | grep server | cut -f 2- -d ":" | tr -d " "
```  
* kubernetes server CA certificate key 값 알아내는 명령어  
```
kubectl get secret $(kubectl get sa jenkins-master -n jenkins -o jsonpath={.secrets[0].name}) -n jenkins -o jsonpath={.data.'ca\.crt'} | base64 --decode
```  
![image](https://user-images.githubusercontent.com/33820372/113526004-182b1400-95f3-11eb-9ff6-13a77fe25d81.png)  

#### 젠킨스 슬레이브를 위한 Pod와 Container 설정하기  
jenkinsci/slave 이미지를 끌어와서 슬레이브 pod 설정
![image](https://user-images.githubusercontent.com/33820372/113530419-f9337e80-9600-11eb-8670-71e0fc11f1c0.png)

---

### jenkins job 구성하기
restrict where this project can be run에 jenkins-slave로 설정하기

job에서 git 소스 가져오는 설정 - credential 필요 (git credential 설정은 우측 블로그 참고 https://jojoldu.tistory.com/442)
