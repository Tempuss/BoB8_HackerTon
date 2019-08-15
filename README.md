# BoB8_HackerTon
BoB8 개발트랙 해커톤

## 주제

* 네트워크 관련된 주제라면 모두 OK(공유기, 공격툴, 방어툴, 관제툴 그외 등등...)

* 하지만 난 웹 개발자... 라서 어떤걸 해야 할까 고민함

* 다른 팀들을 보니 많은 팀들이 OpenWRT를 이용한 공유기 구축 하려고 했음

* 다 공유기 하는데 나도 공유기 하면 망할거 같았음(애초에 구축도 못함)

* 예전에 회사에서 ELK로 API 사용량 Monitoring System 을 사용했던게 생각이 나서 Network Monitoring System을 구축으로 가닥을 잡음

## 설계

* 일단 처음에 생각한 기능은 트래픽 모니터링 기능, Notification 기능이다


*  트래픽 로그를 client 단에서 수집할거냐 GateWay에서 수집하냐는 문제에 봉착

  * client 구축 : 구축 난이도를 생각하면 client에서 구축하는게 유리했겠지만, 그렇게 되면 네트워크 전체에 대한 모니터링이 아니라서 바로 포기

  * GateWay 구축 : 공유기를 따로 구축해야 해서 구축 난이도는 급상승 하지만 진정한 Network Monitoring System이 될거 같아서 선택
 
 * 구상도
 ![KakaoTalk_20190814_221658886](https://user-images.githubusercontent.com/13353498/63094556-c0fea880-bfa3-11e9-9cd5-76e63e0c7c49.png)


 * 일단 팀원들과 이야기 하고 난 후 2명이 ELK 구축 및 로그수집을 담당하고 나머지 3명이 공유기 구축을 하기로 결정!
 
 * ELK 데모 사이트
 
 https://demo.elastic.co/app/kibana#/dashboard/Packetbeat-HTTP-ecs
 
 ### 순서도
 
 1. 1차적으론 공유기를 구축
 
 2. packetbeat로 네트워크 트래픽 로그를 Logstash로 전송
 
 3. Logstash에서 ElasticSearch로 Insert 
 
 4. Kibana로 시각화
 
 설계가 나왔으니 삽질 시작.
 
 
## 실행

* 후... 일단 ELK 구축은 수월했다 이전에 구축해놓은 시스템이 남아있어서 그걸 재사용 하기로 해서 시간을 많이 벌었다

* 는 훼이크고 ELK 구축을 끝내고 외부 IP접근 가능하게 열었는데 갑자기 ElasticSearch가 죽어버림 -> 왜...?

* 알아보니 외부 IP로 열 경우 configuration이 좀 더 엄격해져서 max file description과 max virtual memory관련 오류가 발생

[관련 문제 해결 참조](https://kugancity.tistory.com/entry/elasticsearch-51-관련-설정-변경)

* 근데 18.04 버전에서는 제대로 작동을 안해서 찾아 보니 GUI 버전과 CLI 버전에서의 계정? 문제로 따로 설정을 해줘야 max file description이 수정된다 -> ELK 구축 성공(삽질 2시간)


* 공유기 구축에서 문제가 많았다

### 문제발생 기록

* OpenWRT에 PacketBeat를 설치 할 수가 없음

* 일단 여기서 공유기 구축 팀원들 멘붕

* netflow를 이용해서 OpenWRT에 로그 수집 시스템을 구축 할 수 있다는 글을 발견 -> 특정 버전의 OpenWRT에서만 작동함

* 그래서 그 특정버전의 OpentWRT를 다시 깔고 구축시도 -> netflow 설치 실패...

* 공유기 구축은 실패했다고 판단 -> 다른 공유기 구축팀과 연동을 해볼까 고민해봄 -> 당시 시간이 약 새벽 3시였는데 대부분의 팀들이 공유기 구축에 아직 성공을 못했었음 그리고 로그 수집 시스템 구축하는데 시간이 오래걸리기도 한다 

* 방법을 찾다가 ubuntu를 이용해서 핫스팟을 구축하면 되지 않을까! 하는 팀원의 의견 -> 구축시도

* 는 실패 이번에 처음 알게 된건데 가상머신 OS에서는 노트북에 달린 기본 무선랜카드는 쓸수가 없다고 한다(왜...?) -> 팀원들 멘붕 새벽 4시에 어디서 USB 무선 랜카드를 구하라고 ㅋㅋㅋㅋㅋㅋㅋㅋ

* 그와중에 라즈베리 파이로 공유기를 구축하는 다른 팀에서 무선USB 랜카드 남는게 있어서 이 랜카드를 하사 받음 -> 아멘...

* 본인의 가상 ubuntu 핫 스팟 생성 -> 전체 패킷중 60프로 이상이 retransmission이 발생할 만큼 패킷 손실률이 높아서 아 이거 안되겠는데.. 라고 멘붕함 -> 지금 생각해보니 그 ubuntu에 ElasticSearch가 돌아가고 있어서 그냥 느렸던거 같다

* 다른 팀원의 노트북에 핫 스팟 시스템 구축 -> 속도가 아주 잘나오는걸 확인!

* 핫 스팟 ubuntu에 packetbeats 구축 -> logstash로 전송-> 별다른 어려움 없이 작동됨

* ElasticSearch에 실시간으로 데이터 들어오늘걸 확인 -> 트래픽 사용량을 증가시키기 위해 팀원들에게 게임을 시켰음

* 이제 내 차례 -> ElasticSearch 모델링을 하고 들어온 데이터 중에서 트래픽 모니터링을 위해 필요한 데이터들을 시각화 하기 시작

* 사실 이때 이미 멘탈이 맛이 가서 뭘 뽑아야 하는지 몰랐음

* 일단 뽑은 데이터 리스트

  1. 트래픽량 카운트
  
  2. 프로토콜별 분포도
  
  3. 시간별 트래픽 량
  
  4. HTTP 응답코드 별 분포도
  
  5. 통신했던 최다 IP 리스트

* ELK 시각화 화면

![image](https://user-images.githubusercontent.com/13353498/63096061-ceb62d00-bfa7-11e9-909e-676120475e89.png)


* 자 일단 어설프긴 해도 시각화는 됬다! -> 다음 이슈 처리하자 노예야

* 대부분 모니터링 도구에는 Notification 기능이 있으니 우리도 만들어야지 -> Slack WebHook으로 구현

* 처음엔 python의 elasticalert를 사용하려고 시도함 -> 1시간 30분 삽질끝에 포기(거의 제출 완료 시간이었음)

* 어카지? 라는 생각과 함께 또 미친짓이 떠오름 -> PHP로 ElasitcSearch에서 데이터 조회 후 WebHook을 요청하자!

* 근데 PHP는 웹인데? -> Command Line으로 돌릴수 있다 애송아(crontab으로 등록가능)

* 결국 20분 만에 짧은 코딩으로 메시지 전송 성공

* Slack Notification 화면

![image](https://user-images.githubusercontent.com/13353498/63096282-85b2a880-bfa8-11e9-83c4-28e64c188c3c.png)

* 뚝딱 다 만듬 -> 갓갓 PHP

* 처음에 설계했던 기능들은 다 구축 했다 (트래픽 시각화 / Notification)



## 후기

* 회사 그만두고 되게 오랜만에 진짜 집중해서 빡쎄게 개발했던거 같다 (특히 마지막 2시간 동안에 다른 사람들 목소리도 거의 안들릴 정도로 몰입함)

* 개발하다가 과로로 허리디스크가 터졌는데도 개발이 재밌는거 봐선 미친놈 인게 분명하다

* 회사 그만두고 재활운동해서 진통제를 끊었는데 진통제 처먹으면서 개발했다

* 밤새지 말자

* 밤 잘새는거 좋은거 아니다

* 우리 몸은 소모성이다 아껴쓰자

* 살려줘

