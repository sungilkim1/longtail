# 분석관련 로컬 개발 환경
  - (local) openshift: Red Hat CodeReady Containers(CRC) v4.9.10
  - kafka: strimzi v0.26.1
  - elk stack: logstach-elasticsearch-kibana 
  - service app: flask/python
 
# 샘플 구성
  - kafka producer(flask/python) - kafka - logstash - elasticsearch - kibana
  - 4 kafka topics(es index): order, dispatch, delivery, charge
    
# 설치 방법
1. CRC 설치/설정/로그인: 
   - 다운로드: https://developers.redhat.com/products/codeready-containers/overview
   - crc setup
   - crc config set memory 16384: kafka와 es설치로 메모리 좀 많이 필요
   - crc start
   - crc oc-env | Invoke-Expression
   - oc login -u kubeadmin(or developer)
   - web console 접속
 
![image](https://user-images.githubusercontent.com/80303046/147523438-d2e68776-9613-4c42-a096-cd03352116da.png)

 
 2. kafka producer 설치
    - $longtail-producer로 이동
    - namespace 생성: kubectl create ns longtail-producer
    - openshift용 image 빌드: oc new-app . -n longtail-producer
     .(git연동시) oc new-app <git주소> -n longtail-producer
    - web console에서 longtail-producer용 route 생성
   
 3. strimzi(kafka) 설치
    - $longtail-kafka로 이동
    - kubectl create ns kafka
    - kubectl create ns my-kafka-project
    - kubectl create -f install/cluster-operator/ -n kafka
    - kubectl create -f install/cluster-operator/020-RoleBinding-strimzi-cluster-operator.yaml -n my-kafka-project
    - kubectl create -f install/cluster-operator/031-RoleBinding-strimzi-cluster-operator-entity-operator-delegation.yaml -n my-kafka-project
    - kubectl create -f kafka-cluster-descriptor.yaml -n my-kafka-project
    - kubectl wait kafka/my-cluster --for=condition=Ready --timeout=300s -n my-kafka-project
    - kubectl create -f .\kafka-topic.yaml -n my-kafka-project
    - kafka 테스트 방법
      - oc run kafka-producer -ti --image=strimzi/kafka:0.18.0-kafka-2.5.0  --rm=true  --restart=Never -- bin/kafka-console-producer.sh  --broker-list my-cluster-kafka-  bootstrap.my-kafka-project:9092  --topic my-topic
      - (다른 terminal) oc run kafka-consumer -ti --image=strimzi/kafka:0.18.0-kafka-2.5.0  --rm=true  --restart=Never -- bin/kafka-console-consumer.sh  --bootstrap-server my-cluster-kafka-bootstrap.my-kafka-project:9092  --topic my-topic --from-beginning
                       
 4. logstash 설치
    - $longtail-elk로 이동
    - kubectl create ns logging 
    - kubectl create configmap longtail-log-pipeline --from-file longtail-log-es.conf -n logging
    - kubectl create -f logstash.yaml  -n logging
    
 5. elasticsearch 설치
    - kubectl create -f elastic_search.yaml -n logging
    
 6. kibana 설치
    - kubectl create -f kibana.yaml -n logging

7. 테스트 방법
    - longtail-producer용 route url에서 임의의 값을 입력
      - topic: order, dispatch, delivery, charge
      - 예) http://producer-route-longtail-producer.apps-crc.testing/delivery?user_id=1111&delivery_item=item2
    - kibana > dev tools > console에서 query
      - (예) GET /delivery/_search
            {
              "query": {
                "match_all": {}
              }
            }
    
