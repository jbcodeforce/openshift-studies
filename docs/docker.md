# Some docker and docker compose tricks

## Dockerfile

ENTRYPOINT specifies the default command to execute when the image runs in a container
CMD provides the default arguments for the ENTRYPOINT instruction

Example to build a custom Apache Web Server container image.

```dockerfile
FROM ubi7/ubi:7.7
ENV PORT 8080
RUN yum install -y httpd && yum clean all
RUN sed -ri -e "/^Listen 80/c\Listen ${PORT}" /etc/httpd/conf/httpd.conf && \
    chown -R apache:apache /etc/httpd/logs/ && \
    chown -R apache:apache /run/httpd/
USER apache
EXPOSE ${PORT}
COPY ./src/ /var/www/html/
CMD ["httpd", "-D", "FOREGROUND"]
```

## Run a ubuntu image

This could be a good approach to demonstrate a linux based.

```shell
docker run --name my-linux  --detach ubuntu:20.04 tail -f /dev/null
# connect
docker exec -ti my-linux bash
# Update apt inventory and install jdk, maven, git, curl, ... 
apt update
apt install -y openjdk-11-jre-headless maven git curl vim
```

This project includes a Dockerfile-ubuntu to build a local image with the above tools.

```shell
docker build -t jbcodeforce/myubuntu:20.04 -f Dockerfile-ubuntu .
```

## Docker volume

For mounting host directory, the host directory needs to be configured with ownership and permissions allowing access to the container.

```shell
docker run -v /var/dbfiles:/var/lib/mysql rhmap47/mysql
```

## Reclaim disk space

```shell
docker system df
```

(https://rmoff.net/post/what-to-do-when-docker-runs-out-of-space/)

## Docker network

```shell
docker network list
# create a network
docker network create kafkanet
# Assess which network a container is connected to
docker inspect 71582654b2f4 -f "{{json .NetworkSettings.Networks }}"

> "bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"NetworkID":"7db..."

# disconnect a container to its network
docker network disconnect bridge 71582654b2f4
# Connect an existing container to a network
docker network connect docker_default containernameorid
```

## Start a docker bypassing entry point or cmd

```shell
docker run -ti --entrypoint "/bin/bash" imagename
```

or use the command after the image name:

```shell
docker run -ti imagename /bin/bash 
```

## Docker build image with tests and env variables

Inject the environment variables with --build-arg

```shell
docker build --network host \
            --build-arg KAFKA_BROKERS=${KAFKA_BROKERS} \
            --build-arg KAFKA_APIKEY=${KAFKA_APIKEY} \
            --build-arg POSTGRESQL_URL=${POSTGRESQL_URL}  \
            --build-arg POSTGRESQL_USER=${POSTGRESQL_USER} \
            --build-arg POSTGRESQL_PWD=${POSTGRESQL_PWD} \
            --build-arg JKS_LOCATION=${JKS_LOCATION} \
            --build-arg TRUSTSTORE_PWD=${TRUSTSTORE_PWD} \
            --build-arg POSTGRESQL_CA_PEM="${POSTGRESQL_CA_PEM}"  -t ibmcase/$kname .

```

## Define a docker compose to run python env
