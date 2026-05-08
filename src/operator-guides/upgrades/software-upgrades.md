

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


# Software 업그레이드

EigenDA Operator software 업데이트는 다음 채널에서 확인한다:
- [EigenLayer Telegram](https://t.me/EigenCloudSupp)
- [EigenDA Operator Setup](https://github.com/Layr-Labs/eigenda-operator-setup) repository: 새 release 알림을 받으려면 [watch 설정을 구성](https://docs.github.com/en/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications/configuring-notifications#configuring-your-watch-settings-for-an-individual-repository)한다.

docker compose로 node를 운영 중이라면, 아래 단계에 따라 업그레이드를 수행할 수 있다:

#### Step 1: 최신 repo 가져오기

<Tabs groupId="network">
  <TabItem value="mainnet" label="Mainnet">
    ```
    cd eigenda-operator-setup/mainnet
    git pull
    ```
  </TabItem>
  <TabItem value="hoodi" label="Hoodi">
    ```
    cd eigenda-operator-setup/hoodi
    git pull
    ```
  </TabItem>
</Tabs>


release notes에 따라 `.env` 파일의 `MAIN_SERVICE_IMAGE` 를 최신 EigenDA 버전으로 갱신한다.

:::info 
업그레이드 시 따라야 할 특정 지침이 있다면 그 release의 release notes에 함께 안내된다. 서비스를 다시 시작하기 전에 GitHub의 최신 [release notes](https://github.com/Layr-Labs/eigenda-operator-setup/releases)를 확인하고 안내를 따른다.
:::

#### Step 2: 최신 docker image 가져오기

```
docker compose pull
```

#### Step 3: 기존 서비스 정지

```
docker compose down
```

#### Step 4: 서비스 재시작

node를 재시작하기 전에 `.env` 파일의 `TODO` 영역에 올바른 값이 들어 있는지 확인한다.

```
docker compose up -d
```

