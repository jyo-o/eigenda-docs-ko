
# Metrics 및 모니터링

다음은 Prometheus, Grafana, Node exporter stack을 실행하는 quickstart 가이드다.

**Step 1:** 작업 디렉토리를 monitoring 폴더로 이동한다:

```
cd monitoring
cp .env.example .env
```

- `.env` 파일을 열고, `prometheus.yml` 의 위치가 본인 환경에 맞는지 확인한다.
- `prometheus.yml` 파일에서:
  - prometheus 설정 [file](https://github.com/Layr-Labs/eigenda-operator-setup/blob/master/monitoring/prometheus.yml)을 부모 폴더 `.env` 파일의 eigenda node metrics port (`NODE_METRICS_PORT`) 값으로 갱신한다.
  - `scrape_configs.targets` 의 eigenda container 이름이 부모 폴더 `.env` 파일의 값(`MAIN_SERVICE_NAME`)과 일치하는지 확인한다.
  - [.env](https://github.com/Layr-Labs/eigenda-operator-setup/blob/master/monitoring/.env.example) 파일에 prometheus 파일 위치가 올바르게 지정되어 있는지 확인한다.

**Step 2:** 다음 명령으로 monitoring stack을 시작한다.

```
docker compose up -d
```

**Step 3:** eigenda는 다른 docker 네트워크에서 실행 중이므로, prometheus를 같은 네트워크에 두어야 한다. 다음 명령을 실행한다.

```
docker network connect eigenda-network prometheus
```

참고: `eigenda-network` 는 eigenda가 실행 중인 네트워크의 이름이다. 네트워크 이름은 eigenda [.env](https://github.com/Layr-Labs/eigenda-operator-setup/blob/master/mainnet/.env.example#L2) 파일(`NETWORK_NAME`)에서 확인할 수 있다. 이렇게 해야 Prometheus가 Eigenda node에서 metrics를 scrape할 수 있다.

유용한 Dashboard: EigenDA는 monitoring stack 초기화 시 자동으로 import되는 [Grafana dashboards](https://github.com/Layr-Labs/eigenda-operator-setup/tree/master/monitoring/dashboards) 모음을 제공한다.

metrics 및 monitoring stack을 수동으로 설정하고 싶다면 [여기](https://github.com/Layr-Labs/eigenda-operator-setup#metrics)의 절차를 따른다.
