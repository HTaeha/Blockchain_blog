---
description: Bitclout 코드를 실행하고 개발해보자.
---

# Develop Bitclout

## Setup

```bash
cd $WORKING_DIRECTORY
git clone https://github.com/deso-protocol/core.git
git clone https://github.com/deso-protocol/backend.git
git clone https://github.com/deso-protocol/frontend.git
git clone https://github.com/deso-protocol/identity.git
```

### Running the frontend in development mode

```bash
# Assume we're starting in $WORKING_DIRECTORY, which contains all the repos
cd frontend
npm install

# The following command will serve the frontend on localhost:4200 with
# auto-reloading on changes. You must run a node before the site will
# actually work however (see next section).
ng serve
```

#### npm install 시 발생하는 에러

[git@github.com](mailto:git@github.com) Permission denied (public key) error 가 발생했다.

등록되지 않은 기기에서 유저의 ssh가 등록되지 않아 접근권한이 없어서 나오는 문제이다.

[MAC ) git 문제 Permission denied (publickey).](https://zeddios.tistory.com/120)

해당 블로그를 참고하여 해결했다.

ng serve 전에는 angular를 설치해주어야 한다.

#### Install angular

```bash
npm install -g @angular/cli
```

### Running the node in testnet mode

```bash
# Assume we're starting in $WORKING_DIRECTORY, which contains all the
# repos. Also assume we have "ng serve" running.
cd backend/scripts/nodes

# The n0_test script runs a testnet blockchain locally. It starts mining 
# blocks immediately at a much faster rate than mainnet. You can set your 
# public key to receive the block rewards by setting it as --miner-public-key 
# in the arguments. This gives you funds that you can test with. You can see 
# the status of the node by going to the Admin tab after logging in with an
# account and then going to the Network subtab.
./n0_test

# Once n0_test is running, you must navigate to the following URL. 4200 is the
# port for ng serve. Note that in order to be 
<http://localhost:4200>
```

#### script 실행 시 발생하는 에러

```bash
brew install pkg-config
brew install vips
```

```bash
export CGO_CFLAGS_ALLOW=".*"
export CGO_LDFLAGS_ALLOW=".*"
./n0_test
export CGO_CFLAGS_ALLOW=
export CGO_LDFLAGS_ALLOW=
```

script 실행 후 보안상의 이유 때문에 flag를 다시 원래 설정으로 돌려주어야 한다.

#### Port 번호 변경

bitclout frontend는 17001번 포트에 backend API 요청을 한다. 17001은 메인넷 포트이고 18001이 테스트 포트이다. 따라서 test 노드를 돌렸으면 frontend와 연결이 되어 있지 않다. 테스트넷과 연결하기 위해서는 frontend에서 포트 번호를 변경해야 한다.

![](../.gitbook/assets/Untitled.png)

위 사진처럼 lastLocalNodeV2 를 17001에서 18001로 바꾼 후 다시 실행하자.

### Running the node in mainnet mode

```bash
# Assume we're starting in $WORKING_DIRECTORY, which contains all the repos.
# Also assume we have "ng serve" running.
cd backend/scripts/nodes

# The n0 script runs a node that connects to mainnet peers. It will download
# all the blocks from its peers and then start syncing its mempool from them.
# You can see the status of the node by going to the Admin tab after
# logging in with an account and then going to the Network subtab. Note that
# syncing the blockchain may take an hour or so.
$ ./n0

# Once n0 is running, you must navigate to the following URL. 4200 is the
# ng serve port. It should automatically hit your node, which should be
# exposing its API at localhost:17001.
<http://localhost:4200>
```

mainnet은 17001번 포트에서 실행되므로 testnet과 반대로 frontend lastLocalNodeV2를 다시 17001로 돌려주어야 한다.

## Reference

[Setting Up Your Dev Environment](https://docs.deso.org/code/dev-setup)
