
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

Pod에서 실행되는 Application을 외부에서 접근 가능한 상태로 만드는 추상적인 네트워크 서비스.

Kubernetes를 사용하면 익숙하지 않은 서비스 검색 메커니즘을 사용하기 위해 응용 프로그램을 수정할 필요가 없으며

Kubernetes는 포드에 고유 한 IP 주소와 포드 세트에 대한 단일 DNS 이름을 제공하고 이들간에 로드 밸런싱을 실행.







서비스 DNS

K8S에서는 클러스터 내부에서 사용가능한 DNS를 설정할 수 있다.

K8S DNS는 클러스터 생성시 시스템 구성요소로 설치됨.

POD간 통신을 위해 IP가 아닌 도메인으로 통신을 가능하게 해주며 도메인 설정시 클러스터 IP가 변경되는 경우에도 별다른 이상 없이 동작함.



준비 상태 검사(readinessProbe)



애플리케이션을 시작할 때 요청을 처리할 수 있는 상태까지 일반적으로 수초 에서 수분까지 시간이 소요되며 준비상태를 확인하며 어떤 POD이 요청을 처리 할 수 있는 상태인지 추적하는 기능.

해당 기능은 서비스 객체를 이용하며 동작한다.

readinessProbe:
            httpGet:
              path: /_/health
              port: http
            initialDelaySeconds: {{ .Values.probe.initialDelaySeconds }}
            periodSeconds: 11


준비 상태 검사는 과부하 상태 또는 장애 상황에서 서버에 트래픽을 전달하면 안된다는 것을 시스템에 알리는 방법.

정상적인 종료를 구현하는 방법으로 사용되며 서버는 트래픽이 필요없다는 신호와 기존 연결이 닫힐때까지 기다린다음 종료. 



Service 생성 방법

kubectl 이용

kubectl run alpha-prod --image=alpha-prod --replicas=3 --port=8080 --labels="ver=2,app=alpha,env=prod"
kubectl expose deployment alpha-prod
kubectl run alpha-prod --image=beta-prod --replicas=2 --port=8080 --labels="ver=2,app=beta,env=prod"
kubectl expose deployment beta-prod
템플릿 이용 예시

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
 
 


서비스 타입 종류

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




엔드 포인트



일부 애플리케이션과 시스템의 경우 클러스터  IP를 사용하지 않고 서비스의 접근을 요구한다.

이러한 작업은 Endpoint로 수행되며 모든 서비스 객체에 대해 K8S는 해당 서비스의 IP주소가 포하된 버디 엔드포인트를 생성.

*버디 엔드포인트 : 다중 IP주소로 구성된 주소를 가진 엔드포인트



엔드포인트는 K8S의 시작과 함께 실행 되도록 새로운 코드를 작성하는 경우 유용하나 대부분의 프로젝트에서 사용할 수 있는 방법은 아니다.



kube-proxy와 cluster ip

클러스터 IP는 서비스의 모든 엔드포인트에 트래픽을 로드밸런싱하는 안정적인 가상 IP이다.

이러한 기술은 클러스터의 모든 노드에서 실행되는 kube-proxy에 의해 수행된다.



쿠버네티스에서 서비스를 만들었을 때 나오는 ClusterIP나 NodePort로 접근하게 해주는건 큐브프록시(kube-proxy)입니다. 
큐브프록시는 쿠버네티스 클러스터의 각 노드마다 실행되고 있으면서 클러스터 내부 IP로 연결되기 바라는 요청을 적절한 곳으로 전달해 주는 역할을 합니다. 
큐브프록시가 네트워크를 관리하는 방법은 userspace, iptables, ipvs 3가지가 있습니다. 초기에는 userspace가 기본 모드였고 현재(2018년 6월)에는 iptables가 기본모드입니다. 
그리고 iptables에서 ipvs 모드로 넘어가려고 하고 있습니다.

출처: https://arisu1000.tistory.com/27839 [아리수]





iptables 모드
iptables 모드는 다음 그림같은 구조를 가집니다. 
userspace모드와 다른점은 큐브프록시는 iptable을 관리하는 역할만 하고 직접 클라이언트로 부터 트래픽을 받지는 않습니다. 
클라이언트로부터 오는 모든 요청은 iptables을 거쳐서 직접 포드로 전달됩니다. 그래서 userspace모드보다 빠른 성능을 가지게 됩니다. 
userspace모드에서는 포드 하나로의 요청이 실패하면 자동으로 다른 포드로 연결을 재시도 하지만 iptables모드에서는 포드하나로 가는 요청이 실패하면 재시도를 하지 않고 그냥 요청이 실패합니다. 
그래서 이런걸 방지하기 위해서 readiness probe 설정이 잘 되어 있어야 합니다.
출처: https://arisu1000.tistory.com/27839 [아리수]



요약

서비스 객체를 사용하여 서비스를 서로 연결 하고 클러스터 외부로 서비스를 제공 할 수 있다.
