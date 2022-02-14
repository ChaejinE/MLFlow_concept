# Intro
```shell
pip install mlflow
```
- 특정 MLflow 모듈들이나 function들을 사용하기 위해서는 extra libraries를 Install 할필요가 있다.
  - ML model persistence/inference
  - artifact storage option
  - etc ...
  - ex) mlflow.tensorflow module -> tensorflow가 필요
  - MLflow skinny는 다른 dependency들도 설치해줘야한다.
    - mlflow.set_tracking_uri("uri") -> mlflow-skinny, sqlalchemy alembic sqlparse 등 pip install이 필요
  - 이에대한 자세한 설명은 tutorial을 통해 할 수 있을 것 같다.

# Downloading the Quickstart
```shell
git clone https://github.com/mlflow/mlflow
```

# Using the Tracking API
- MLflow Tracking API는 데이터 사이언스 code 로부터 metrics, artifacts files를 logging 하도록 해준다.
  - 내가 실행한 history를 포함한다.

```shell
cd mlflow
cd examples
python quickstart/mlflow_tracking.py
```

```python
import os
from random import random, randint
from mlflow import log_metric, log_param, log_artifacts

if __name__ == "__main__":
    # Log a parameter (key-value pair)
    log_param("param1", randint(0, 100))

    # Log a metric; metrics can be updated throughout the run
    log_metric("foo", random())
    log_metric("foo", random() + 1)
    log_metric("foo", random() + 2)

    # Log an artifact (output file)
    if not os.path.exists("outputs"):
        os.makedirs("outputs")
    with open("outputs/test.txt", "w") as f:
        f.write("hello world!")
    log_artifacts("outputs")
```
- mlflow_tracking.py의 내용은 위와 같다.

```shell
vim outputs/test.txt
```

```txt
hello world!
```
- outputs 폴더에 test.txt 파일이 다음과 같이 저장된다.

# Viewing the Tracking UI
- default로 program이 실행할 떄마다 tracking API가 local의 **./mlruns** 디렉토리에 데이터를 쓴다. 이를 참고해 ui를 update 한다.

```shell
mlflow ui
```
- MLFlow의 Tracking ui를 실행한다.
- http://localhost:5000에서 확인할 수 있다.

