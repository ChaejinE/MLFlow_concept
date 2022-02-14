# Introduce
- MLflow는 end-to-end mahcine learning lifecycle을 관리하는 오픈소스 플랫폼이다.
- MLflow의 Primary functions은 4가지로 아래와 같다.
  - 1. MLflow Tracking : parameter & result들을 기록하고 비교하도록 실험한 내용들을 Tracking 한다.
  - 2. MLflow Projects : Data 사이언티스트와 같은 동료들과 결과물을 공유하거나 전달하기 위해 **재사용이 가능하고, 재생산이 가능**한 ML code를 
  패키징한다.
  - 3. MLflow Models : 다양한 ML 라이브러리를 통해 **다양한 모델 제공 및 추론 플랫폼**을 통해 **모델을 관리하고 배포**한다.
  - 4. MLflow Model Registry : 모델 버전 관리, stage 전환 및 annotation을 포함해 MLflow 모델의 전체 라이프 사이클을 공동으로 관리할 수 있는 중앙 모델 저장소를 제공한다.

- MLflow는 라이브러리에 구애받지 않는다.
- 모든 fucntion들은 REST API & CLI를 통해 접근이 가능하기 떄문에 어떤 프로그래밍 언어라던지, 머신러닝 라이브러리든지 사용할 수 있다.
  - 편의를 위해서 Python API, R API, Java API를 포함하고 있다.
