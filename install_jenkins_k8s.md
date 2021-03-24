## 쿠버네티스 환경에서 젠킨스 설치
### 전제조건: 쿠버네티스 환경(로컬이기 때문에 minikube) 설치 필요

구성하고 싶은 구조
![k8s 위의 젠킨스 구조](https://user-images.githubusercontent.com/33820372/112235082-46b6ff80-8c81-11eb-8be4-0a1806448085.png)

Jenkins master는 docker이미지로 설치하고,
Jenkins slave는 구성 후에 jnlp로 통신한다.

### jenkins master 설치
