# MLflow Tracking Server on Azure ACI

- MLflow tracking serverをAzure Container Instanceで立てる.
- Artifact storeに、Azure Storage Containerを利用する.

# Docker imageのビルド

```bash:
docker compose build
```

または、

```bash:
docker build -t mlflow_server env/Dockerfile_mlflow
```

# Docker imageのプッシュ

イメージをACRに登録する.

```bash:
az acr login -n <registry name>

docker tag mlflow_server resigtry_name.azurecr.io/mlflow_server
docker push registry_name.azurecr.io/mlflow_server
```

# ACIのデプロイ

Azure portalまたは、Azure CLIで、ACIをデプロイする.

```bash:
az container create -g <resource_group_name> --name <container_name> \
 --image registry_name.azurecr.io/mlflow_server \
 --ip-address Public \
 --ports 80 5000 \
 --environment-variables \
  MLFLOW_BACKEND_STORE_URI=sqlite:///backend.db \
  MLFLOW_ARTIFACTS_DESTINATION=wasbs://container_name@storage_name.blob.core.windows.net/Path \
  AZURE_STORAGE_CONNECTION_STRING="..."
```

# MLflowクライアントで、tracking uriを設定する

```python:
import mlflow

mlflow.set_tracking_uri("http://<Container Public IP>:5000")
mlflow.get_tracking_uri()
```

# MLflow ui

ブラウザで、`<IP Address:5000>`にアクセス
