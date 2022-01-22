---
layout: post
title: 개발자를 위한 비트코인 해부학
date: 2022-01-21 19:58:00 +0900
cover: /covers/bitcoin.png
disqusId: c8288934cdab2a704d2cf0b8d441d399bd8014bb
toc: true
category: Blockchain
tags:
- bitcoin
- blockchain
---

개발자의 시각에서 정확한 비트코인의 동작 방식을 알아보겠습니다.

<!-- more -->

# 비트코인이란?

### **네트워크로 연결된 여러 노드에서 동일한 송금 장부를 유지하고 작성하는 컴퓨터 프로그램**

# 블록체인이란?

### **송금 장부를 저장하는 방식**

# 1. 비트코인 노드

비트코인 노드는 비트코인 데몬을 실행하고 있고, 비트코인의 모든 송금 장부(=블록체인)을 저장하고 있는 컴퓨터를 의미합니다.
트랜잭션을 생성하고, 이렇게 생성된 트랜잭션을 모아서 Block을 만들고(채굴), 이렇게 만들어진 Block을 검증하고, 승인하는 주체입니다.
따라서 전체 비트코인 네트워크에서 각각의 노드는 어느 Block을 거절하고 승인할 지 결정할 수 있는 민주 시민의 역할을 하고 있다고 볼 수 있습니다.

## 현재 비트코인 블록체인 크기

