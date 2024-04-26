# Criando Apache Spark com Delta Lake e Apache Iceberg

### Pré-Requisitos:

- [Visual Studio Code](https://code.visualstudio.com/download)
- [Python 3](https://www.python.org/downloads/)
- [Docker](https://docs.docker.com/get-docker/)

## Delta Lake - Spark

### Instalação do Scoop e do Poetry

-> Scoop é um command-line installer

```powershell
  Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
  Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

-> Poertry é um package manager que gera um ambiente virtual

```python
  scoop install pipx
  pipx ensurepath
```

-> Iniciando o Poetry e abrindo o terminal dele

```python
  poetry init
  poetry shell
```

-> Adicionando as libs

```
  <caminho/do/python.exe/dentro/do/venv> -m pip install pyspark
  <caminho/do/python.exe/dentro/do/venv> -m pip install delta-spark
```

## Apache Iceberg

#### Usando Docker-Compose para criar o ambiente Iceberg-Spark

1. Criando o arquivo docker-compose.yml

```
version: "3"

services:
  spark-iceberg:
    image: tabulario/spark-iceberg
    container_name: spark-iceberg
    build: spark/
    networks:
      iceberg_net:
    depends_on:
      - rest
      - minio
    volumes:
      - ./warehouse:/home/iceberg/warehouse
      - ./notebooks:/home/iceberg/notebooks/notebooks
    environment:
      - AWS_ACCESS_KEY_ID=admin
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_REGION=us-east-1
    ports:
      - 8888:8888
      - 8080:8080
      - 10000:10000
      - 10001:10001
  rest:
    image: tabulario/iceberg-rest
    container_name: iceberg-rest
    networks:
      iceberg_net:
    ports:
      - 8181:8181
    environment:
      - AWS_ACCESS_KEY_ID=admin
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_REGION=us-east-1
      - CATALOG_WAREHOUSE=s3://warehouse/
      - CATALOG_IO__IMPL=org.apache.iceberg.aws.s3.S3FileIO
      - CATALOG_S3_ENDPOINT=http://minio:9000
  minio:
    image: minio/minio
    container_name: minio
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=password
      - MINIO_DOMAIN=minio
    networks:
      iceberg_net:
        aliases:
          - warehouse.minio
    ports:
      - 9001:9001
      - 9000:9000
    command: ["server", "/data", "--console-address", ":9001"]
  mc:
    depends_on:
      - minio
    image: minio/mc
    container_name: mc
    networks:
      iceberg_net:
    environment:
      - AWS_ACCESS_KEY_ID=admin
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_REGION=us-east-1
    entrypoint: >
      /bin/sh -c "
      until (/usr/bin/mc config host add minio http://minio:9000 admin password) do echo '...waiting...' && sleep 1; done;
      /usr/bin/mc rm -r --force minio/warehouse;
      /usr/bin/mc mb minio/warehouse;
      /usr/bin/mc policy set public minio/warehouse;
      tail -f /dev/null
      "
networks:
  iceberg_net:
```

2. Rodando o arquivo

```docker
docker-compose up
```

4. Vamos utilizar o PySpark então

```docker
docker exec -it spark-iceberg pyspark
```

#### Instalando PySpark no notebook Jupyter

1. Instalando a biblioteca PySpark

```python
!pip install pyspark
```

2. Iniciando uma sessão Spark

```python
from pyspark.sql import SparkSession
spark = SparkSession \
    .builder \
    .appName("Python Spark SQL basic example") \ #Nome do aplicação
    .config("spark.some.config.option", "some-value") \ #Configuração
    .getOrCreate()
```
