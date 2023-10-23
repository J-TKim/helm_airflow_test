# helm_airflow_test

```bash
airflow helm 1.11.0
airflow version 2.7.3
```

## 설치 명령어
```bash
export NAMESPACE=example-namespace
export RELEASE_NAME=example-release

kind create cluster --image kindest/node:v1.21.1
kubectl cluster-info --context kind-kind

helm repo add apache-airflow https://airflow.apache.org
helm repo update

helm pull apache-airflow/airflow --version 1.11.0 #--namespace $NAMESPACE
tar -zxvf airflow-1.11.0.tgz
cp airflow/values.yaml values.yaml 
python3 -c 'import secrets; print(secrets.token_hex(16))' # values.yaml 에 여기서 나온 값 추가  webserverSecretKey: <시크릿_키>

kubectl create namespace $NAMESPACE

helm install $RELEASE_NAME apache-airflow/airflow \
    --namespace $NAMESPACE \
    --version 1.11.0 \
    -f values.yaml \
    --timeout 600s \
    --debug

docker build --pull --tag my-image:0.0.1 .
kind load docker-image my-image:0.0.1
    
helm upgrade $RELEASE_NAME apache-airflow/airflow \
    --namespace $NAMESPACE \
    --set images.airflow.repository=my-image \
    --set images.airflow.tag=0.0.1 \
    --timeout 600s \
    --debug
```


### 설치 완료되었는지 확인
```bash
export NAMESPACE=example-namespace
export RELEASE_NAME=example-release

kubectl get pods --namespace $NAMESPACE
kubectl get services -n $NAMESPACE
helm list --namespace $NAMESPACE
```

### 웹서버 접속
```bash
# airflow 포트포워딩 (localhost:38080 접속)
kubectl port-forward svc/$RELEASE_NAME-webserver 38080:8080 --namespace $NAMESPACE 
# postgresql 포트포워딩 (localhost:35432 접속)
kubectl port-forward svc/$RELEASE_NAME-postgresql 35432:5432 --namespace $NAMESPACE 
```

### 꼬일 시 아래 명령어로 삭제 후 재설치
```bash
export NAMESPACE=example-namespace
export RELEASE_NAME=example-release

helm uninstall $RELEASE_NAME -n $NAMESPACE
kubectl delete namespace $NAMESPACE
kubectl create namespace $NAMESPACE

```

### Fernet Key 얻는 방법
```bash
export NAMESPACE=example-namespace
export RELEASE_NAME=example-release

echo Fernet Key: $(kubectl get secret --namespace $NAMESPACE $RELEASE_NAME-fernet-key -o jsonpath="{.data.fernet-key}" | base64 --decode)
```


## 참고 링크

[Quick start with kind — helm-chart Documentation](https://airflow.apache.org/docs/helm-chart/stable/quick-start.html)

[airflow 1.11.0 · apache-airflow/apache-airflow](https://artifacthub.io/packages/helm/apache-airflow/airflow/1.11.0)

https://airflow.apache.org/docs/helm-chart/stable/production-guide.html#webserver-secret-key