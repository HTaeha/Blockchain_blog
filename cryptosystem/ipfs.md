# IPFS

## 현재의 웹

모든 정보들이 중앙 집중화 되어 있다.

중앙 서버가 다운되면 사용자들은 정보를 얻을 수 없게 된다. 또한 몇몇 서버(구글, 유튜브 등)에 정보들이 모아져있기 때문에 정부측에서 검열하기 쉽다.

ex) 터키 정부는 “국가 안보에 대한 위협”을 이유로 위키피디아 접속을 차단했다.

이러한 단점들이 있음에도 웹이 집중화되어 있는 이유는 우리가 데이터에 빠르게 접근하고 좋은 퀄리티를 원하기 때문이다. 회사들은 집중화된 서버에서 성능을 조절할 수 있다.

## Inter Planetary File System

행성간 파일 전송 시스템 IPFS는 좋은 성능을 유지하면서 웹을 분산화한다.

### 웹 데이터 접근 (Location based addressing)

![](<../.gitbook/assets/Untitled (10).png>)

1. 어떤 장소에서 사진을 받을 수 있는지 알려줘야 한다. (IP or domain address)
2. 1번의 내용은 위치 기반 주소 지정이다.
   1. 이 장소가 접속이 안 되는 상태이면 사진을 다운 받을 수 없다.
   2. 나 이전에 다른 사람들이 사진을 받아가서 사본이 있더라도 사본의 주소를 모르기 때문에 데이터를 받을 수 없다.

### Content based addressing

어디에서 사진을 찾을 지 말해주는 대신 어떤 사진을 찾을지 말해주는 방식이다.

모든 파일은 고유의 해시값을 가지고 있다.

1. 어떤 사진을 받고 싶을 때 그 사진의 해시값을 누가 가지고 있는지 찾는다.
2. IPFS 네트워크에 있는 누군가가 그 사진을 제공해준다.

#### 해시값 사용의 이점

1. 해시값이 일치하는 자료만 받기 때문에 보안성을 확보할 수 있다. 해시값을 체크해서 데이터가 변질되지 않았는지 확인할 수 있다.
2. 네트워크의 효율성
   * 해시값을 사용하기 때문에 여러 사람이 같은 자료를 업로드해도 중복을 방지할 수 있다.

### 데이터 저장

![](<../.gitbook/assets/Untitled (6).png>)

데이터들은 IPFS Object에 저장된다.

최대 256kb까지 저장할 수 있다.

다른 IPFS Object를 link할 수 있다.

![](<../.gitbook/assets/Untitled (9).png>)

사진이나 동영상 같이 크기가 큰 데이터는 256kb씩 여러개로 쪼개져 IPFS Object에 담기게 된다. 그리고 쪼개진 Object들을 연결하는 Links만 존재하는 IPFS Object를 하나 생성한다.

![](<../.gitbook/assets/Untitled (2).png>)

폴더 형식으로 파일들을 관리하는 것도 가능하다.

### 특징

IPFS는 컨텐츠 기반 주소지정을 사용하기 때문에 한번 지정하면 변경이 불가능하다. 블록체인과 같은 불변성을 지니고 있다.

#### Versioning

![](<../.gitbook/assets/Untitled (7).png>)

파일에 대해 변경을 하고 싶을 때 versioning 기능을 이용할 수 있다. IPFS로 데이터를 공유할 때 Commit 객체가 생성된다. 이것은 commit에 대한 내용과 IPFS object만 연결지어져 있다. 특정 파일에 대해서 새로 commit 을 하면 새로운 commit 객체가 생성되어 versioning할 수 있고 이것 또한 모두에게 공유된다.

### 문제점

#### IPFS 네트워크에서 어떻게 해당 파일을 계속 유지할 것인가?

데이터를 받은 사람들이 해당 파일의 캐시를 유지해줌으로써 사람들은 데이터를 받을 수 있다. 하지만 그 캐시들이 모두 사라진다면 데이터를 받을 수 없게 된다. Seeder가 없는 토렌트와 비슷한 상태이다.

이 문제를 해결하기 위해 2가지 방안이 있다.

1. Incentivize nodes
   * 인센티브를 지불하는 방안
2. Proactively distribute files
   * 좀 더 적극적으로 파일들을 배포해서 네트워크상에 항상 특정수 이상의 사본들이 존재하도록 하는 방안

## Filecoin

IPFS의 문제점을 해결하기 위해 IPFS 와 같은 그룹에서 만들어졌다. 저장공간의 분산화를 목적으로 만들어진 IPFS 기반 블록체인이다.

하드디스크에 여분의 공간이 있다면 이 공간을 다른이들에게 임대하여 수익을 올릴 수 있게 한다. 노드들에게 인센티브로 지불하기 위해 filecoin이 만들어졌다.

![](<../.gitbook/assets/Untitled (13) (1).png>)

파일코인 시스템은 파일들을 다수의 노드들에게 복제시켜 다운로드가 가능하게 만든다.

## 활용 방안

IPFS는 터커 정부가 위키피디아를 차단했을 때 위키피디아 터키 버전을 IPFS에 올려 공유했다.

Dtube는 유튜브와 거의 비슷하지만 IPFS를 이용하여 완전하게 분산화 되어 있다.

![](<../.gitbook/assets/Untitled (12) (1).png>)

화성과 지구 사이에 신호를 보내기까지 걸리는 시간은 4분에서 24분이 걸린다. 만약 화성에서 지구 위키피디아 정보를 요청한다면 8분에서 48분의 시간이 걸릴 것이다. 하지만 화성에서 IPFS로 누군가 데이터를 받은 적이 있다면 화성에 서버를 설치하지 않아도 빠르게 화성에서도 데이터 공유를 할 수 있다.

![](<../.gitbook/assets/Untitled (11).png>)

IPFS는 탈중앙화된 인터넷을 만들기 위해 발전하고 있다.

## Reference

{% embed url="https://www.youtube.com/watch?t=15s&v=5Uj6uR3fp-U" %}
