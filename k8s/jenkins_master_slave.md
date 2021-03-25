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
### 젠킨스 클라우드 구성
#### 젠킨스 마스터 service account를 credential manager에 등록
