
# EigenDA Operator FAQ

#### static IP/DNS 주소가 있는데, 이 주소로 EigenDA에 등록하고 고정하려면 어떻게 하나?

트래픽을 받기 위한 static IP 주소나 DNS 주소를 미리 설정해 두었고 (예: k8s에서 운영 중이거나 EigenDA node 앞에 load balancer를 두었음), 등록 시 EigenDA에 전송되는 IP를 자동으로 갱신하지 않도록 하려면 다음 단계를 따라 올바른 IP가 등록되도록 한다:

* [NODE_HOSTNAME](https://github.com/Layr-Labs/eigenda-operator-setup/blob/31d99e2aa67962878969b81a15c7e8d13ee69750/mainnet/.env.example#L71)을 트래픽을 받을 공인 IP로 갱신한다.
* [provided steps](./run-a-node/registration/)를 따라 opt-in한다.
* node IP 주소가 자동으로 갱신되지 않도록 [NODE_PUBLIC_IP_CHECK_INTERVAL](https://github.com/Layr-Labs/eigenda-operator-setup/blob/31d99e2aa67962878969b81a15c7e8d13ee69750/mainnet/.env.example#L65) 값을 `0` 으로 설정한다.