2022년 1월 기준 비트코인 전체 블록체인의 크기는 **385GB**입니다.
비트코인 데몬을 실행하면 모든 블록체인이 다른 비트코인 노드로 부터 다운로드 됩니다.
[여기](https://www.blockchain.com/charts/blocks-size)에서 실시간 그래프를 볼 수 있습니다.

## 사람들이 노드를 유지하는 이유

이렇게 방대한 양의 블록체인을 가지고 있는 노드를 사람들은 왜 유지하고 있을까요?
노드를 유지하고 있는 사람을 크게 3가지로 나눌 수 있습니다.
([출처](https://river.com/learn/why-should-i-run-a-bitcoin-node/))

#### 1) 송금, 채굴 중개 사업자

송금, 채굴등 모든 비트코인 관련 기능은 노드 밖에 할 수 없습니다.
다르게 말하면 비트코인의 모든 블록체인을 다운로드 받지 않는 이상 송금이나 채굴을 할 수 없다는 것입니다.
비트코인을 송금하려는 모든 사람이 수 백 GB에 달하는 모든 블록체인을 가지고 있기에는 무리가 있기 때문에,
송금을 중개를 해주는 서비스가 존재하고 있고, 이런 서비스들이 노드를 유지하고 있습니다.

#### 2) 비트코인 부자들

하지만 이런 송금 중개를 해주는 서비스들은 기술적으로 사용자의 IP를 알아낼 수 있습니다.
또한 송금을 하기 위해서는 Private Key의 사용이 필수적입니다.
따라서 많은 양의 비트코인을 소유하고 있는 사용자는 해킹의 위협 때문에 믿을 수 없는 송금 중개 서비스의 사용을 꺼릴 수 있습니다.
이들은 독자적인 노드를 유지하여 개인 정보의 보안을 유지하고자 할 것입니다.

#### 3) 열렬한 비트코인 신봉자들

수 백 GB이 데이터는 어찌보면 하드디스크 하나에 다 들어갈 수 있을 만큼 작다고 볼 수 있습니다.
따라서 한 개인이 유지하고 있기에는 그렇게 부담스러운 크기는 아닙니다.
탈중앙화 금융에 매료된 사람들이나 블록체인에 대하여 연구하는 사람들은 범지구적인 블록체인 네트워크의 일원이 되고 싶어할 수도 있습니다.

## DNS Seeds

처음 비트코인 데몬을 실행하면 그동안 쌓여왔던 블록체인을 다른 노드로 부터 다운로드 받으려고 할 것입니다.
그러기 위해서는 인터넷에 존재하는 다른 노드의 IP 주소를 알아야 합니다.
**비트코인은 다른 노드의 IP 주소를 DNS Query를 이용하여 얻어오는데, 이 때 사용하는 DNS를 `DNS Seeds`라고 부릅니다.**
이러한 DNS Seeds는 비트코인 코드에 하드코딩 되어 있으며
[Github 코드](https://github.com/bitcoin/bitcoin/blob/0.21/src/chainparams.cpp#L123-L131)
에서 확인할 수 있습니다.

# 2. 송금 (Transaction)

송금은 특별한 과정이 필요하지 않습니다.
그냥 철수가 영희에게 50 비트코인을 송금한다고 다른 노드에게 알려주고,
그 정보를 수신한 어떤 노드가 송금 정보(=Transaction)를 포함하고 있는 Block을 만들어주기만 하면 됩니다.

## Memory Pool

각 노드들은 네트워크에 연결되어 있는 다른 노드로 부터 트랜잭션 정보를 전달 받습니다.
이렇게 전달 받았지만 아직 Block으로 생성되지 않은 **트랜잭션을 임시로 저장해두는 장소를 `Memory Pool`이라고 합니다.**
실시간 Memory Pool을 [이 사이트](https://mempool.space/)에서 볼 수 있습니다.

## 트랜잭션의 승인 (Confirmation)

Memory Pool에 있는 트랜잭션은 아직 완료된 것이 아닙니다.
즉, 아직 비트코인이 완전히 상대방에게 넘어간 것이 아닙니다.
이 트랜잭션이 포함된 Block이 생성되어야 합니다.
**이 트랜잭션이 포함된 Block이 생성되면 한 번 승인 되었다라고 표현합니다.**
그 뒤에 Block이 하나 더 연결되면 두 번, 그 뒤에 Block이 하나 더 연결 되면 세 번, 이런식으로 우리는 어떤 트랜잭션이 몇 번 승인 되었다고 말할 수 있습니다.

금액에 따라 송금이 완료되었다고 판단할 만한 승인 횟수는 다릅니다. 적은 금액은 한 두번이면 충분하지만, 많은 금액은 6번은 필요합니다.
어떤 금액에 몇 번 승인이 필요한지는 각 노드마다 다르게 판단합니다. 

## 비트코인 계좌 (지갑)

비트코인 데몬은 따로 계좌 목록 데이터를 가지고 있지 않습니다. 때문에 명시적으로 계좌를 만들어서 등록하는 과정이 없습니다.
그냥 랜덤으로 생성한 문자열을 하나 만들고 이 문자열에 누군가 송금해주면 해당 계좌가 세상에 알려지는 겁니다.
그리고 비대칭 암호화를 이용한 디지털 서명 기술을 이용하여 해당 계좌의 돈을 함부로 인출할 수 없게 해놨습니다.

비트코인은 랜덤한 32byte 값(Private Key)을 생성한 다음,
`Secp256k1` 라는 타원곡선 암호(Elliptic curve cryptography)를 이용하여 Public Key를 생성하는 방식을 사용합니다.
**여기서 생성한 Public Key가 계좌 번호이고 Private Key가 계좌 비밀번호입니다.**

## UTXO

![트랜잭션 시각화. 여기서 파란색 박스가 UTXO이다.](./utxo.png)

비트코인은 '계좌 잔액' 이란 개념이 없습니다.
'다 쓴 금액'과 '아직 안 쓴 금액'만 있을 뿐입니다.
위 그림에서 빨간색이 '다 쓴 금액' 파란색이 '아직 안 쓴 금액'을 의미합니다.
**UTXO는 아직 안 쓴 금액**을 의미합니다.

위 그림과 같이 트랜잭션에서 보내는 금액(Input)은 정확히 예전에 받았던 금액(Output)과 일치합니다.
각 Input은 해당 금액을 받았던 트랜잭션의 ID를 가지고 있어서 돈의 출처를 알 수 있습니다.

위 그림을 보면 알 수 있듯이 어떤 금액을 보내기 위해서는 그보다 많은 총액을 가지고 있는 Input들을 사용합니다.
이 트랜잭션에서 사용된 모든 Input들은 '다 쓴 금액' 처리가 되고, 보내고 남은 금액은 또 다른 Output으로 생성됩니다.

비트코인 데몬은 어떤 계좌가 어떤 UTXO를 가지고 있는지 인덱싱 해둔 데이터를 가지고 있습니다. 이 것을 `UTXO Set`라고 부릅니다.
트랜잭션을 빠르게 검증하기 위해서 사용합니다.
UTXO Set은 비트코인 데몬을 처음 실행하면 다른 노드로부터 블록체인 데이터를 받을 때 생성합니다.

2022년 1월 기준 UTXO 수는 8천만개에 달하고, UTXO Set의 데이터 양은 4.5GB 정도 합니다.
[여기](https://statoshi.info/d/000000009/unspent-transaction-output-set?orgId=1&from=now-1y&to=now&refresh=10m)에서 UTXO의 수와 UTXO Set의 데이터 양을 실시간 그래프로 볼 수 있습니다.

# 3. Block에 담긴 데이터

![최초 Block에 담긴 데이터](./genesis-block.png)

최초 Block의 데이터를 살펴보면서 비트코인 Block에는 정확히 어떤 데이터가 담겨있나 알아보겠습니다.
[여기](https://blockchain.info/block-height/0)
나
[여기](https://www.blockchain.com/btc/block/000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f)
를 보면 비트코인 최초 Block에 어떤 데이터가 담겨있는지 자세히 알 수 있습니다.

## 각 값의 이름과 의미

| 이름 | 값 |
|---|---|
| Hash | 각 Block을 식별할 수 있는 고유 값. 모든 Block은 서로 다른 Hash를 가지고 있습니다. |
| Confirmations | 이 Block 이후에 블록체인에 생성된 Block의 수 |
| Timestamp | Block이 생성된 시간 |
| Height | 이 Block이 몇 번째 Block인지를 나타내는 값 |
| Difficulty | [밑에서 자세히 설명](#Difficulty) |
| Merkle root | [밑에서 자세히 설명](#Merkle-Root) |
| Version | 이 Block의 버전. 버전에 따라 Block에는 다른 형식의 데이터가 담겨 있습니다. |
| Bits | [밑에서 자세히 설명](#Target) |
| Weight | 2017년에 새롭게 등장한 값으로 Block의 크기를 의미. 추후에 자세히 설명 |
| Size | Block의 바이트 크기 |
| Nonce | [밑에서 자세히 설명](#Nonce) |
| Transaction Volume | 트랜잭션의 총 비트코인 양 |
| Block Reward | 이 Block을 채굴한 노드에게 주어진 보상 |
| Fee Reward | 모든 트랜잭션의 수수료 합 |

## Merkle Root

![Merkle Tree](https://github.com/bitcoinbook/bitcoinbook/raw/a3229bbbc0c929dc53ec11365051a6782695cb52/images/mbc2_0904.png)

Merkle Root를 알기 위해서는 먼저 Merkle Tree를 알아야합니다.

Merkle Tree는 Binary Hash Tree라고도 불리웁니다. 방대한 데이터의 무결성을 빠르게 검증하기 위해서 생겨난 자료 구조입니다.
Merkle Tree의 Leaf는 어떤 한 데이터의 Hash 값이고, 각 부모 노드들은 두 자식 노드를 더한 값의 Hash 값입니다.
이런식으로 **각 노드를 합치는 것을 반복한 후 남은 하나의 Hash값을 `Merkle Root`라고 부릅니다.**

비트코인에서는 한 Block 안에 있는 모든 트랜잭션을 요약하기 위한 용도로 쓰입니다.
**비트코인에서 Merkle Tree의 Leaf 값들은 특정 트랜잭션을 두 번 SHA256을 한 값입니다.**
하나의 트랜잭션을 조금만 수정해도 Merkle Root는 크게 달라지므로, Merkle Root 값을 가지고 Block 안에 있는 모든 트랜잭션의 무결성을 보장할 수 있습니다.

![Merkle Path](https://github.com/bitcoinbook/bitcoinbook/raw/a3229bbbc0c929dc53ec11365051a6782695cb52/images/mbc2_0905.png)

그냥 모든 Transaction을 한 번에 Hash하면 되지 왜 이런 방식을 쓸까요?
만약에 그런 방식을 사용한다면 특정 트랜잭션을 검증하기 위해서 같은 Block에 있는 다른 모든 트랜잭션의 데이터가 필요합니다.
하지만 Merkle Tree를 사용한다면 위 그림과 같이 몇 개의 Hash 값만 가지고 손 쉽게 특정 트랜잭션을 검증할 수 있게 됩니다.

[출처1](https://www.gemini.com/cryptopedia/merkle-tree-blockchain-merkle-root#section-merkle-roots-in-blockchain-why),
[출처2](https://www.oreilly.com/library/view/mastering-bitcoin/9781491902639/ch07.html#merkle_trees)

# 4. Block 생성 (채굴)

Memory Pool에서 Block으로 만들기를 원하는 트랜잭션을 가지고
적절한 Hash 값만 찾아내면 바로 Block을 생성해 낼 수 있습니다.
Block은 그냥 각 값들을 열거해 놓은 데이터 모음에 불과하기 때문입니다.
하지만 이 Hash 값을 생성해 내는 것이 어렵습니다.

## Block Hash 생성 방법

위에서 언급된 바와 같이 Block Hash는 무작위로 생성된 값이 아닙니다.
아래의 6개의 값을 바탕으로 생성합니다.

| 이름 | 설명  | 크기 |
|---|---|---|
| Version | Block의 버전 | 4 바이트 |
| PrevBlockHash | 이전 Block의 해시 값  | 32 바이트 |
| MerkleRoot | [Merkle Root 값](#Merkle-Root) | 32 바이트 |
| Time | 1970-01-01T00:00 UTC 부터 지금까지 지난 시간 (단위: 초) | 4 바이트 |
| Bits | Block 생성 난이도를 조절하는 값으로 [밑에서 자세히 설명](#Target) | 4 바이트 |
| Nonce | 조건을 Hash값을 생성하기 위해 조절되는 값으로 [밑에서 자세히 설명](#Nonce) | 4 바이트 |

이렇게 총 80바이트를 두 번 SHA256 해서 생성해 냅니다.

> hash = SHA256(SHA256(Version + PrevBlockHash + MerkleRoot + Time + Bits + Nonce))

## Target

그런데 비트코인은 정말 특이한 규칙을 가지고 있습니다.
**Block Hash는 특정 target 값 보다 작은 값을 가지고 있어야 한다**는 것입니다.

예를 들어 target이 `0x010000` 인 경우 `0x012345`은 target 보다 큰 값이므로 Hash로 사용할 수 없고
`0x001234`은 target 보다 작은 값이므로 사용할 수 있습니다.

이 target 값은 Block Hash를 만들 때 쓰이는 값 중에 하나인 `Bits` 값을 통해 다음 규칙에 따라 만들어 집니다.

> x = Bits의 1번째 바이트    
> y = Bits의 2 ~ 4번째 바이트    
> target = y * (1 << (8 * (x - 3)))    
> 
> ex)    
> Bits = 0x12345678    
> x = 0x12    
> y = 0x345678    
> target = 0x0000000000000000000000000000345678000000000000000000000000000000    

이렇게 계산된 target 값 보다 Block Hash 값이 작아야만 합니다.

## Nonce

이렇게 특정 target 값보다 Block Hash 값을 작게 만들어 주기 위해서 Nonce 값을 바꿔가며 일일히 Block Hash를 계산합니다.
Nonce 값은 4바이트 이므로 총 4,294,967,296 개의 후보가 있습니다.

#### **이런 조건을 만족하는 Nonce를 찾는 것이 바로 채굴입니다.**

#### **사실 채굴은 어려운 수학문제를 푼다기 보다는 많은 복권을 긁는 행위에 더 가까운 것입니다!**

따라서 복잡한 단일 연산을 빠르게 처리하는 CPU보다 간단한 연산을 병렬로 처리하는 것에 특화된 GPU가 채굴 부품으로써 각광을 받게 된 것입니다.

Nonce의 모든 후보를 체크해봐도 target 보다 작은 Hash 값을 찾을 수 없을 수도 있습니다.
이런 경우에는 Block 생성 시간을 의미하는 `Time` 값을 변경하거나, 트랜잭션 몇 개를 memory pool에서 바꿔서 `MerkleRoot` 값을 변경해가며
다시 조건을 만족하는 Nonce 값을 찾습니다.

이 [gist 코드](https://gist.github.com/turunut/7857bd34bac37a04a91a91ee9ea33520)는 이러한 Hash 생성 방법을 Python3으로 작성한 것입니다.

[출처](https://en.bitcoin.it/wiki/Block_hashing_algorithm)

## Hash Rate

![비트코인 네트워크의 해시 레이트 추이](bitcoin-hash-rate.png)

해시 레이트는 1초에 얼마나 많은 해시 연산을 할 수 있는가에 대한 척도입니다.
`1GH/s` 는 1초에 10억 번 해시 연산을 할 수 있다는 의미입니다.
보통 이 단위로 채굴 기기들의 성능을 다룹니다.
[여기](https://www.quora.com/How-good-is-the-Nvidia-RTX-3090-for-mining-Bitcoin)에 따르면 RTX3090의 비트코인 해시레이트는 4.85 GH/s 라고 합니다.

위 그림은 비트코인 네트워크에 연결된 모든 노드의 해시 레이트 추이를 나타냅니다.
2022년 1월 현재 전체 해시 레이트는 **200EH/s**입니다.
1초에 200,000,000,000,000,000,000 번 해시 연산을 할 수 있다는 뜻입니다!

[여기](https://www.blockchain.com/charts/hash-rate)에서 비트코인 전체 해시 레이트를 실시간으로 볼 수 있습니다.

## Difficulty

![Bitcoin Difficulty 추이](./bitcoin-difficulty.png)

Difficulty는 얼마나 Block 생성이 어려운지, 즉, 조건을 만족하는 Hash를 찾는 것이 얼마나 어려운지를 측정하는 척도입니다.
아래와 같이 계산합니다.

> Difficulty = 최초 Block의 target 값 / 현재 Block의 target 값

최초 Block의 Bits 값은 `0x1D00FFFF` 이고, 2022년 1월 기준 현재 Bits 값은 `0x170B8C8B` 이므로, 현재 Difficulty는 `24,371,874,614,345` 입니다!
**비트코인이 처음 생겼을 때 보다 Block을 생성하는 것이 24조배 어려워졌다**는 것을 의미합니다. 

비트코인은 Block을 10분에 한 번 생성되는 것을 지향하기 때문에
이 Difficulty 값을 조절해서 Block 생성 간격을 일정하게 만듭니다.
Miner가 많이 몰려서 Block이 10분 보다 빨리 생성되면 Difficulty를 올립니다. 그리고 그 반대의 경우도 마찬가지 입니다.
하지만 Difficulty가 1보다는 작아지지 않게 설정되어 있습니다.

Difficulty 값은 매 2016 Block마다 조절됩니다.
2016이라는 숫자는 2주 동안 생성되는 Block의 수 입니다.
Difficulty 값을 `2주 / 직전 2016개의 Block이 생성되는데 든 총 시간` 으로 조정하고 Difficulty 값을 통해 다음 2016개의 Block의 target 값을 정합니다.

[여기](https://btc.com/stats/diff)에서 현재 Difficulty 값을 실시간으로 볼 수 있습니다.

[출처1](https://en.bitcoin.it/wiki/Difficulty),
[출처2](https://www.investopedia.com/terms/d/difficulty-cryptocurrencies.asp),
[출처3](https://bitcoin.stackexchange.com/questions/5838/how-is-difficulty-calculated)

## Mining Pool의 등장

![Mining Pool 랭킹](./mining-pool-ranking.png)

남들보다 더 빠르게 조건을 만족하는 Hash 값을 찾아야 채굴 보상을 받을 수 있는 비트코인 특성상
가능한 많은 컴퓨팅 자원을 가지고 있는 사람이 유리합니다.
그래서 사람들은 힘을 합쳐서 Nonce 값을 구간 별로 나눠서 조건에 맞는 Hash를 찾습니다.

보통 채굴한다는 사람들은 전부 이런 Mining Pool에서 Nonce 값 범위를 지정 받아서 조건에 맞는 Hash를 찾는 것입니다.
따라서 모든 Blockchain을 다운로드 받을 필요도 없고, 휴대폰이나 웹 브라우저로도 채굴이 가능한 것입니다.

무조건 강력한 컴퓨팅 자원을 가지고 있는 Mining Pool만 채굴 보상을 독점하는 것도 아닙니다.
채굴이라는 행위 자체가 복권을 긁는 것과 같아서 약소한 Mining Pool도 우연히 강력한 Mining Pool 보다 조건에 맞는 Hash를 빨리 찾을 수 있기 때문입니다.

각 Mining Pool 마다 채굴 보상을 나누는 규칙이 다릅니다.
자세한 정보는 [이 글](https://www.buybitcoinworldwide.com/mining/pools/)을 읽어 주세요.

[여기](https://btc.com/stats/pool)에서 실시간으로 각 Mining Pool의 성능과 점유율을 확인할 수 있습니다.

## 현재 채굴 수익성

![2022년 1월 기준 비트코인 채굴 수익성](./mining-profit.png)

현재 그래픽카드 최고 라인업 중에 하나인 RTX3090을 1000대 샀다고 가정해 봅시다.
[여기](https://www.quora.com/How-good-is-the-Nvidia-RTX-3090-for-mining-Bitcoin)에 따르면 RTX3090의 비트코인 해시레이트는 4.85 GH/s 라고 합니다.
따라서 1000대의 총 해시레이트는 4.85 TH/s 이고, 이 값을 이 [사이트](https://www.cryptocompare.com/mining/calculator/btc)에 입력하면 예상 수익을 계산할 수 있습니다.

위 사진에서 볼 수 있듯이
**전기세 무료에, Mining Pool 수수료를 전혀 내지 않는다고 가정해도 한 달에 31달러밖에 벌 수 없습니다** (2022년 1월 기준).
비트코인의 채굴 난이도는 시간이 지날 수록 점점 높아지니, 비트코인의 가격이 많이 오르지 않는 이상 이 금액은 점점 줄어들겁니다.

## 반감기

![연도별 발행되는 비트코인 수](./bitcoin-halving.png)

비트코인은 **21만개의 Block이 생성될 때 마다 채굴 보상이 반으로 줄어들게** 설계되어 있습니다.
이걸 시간으로 환산하면 대략 4년정도 입니다.
처음에는 채굴 보상이 `50 BTC`이었지만 지금은 3번의 반감기를 거쳐 `6.25 BTC`로 줄어들었습니다.
주기적으로 채굴 보상이 줄어드니 시간이 아무리 지나도 시중에 2100만개 이상의 비트코인이 풀리지 않을 것을 알 수 있습니다.
2022년 1월 현재 약 1900만개의 비트코인이 시중에 존재하고 있습니다.

# 5. Block File Format

[출처1](https://www.arcblock.io/blog/en/post/2018/08/16/index-bitcoin),
[출처2](https://wiki.bitcoinsv.io/index.php/Genesis_block),
[출처3](https://en.bitcoin.it/wiki/Genesis_block),
[Github](https://github.com/bitcoin/bitcoin/blob/master/src/primitives/block.h)

![Block 파일 포맷](./bitcoin-block-file-format.png)

비트코인 최초 Block은 파일에 정확히 아래와 같이 저장되어 있습니다.

```
F9 BE B4 D9 1D 01 00 00  01 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00  00 00 00 00 3B A3 ED FD
7A 7B 12 B2 7A C7 2C 3E  67 76 8F 61 7F C8 1B C3
88 8A 51 32 3A 9F B8 AA  4B 1E 5E 4A 29 AB 5F 49
FF FF 00 1D 1D AC 2B 7C  01 01 00 00 00 01 00 00
00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00  00 00 00 00 00 00 FF FF
FF FF 4D 04 FF FF 00 1D  01 04 45 54 68 65 20 54
69 6D 65 73 20 30 33 2F  4A 61 6E 2F 32 30 30 39
20 43 68 61 6E 63 65 6C  6C 6F 72 20 6F 6E 20 62
72 69 6E 6B 20 6F 66 20  73 65 63 6F 6E 64 20 62
61 69 6C 6F 75 74 20 66  6F 72 20 62 61 6E 6B 73
FF FF FF FF 01 00 F2 05  2A 01 00 00 00 43 41 04
67 8A FD B0 FE 55 48 27  19 67 F1 A6 71 30 B7 10
5C D6 A8 28 E0 39 09 A6  79 62 E0 EA 1F 61 DE B6
49 F6 BC 3F 4C EF 38 C4  F3 55 04 E5 1E C1 12 DE
5C 38 4D F7 BA 0B 8D 57  8A 4C 70 2B 6B F1 1D 5F
AC 00 00 00 00 F9 BE B4  D9
```

- 비트코인 Block은 Script 필드를 제외한 모든 필드가 Little-Endian 으로 저장되어 있습니다.
- variable bytes는 다음 규칙으로 값을 읽습니다
  1. 첫 바이트가 253보다 작을 경우, 그냥 그 값을 읽습니다.
  1. 첫 바이트가 253일 경우, 그 다음 바이트 값을 읽습니다.
  1. 첫 바이트가 254일 경우, 그 다음 4바이트 값을 읽습니다.
  1. 첫 바이트가 255일 경우, 그 다음 8바이트 값을 읽습니다.

이를 해석하면 다음과 같습니다

![비트코인 최초 블럭](./bitcoin-genesis-block.png)

### Magic Number

`0xD9B4BEF9` 으로 하드코딩 되어 있습니다. 테스트 네트워크의 경우에는 `0xDAB5BFFA` 입니다.
[Github 코드](https://github.com/bitcoin/bitcoin/blob/0.21/src/chainparams.cpp#L104-L107)에서 확인할 수 있습니다.
Block을 파일 시스템에 저장할 때, 한 파일에 한 Block을 저장하는 것이 아니라 여러 Block을 저장하기 때문에 각 Block을 구분하기 위하여 고안되었습니다.
일반적으로 한 파일이 128MB가 넘어가면 다음 Block 부터는 다른 파일에 저장합니다.

[출처1](https://bitcoin.stackexchange.com/questions/2337/how-was-the-magic-network-id-value-chosen),
[출처2](https://bitcoin.stackexchange.com/questions/43189/what-is-the-magic-number-used-in-the-block-structure)

### Lock Time

해당 트랜잭션이 언제부터 valid 한지를 나타내는 필드입니다.
**즉 예약 송금을 할 수 있다는 의미입니다.**
이 시기가 되기 전까지는 해당 트랜잭션은 Memory Pool에만 있고, 새로 채굴되는 블럭에 포함될 수 없습니다.

| 값 | 의미 |
|---|---|
| 0 | 즉시 송금 |
| < 500,000,000 | 이 숫자 만큼의 블록이 쌓인 후 송금 |
| >= 500,000,000 | 해당 날짜가 지난 후 송금. 1970-01-01T00:00 UTC 부터 지금까지 지난 시간(초)를 의미합니다. |

[출처](https://learnmeabitcoin.com/technical/locktime)

### Sequence

예약 송금을 한 후, 송금 금액을 변경하고 싶을 수 있습니다.
그럴 때는 원래 트랜잭션보다 더 높은 Sequnce 값을 가지고 있는 새로운 트랜잭션을 다른 노드에 전파하면 **기존의 예약된 트랜잭션을 덮어쓸 수 있습니다.**

일반적으로 예약 송금을 하지 않을 때는 Lock Time 값을 0으로, Sequnce 값을 MAX_INT 값으로 지정해 놓습니다.

추가적인 수수료 없이 다른 노드에 전파된 트랜잭션을 마음껏 수정할 수 있다 보니,
악의적인 사용자가 이 기능을 이용하여 막대한 트래픽을 발생시켜서 비트코인 네트워크를 마비시킬 수 있습니다.
따라서 중간에 기능을 막아두었다가, 트랜잭션을 수정할 때 수수료를 더 내야하는 구조로 변경되었습니다.
이를 `Opt-in RBF` 라고 합니다 더 자세한 정보는
[이 글](https://bitcoin.stackexchange.com/questions/87372/what-does-the-sequence-in-a-transaction-input-mean)과
[이 글](https://bitcoincore.org/en/faq/optin_rbf/#who-invented-unconfirmed-transaction-replacement-does-it-go-against-the-vision-of-bitcoin)을 읽어주세요

# Appendix

- [사토시 나카모토가 쓴 비트코인 논문 링크](https://bitcoin.org/bitcoin.pdf)
- [Mastering Bitcoin - Andreas M. Antonopoulos (O'Reilly)](https://www.oreilly.com/library/view/mastering-bitcoin/9781491902639/index.html)
- [Mastering Bitcoin - Andreas M. Antonopoulos (Github)](https://github.com/bitcoinbook/bitcoinbook)
