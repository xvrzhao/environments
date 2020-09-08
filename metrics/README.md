# Metrics

因本地开发环境，没有做服务发现机制，而是通过 `static_configs` 静态指定 `targets` 的形式让 Prometheus 监控目标。

需要被监控的服务可加入到该 Docker Compose 默认创建的网络中，以便 Prometheus 直接通过容器名称作为 `target hostname` 进行采集；若服务不在容器中运行，可指定 Prometheus `target` 配置为宿主机 `host.docker.internal`。

可在需要被监控应用的 `docker-compose.yml` 中指定 `service` 的 [networks](https://docs.docker.com/compose/compose-file/#networks) 字段来加入该网络，或者直接通过 `docker run --network prometheus_default ...` 来加入。