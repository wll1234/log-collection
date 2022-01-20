# log-collection 적용기
### 왜?
서비스 아키텍쳐를 MSA로 바꾸면서 각각 서비스의 로그를 확인하는데 어려움이 생겼다.  
따라서, 로그를 한 곳으로 모은다면 디버깅은 물론, 추후 운영상에도 도움을 줄 수 있을거라 생각되어 구상하게 됐다.  
이미 여러 곳에서 많이 쓰는 기술들이고 많은 자료가 있지만, 내가 적용하면서 겪은 시행착오와 그 결과를 공유하기위해 적용기를 작성한다.  

### 사용된 기술 및 라이브러리
* Elasticsearch
* Logstash
* Filebeat
* Kibana
* Kafka
* SSL
* Kubernetes
* Docker-compose

### 적용한 프로젝트의 특징
log-collection을 적용하기로한 프로젝트는 다음과 같은 특징을 가졌다.

* OKE(Oracle Kubernetes Engine)과 OCI(Oracle Cloud Instance)에 Docker로 배포된 서비스가 혼재돼있다.
* 로그를 다른 Cloud서버로 전송하여 저장하여야 한다.(용량이슈)  

따라서, log를 수집포인트는 2개(OKE, OCI), 적재포인트는 1개이다.  
수집포인트 2개에 각각 Filebeat, Logstash를 설치하여 Oracle Cloud Stream으로 Produce하고, 적재포인트에서 Logstash로 Consume하여 Elasticsearch에 저장하고 Kibana로 확인하도록 했다.
---
### 설치 (적재포인트)
#### 1. Elasticsearch 설치 (SSL 적용)
`consumer > 1. elasticsearch > 1. Generate SSL Certificates`  
자가서명인증서를 생성 후 Kubernetes의 Secret으로 등록해 준다.  
필요에 따라 스크립트를 수정하여 특정 Namespace나 위치로 이동 등을 설정하여 사용하면 된다.  

`consumer > 1. elasticsearch > 2. elastic-sts.yaml`
Kubernetes Cluster에 elasticsearch를 배포해준다.  
보안 설정을 위해 `xpack.security`가 활성화된 설정이 적용됐다.  
이전 단계에서 생성한 SSL 인증서가 포함되니 secret위치 등을 잘 확인하여 배포하면 된다.  
네임스페이스는 파일 내 NAMESPACE라는 문자로 대치되었으니 본인의 설정에 맞게 바꾸면 된다.  

`consumer > 1. elasticsearch > 3. Generate Elasticsearch Account`
첫 줄의 명력어를 입력하면 그 아랫줄의 결과가 출력된다. 이를 바탕으로 Logstash를 연동하고 Kibana에 로그인 할 수 있다.

#### 2. Kibana 설치
`consumer > 2. kibana > 1. kibana.yaml`
Kubernetes Cluster에 kibana를 배포해준다.  
역시 `xpack`관련 설정이 포함됐고 Elasticsearch에서 생성된 계정과 Ingress Controller를 위한 설정 또한 포함됐다.  
Nodeport를 사용하거나 하면 삭제해도 무방하다.

#### 후추 작성 예정



