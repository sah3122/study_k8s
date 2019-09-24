
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