![image](https://user-images.githubusercontent.com/69780812/153746404-394444e8-30cd-4167-a22a-cf1f9b7e86e8.png)
- 위와 같은 창이 뜨면 성공이다.

![image](https://user-images.githubusercontent.com/69780812/153746474-ff79610f-f6da-49ac-8d03-0ac013591fd8.png)
- 두번 더 실행해보니 정상적으로 Tracking이 되고있다.
- source 항목에서 mlflow_tracking.py 라는 파일을 run 한 것이 확인된다.

# Running MLflow Projects
- MLflow는 project로써 dependency, code를 패키징할 수 있게해준다.
- 각 project는 코드를 포함하고 있고, MLproject file을 통해 dependency들을 정의한다.
  - Python 환경 등을 정의한다.
- 또한, 프로젝트를 실행할 때 run할 command들, 해당 command를 실행하기 위한 argument들을 포함한다.

```shell
mlflow run sklearn_elasticnet_wine -P alpha=0.5 # or
mlflow run https://github.com/mlflow/mlflow-example.git -P alpha=5.0
```
- mlflow run command를 통해 만들어 놓은 project들을 쉽게 실행 가능하다.
  - project는 local directory에서 또는 GitHub URI를 통해 실행될 수 있다.

```shell
mlflow run --no-conda sklearn_elasticnet_wine -P alpha=0.5
```
- 이미 conda 환경을 만들어서 사용중이어서 --no-conda를 통해 환경을 참조하지 않게했다.
  ```shell
  conda install -c anaconda scikit-learn
  ```

```
2022/02/14 10:22:26 INFO mlflow.projects.utils: === Created directory /tmp/tmpibjt_cdt for downloading remote URIs passed to arguments of type 'path' ===
2022/02/14 10:22:26 INFO mlflow.projects.backend.local: === Running command 'python train.py 0.5 0.1' in run with ID '82fbd955db3647a48431142bb1c76c0c' === 
Elasticnet model (alpha=0.500000, l1_ratio=0.100000):
  RMSE: 0.7460550348172179
  MAE: 0.576381895873763
  R2: 0.21136606570632266
2022/02/14 10:22:30 INFO mlflow.projects: === Run (ID '82fbd955db3647a48431142bb1c76c0c') succeeded ===
```

![image](https://user-images.githubusercontent.com/69780812/153785075-6d15c22c-f3b4-471c-90a1-235aa0ca8d26.png)
- UI에서도 Tracking API를 통해 정상적인 실행을 확인했다.
- RMSE, MAE, r2가 열로 log된 것으로 보아 logging 정보가 입력된 프로그램인 것을 추측할 수 있다.
- 실제로 확인해보자.

![image](https://user-images.githubusercontent.com/69780812/153785281-12ab51fa-738c-4ddf-ac0e-869364c29ed6.png)
- 실제 Project의 구성은 위와 같은 형식을 따르면된다.

```yaml
name: tutorial
channels:
  - conda-forge
dependencies:
  - python=3.7
  - pip
  - pip:
      - scikit-learn==0.23.2
      - mlflow>=1.0
      - pandas
```
- conda 환경을 정의하는 것은 yaml로 작성되어 있다.

```yaml
name: tutorial

conda_env: conda.yaml

entry_points:
  main:
    parameters:
      alpha: {type: float, default: 0.5}
      l1_ratio: {type: float, default: 0.1}
    command: "python train.py {alpha} {l1_ratio}"
```
- MLProject 파일은 위와 같이 작성함으로써 관리한다.

```python
mlflow.log_param("alpha", alpha)
mlflow.log_param("l1_ratio", l1_ratio)
mlflow.log_metric("rmse", rmse)
mlflow.log_metric("r2", r2)
mlflow.log_metric("mae", mae)
```
- 실제 train.py에서 log를 저장하는 코드가 실행되고 있었다.
- dependency들을 정의하는 자세한 방법은 tutorial을 참고한다.
- tracking server를 정의하지 않았다면 프로젝트 log는 Tracking API에 의해 local의 mlruns 디록토리에 기록되며 mlflow ui를 통해 확인할 수 있다.
- default로 mlflow run 은 conda를 사용해서 모든 Dependency들을 설치한다. conda를 사용하지 않고 project를 run 하려면 --no-conda option을 붙여서 mlflow run command를 실행한다. 이 경우, 개인이 실행하고자하는 파일에 대한 Python 환경에서 필수적인 dependency들은 설치되어있야한다.

# Saving and Serving Models
- MLflow는 제네릭 MLmodel을 제공한다.
- 많은 model들은 Python function들로 제공될 수 있다.
- MLmodel file을 통해 어떻게 모델이 Python function으로 interprete 되야 하는지 선언할 수 있다.
- Docker container or 상업적인 serving platform으로 export할 수 있다.

```python
python sklearn_logistic_regression/train.py
```

```shell
Score: 0.6666666666666666
Model saved in run 5d8d4c25b3314e429bd557ca324d4689
```
- 위 코드를 실행하면 experiment에대한 MLflow의 run ID가 출력된다.

![image](https://user-images.githubusercontent.com/69780812/153804015-1d7697bf-e836-47e0-88d2-151ca5ffdcfe.png)
- mlflow ui를 통해 ui를 확인해보면 위와같이 saved model을 확인해볼 수 있다.
  - sklearn model이므로 pickle 모델이 저장된다.
  
![image](https://user-images.githubusercontent.com/69780812/153804128-347a8a0e-2017-44c1-821b-902ad1ef3838.png)  
- saved model을 클릭해보면 MLmodel에 대한 설명을 포함하고있다.

```shell
mlflow models serve -m runs:/<RUN_ID>/model
```
- mlflow는 run id와 model의 경로를 pass해서 server를 통해 serving 할 수 있다.
  - MLflow는 python 기반 모델을 위해 simple REST server를 포함하고 있다.
- 위에서 /runid/model의 model은 정해주면 되는 이름이며 artifacts directory라고 부른다. 이 폴더안에 모델이 저장된다.
- 위 코드는 deafult로 5000 port로 ui 서버가 Run되기때문에 다른 port를 쓰려면 포트를 지정해줘야한다.
  ```shell
  mlflow models serve -m runs:/<RUN_ID>/model --port 1234
  ```
  - Test 중 5000 포트를 ui로 사용하고 있으므로 --port를 통해 다른 port를 지정해주자.
  ```shell
  mlflow models serve -m runs:/5d8d4c25b3314e429bd557ca324d4689/model --no-conda --port 1234
  ```
  - 이미 환경이 셋팅되어있는 python 환경에서는 --no-conda를 사용해준다.

```shell
curl -d '{"columns":["x"], "data":[[1], [-1]]}' -H 'Content-Type: application/json; format=pandas-split' -X POST localhost:1234/invocations

result : [1, 0]
```
- curl을 이용해 json-serialized pandas Dataframe을 보내본다.
- 해당 포트로 serving되어있는 모델로 값이 전달되고, 원하는 출력을 Return 받을 수 있다.

# Logging to a Remote Tracking Server
- running 시 MLflow는 local에 파일을 저장한다.
- 결과들을 확실히 공유하고 관리하기 위해서는 MLflow의 remote tracking server를 설정할 수 있다.

```python
import mlflow
mlflow.set_tracking_uri("http://My-Server:4040")
mlflow.set_experiment("my-experiment")
```

```python
import mlflow
mlflow.set_tracking_uri("databricks")
# Note: on Databricks, the experiment name passed to set_experiment must be a valid path
# in the workspace, like '/Users/<your-username>/my-experiment'. See
# https://docs.databricks.com/user-guide/workspace.html for more info.
mlflow.set_experiment("/my-experiment")
```
- Databricks 커뮤니티 edition을 가입하면 무료로 호스팅 되어있는 tracking server를 사용할 수도 있다.
