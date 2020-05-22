# Some docker and docker compose tricks

## reclaim disk space

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

## Define a docker compose to run python env
