# Klaytn Endpoin Node 구성하기

**Klaytn v2.0이 나오며 바뀐 내용들은 아직 체크 못 함**<br>
**아래 문서는 Klaytn v1.7.1 기준**


## Klaytn 개발 환경 - 테스트넷(Baobab)

- [ ] 개발환경 추가

***
<br>

## [:sparkles:설치 가이드](https://ko.docs.klaytn.com/node/endpoint-node/installation-guide)

```shell
sudo apt install wget
wget https://packages.klaytn.net/klaytn/v1.7.1/ken-baobab-v1.7.1-0-linux-amd64.tar.gz

# 버전에 맞게 다운받아 압축 풀기
tar zxf ken-baobab-v1.7.1-0-linux-amd64.tar.gz

# kend 명령어 사용을 위해 모든 사용자 환경변수를 영구적으로 등록
vim /etc/bash.bashrc
```
> 아래 내용 추가<br>
> ```export PATH=$PATH:/home/klaytn/ken-linux-amd64/bin```

```shell
# 내용 추가 후 변경한 환경 변수 적용
source /etc/bash.bashrc

cp ~/ken-linux-amd64/conf/kend_baobab.conf ~/ken-linux-amd64/conf/kend.conf
# kend_baobab.conf를 직접 수정하는 것이 아니라
# 복사한 **kend.conf**를 수정해야 함

# 해당 디렉토리에 블록데이터 싱크를 맞추기 위한 충분한 용량 확보 필요
$ mkdir -p /kend_home
```
- [ ] 용량 관련 확보 관련… 우분투에서 하드디스크 mount하기 - 추가하기

```shell
# 이후 디렉토리 chmod 777 해줘야 파일 쓰기 가능
chmod 777 /kend_home

# 현재 디렉토리에 대한 권한 변경
# chmod 777 ./*

vim ~/ken-linux-amd64/conf/kend.conf
```
> 아래 내용으로 수정: block data 저장할 디렉토리 수정
>DATA_DIR=/kend_home

***
genesis.json 관련 이슈로 인해서 패스했지만<br>
원래라면 해당 스냅샷을 통해 더 빠른 싱크를 맞출 수 있다고 한다...<br>
:thought_balloon:돌이켜 보길, DATA_DIR 설정을 잘못해서 genesis.json에 문제가 있었던 것 같음

```shell
wget https://s3.ap-northeast-2.amazonaws.com/klaytn-chaindata/baobab/klaytn-baobab-chaindata-20211202011212.tar.gz

# 대상 디렉토리 경로 지정 후 압축 풀기
tar -C ~/kend_home -xvf klaytn-baobab-chaindata-20211202011212.tar.gz
```
***

```shell
# 블록 싱크 맞추기 시작
kend start

kend status
# ⇒ kend is running

# 로그 확인
tail -f /kend_home/logs/kend.out

# rpc, ws 모듈 활성화
curl -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"rpc_modules","params":[],"id":1}' http://localhost:8551
```

## [:sparkles:API 참조문서](https://ko.docs.klaytn.com/bapp/json-rpc/api-references)
### klay_syncing
```shell
# Request
curl -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"klay_syncing","params":[],"id":1}' http://localhost:8551
```
```
# Result
# 싱크 미완료 시
```
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "currentBlock": "0x3e31e",
    "highestBlock": "0x827eef",
    "knownStates": "0x0",
    "pulledStates": "0x0",
    "startingBlock": "0x0"
  }
}
```
```
# 싱크 완료 시
```
```json
  {
    "jsonrpc": "2.0",
    "id": 1,
    "result": false
  }
```
또는 klay.ipc가 있는 디렉토리로 이동 <br>
* attach:
대화형 자바스크립트 환경 시작, 이미 실행 중인 노드에 연결
* console:
대화형 자바스크립트 환경 시작, 노드를 새로 실행시켜 연결

```shell
ken attach klay.ipc

# 자바스크립트 콘솔에 연결된 상태에서
# 현재 나의 EN의 블록 넘버 확인 가능
klay.blockNumber
```
[Klaytnscope](https://baobab.scope.klaytn.com/)에서 block height를 확인해보자.<br>
콘솔의 klay.blockNumber의 결과값과 Klaytnscope의 블록 넘버가 일치한다면,<br>
**⇒ 싱크 완료**
<br>
***
<br>
모든 블록을 다운받아 블록 싱크가 이루어졌다면,
## [:sparkles:빠른 시작](https://ko.docs.klaytn.com/getting-started/quick-start)
### 개발 도구 설치하기 - [vvisp 설치 관련 이슈](https://github.com/HAECHI-LABS/vvisp/issues/194)
truffle을 사용하도록 하자....!!<br>
:thought_balloon:노드 버전 관련 문제인 듯
<br>
*:collision: 컨트랙트 배포 전, **계정이 잠겨있으면 오류가 생긴다** *
<br>
>Running migration: 1_initial_migration.js
>Replacing Migrations...
>... …
>Error: authentication needed: password or unlock

klay.ipc에 attach한 후
```shell
personal.unlockAccount("자신이 생성한 계정 주소")
```

*:collision: 트러플을 사용하여 스마트 컨트랙트를 배포하려 하는데 다음 오류 메시지가 표시됩니다. *
>Error: Returned error: The method net_version does not exist/is not available at Object.ErrorResponse (/usr/local/lib/node_modules/truffle/build/webpack:/~/web3-eth/~/web3-core-helpers/src/errors.js:29:1)...

<br>
```shell
vim ~/ken-linux-amd64/conf/kend.conf
```
>아래 내용으로 net 및 RPC 콘솔을 위한 API 활성화
>RPC_API="admin,debug,klay,miner,net,personal,rpc,txpool,web3"

kend.conf 업데이트 후 Klaytn 노드 재시작!

- [ ] 컨트랙트 배포 관련 추가
- [ ] 생략된 계정 생성 관련 추가
