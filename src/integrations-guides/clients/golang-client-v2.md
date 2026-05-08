
# 클라이언트 (Clients)

EigenDA는 하부의 GRPC 클라이언트를 ECDSA keypair 인증 로직으로 감싸는 low-level golang 클라이언트를 제공한다.
EigenDA v2 클라이언트는 [EigenDA repo](https://github.com/Layr-Labs/eigenda/blob/master/api/clients/v2)에서 확인할 수 있다.

v2 go 클라이언트 사용 예시는 다음을 참고한다:
* [Client construction](https://github.com/Layr-Labs/eigenda/blob/master/api/clients/v2/examples/client_construction.go)
* [Retrieval from relay](https://github.com/Layr-Labs/eigenda/blob/master/api/clients/v2/examples/example_relay_retrieval_test.go)
* [Retrieval from validator](https://github.com/Layr-Labs/eigenda/blob/master/api/clients/v2/examples/example_validator_retrieval_test.go).
