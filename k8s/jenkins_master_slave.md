# 쿠버네티스 환경에서 젠킨스 마스터 슬레이브 설정
참고 링크: https://medium.com/swlh/quick-and-simple-how-to-setup-jenkins-distributed-master-slave-build-on-kubernetes-37f3d76aae7d

## 젠킨스 마스터
### 플러그인 설치
* Kubernetes plugin
* Docker plugin
* Git plugin
* Node and Label Parameter Plugin
![image](https://user-images.githubusercontent.com/33820372/112439171-22941500-8d8c-11eb-9046-beef0eefc015.png)

#### 플러그인 설치시 오류가 난다면
"Dashboard > Update Center > 고급 > 업데이트 사이트"에 있는 사이트경로를 https에서 http로 바꾸기.
![image](https://user-images.githubusercontent.com/33820372/112439102-0d1eeb00-8d8c-11eb-99f0-cab426fa36a8.png)


### 젠킨스 클라우드 구성
#### 젠킨스 마스터 service account를 credential manager에 등록
