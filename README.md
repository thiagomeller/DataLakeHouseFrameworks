# Usando Docker-Compose para criar o ambiente Iceberg-Spark

Criando o arquivo docker-compose.yml
<code>
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
</code>

Rodando o arquivo
<code>
    docker-compose up
</code>

Vamos utilizar o PySpark então
<code>
    docker exec -it spark-iceberg pyspark
</code>


## Instalando PySpark no notebook Jupyter

Instalando a biblioteca PySpark
<code>
    !pip install pyspark
</code>

Iniciando uma sessão Spark
<code>
    from pyspark.sql import SparkSession

    spark = SparkSession \
        .builder \
        .appName("Python Spark SQL basic example") \ #Nome do aplicação
        .config("spark.some.config.option", "some-value") \ #Configuração
        .getOrCreate()
</code>