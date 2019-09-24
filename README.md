
라벨과 애노테이션



라벨

pod 과 replicaset 같은 k8s 내부 객체에 첨부 가능한 key - value 
라벨은 객체를 그룹화 하는 기초 제공
라벨 생성

kubectl run test-app --image=alpha-prod --replicas=2 --labels="ver=1,app=alpha,env=prod"
kubectl run test-app --image=alpha-test --replicas=2 --labels="ver=2,app=alpha,env=prod"
kubectl run test-app --image=beta-prod --replicas=2 --labels="ver=2,app=beta,env=prod"
kubectl run test-app --image=beta-staging --replicas=2 --labels="ver=3,app=beta,env=staging"
라벨 수정

kubectl label deployments alpha-prod "newlabel=new"



라벨 조회

전체

kubectl get pods --show labels

selector 이용

kubectl get pods --selector="ver=2"

kubectl get pods --selector="ver=2,app=beta" // AND 연산

표현식 이용

kubectl get pods --selector="app in (alpha,beta)" // app 라벨이 alpha 또는 beta인 pod 조회



애노테이션

라벨과 비슷하나 도구와 라이브러리에서 활용할 수 있게 식별 불가능한 정보를 유지하기 위한 key-value쌍
애노테이션과 라벨은 일부 기능이 겹치나 사용하는 case나 취향에 따라 선택하여 사용


용도

객체에 대한 최신 업데이트 이유 추적
특별한 스케줄링 정책을 스케줄러에 전달.
최신 도구에 대한 데이터를 확장하여 자원을 업데이트한 내용과 업데이트 방법
라벨에 적합하지 않은 작성, 릴리즈. 또는 이미지 정보(git hash, timestamp, pr number 등)
롤아웃을 관리하는 replicaset을 추적하기 위해 deployment객체를 활성화
example 

metadata:
	annotations:
		example.com/icon-url: "https://example.com/icon.png"


요약

라벨은 k8s에서 객체 식별 및 선택적 그룹화를 위하여 사용.

애노테이션은 자동화 도구 및 클라이언트 라이브러리에서 사용할 수 있는 메타 데이터 저장용도로 사용.


서비스 객체

Pod에서 실행되는 Application을 외부에서 접근 가능한 상태로 만드는 추상적인 네트워크 서비스.

Kubernetes를 사용하면 익숙하지 않은 서비스 검색 메커니즘을 사용하기 위해 응용 프로그램을 수정할 필요가 없으며

Kubernetes는 포드에 고유 한 IP 주소와 포드 세트에 대한 단일 DNS 이름을 제공하고 이들간에 로드 밸런싱을 실행.







Service 생성 방법

kubectl 이용

kubectl run alpha-prod --image=alpha-prod --replicas=3 --port=8080 --labels="ver=2,app=alpha,env=prod"
kubectl expose deployment alpha-prod
kubectl run alpha-prod --image=beta-prod --replicas=2 --port=8080 --labels="ver=2,app=beta,env=prod"
kubectl expose deployment beta-prod
템플릿 이용

kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  type: ClusterIP
  clusterIP: 10.0.10.10
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376


서비스 종류

ClusterIP
기본 서비스 타입이며 K8S 클러스터 내부에서 사용 가능.
클러스터 내부 Node 및 Pod에서 ClusterIP를 이용한 통신 가능.
NodePort
각 노드의 Port를 할당하는 방식. 
Node1:8080
Node2:8082
적용하기 가장 간편함,
노드의 Port를 사용하기 때문에 외부에서도 접근 가능
LoadBalancer
클라우드 서비스를 이용하는 상황에서 적용 가능
Pod을 클라우드에서 제공하는 LoadBalancer와 연결하여 외부에서 접근이 가능하도록 설정함.
ExternalName
Service를 externalName의 값이랑 매칭
클러스터 내부에서 외부로 접근할 때 주로 사용.