# 개발환경
>+ 개발언어/프레임워크: Ruby On Rails
>+ 개발도구: SublimeText3
>+ 운영체제: OSX

# 작동방법
>+ Ruby On Rails가 셋팅된 환경이라면, source를 pull 받은 후에 다음 명령을 수행합니다. 'bin/rails test test/jobs/block_chain_job_test.rb'

# 과제 해결과정
## 블록체인을 이해하기
블록체인에 대해서 아는게 거의 없었습니다. 해결 대상 도메인을 이해하지 못한 상태에서 그 도메인에 관한 문제를 풀기는 어렵다는 생각이 들었기 때문에, 위키, 뉴스, 블록체인 커뮤니티 등을 통해 블록체인이 무엇인지, 블록체인에서 각각의 블록에 저장된 데이터들이 어떤 의미를 갖는지를 파악하려고 했습니다.

## 임의의 블록 해쉬를 선택하여 JSON데이터를 분석하기
1. 특정 블록을 분석하기 위해, 블록해쉬 하나를 임의로 선정하기
>+ (임의로 선정한 블록해쉬: 0000000000000000010d1fecd817e30ac44c3e82453a84b54b31674e8192a4f8)
2. 데이터 분석하기
>+ json데이터 구조(https://blockchain.info/block-index/0000000000000000010d1fecd817e30ac44c3e82453a84b54b31674e8192a4f8?format=json) 와,
블록구조 정보페이지(https://blockchain.info/block/0000000000000000010d1fecd817e30ac44c3e82453a84b54b31674e8192a4f8), 과제 소개에 '블록에 포함된 내용'에 나온 항목들 참고했습니다.
>+ 분석 결과, fee는 해당 블록의 총수수료이고, 트랜잭션리스트는 tx에 배열로 저장되어 있으며, 그 배열에 저장되어있는 각각의 트랜잭션에서 value는 트랜잭션의 값, size는 트랜잭션의 크기라는 것을 파악했습니다. 그리고 tx에는 input과 output데이터가 각각 inputs와 out이라는 키에 저장되어 있는 것을 확인했습니다.

## 첫 번째 과제 수행 : "블록 해쉬 값"을 Argument로 전달 받아서 트랜잭션(tx) 수, 평균 트랜잭션의 값(value), 평균 트랜잭션의 수수료(fee), 평균 트랜잭션의 크기(size)를 출력하라.
1. 테스트케이스 만들기
>+ 테스트케이스를 통해 코드를 짜기 위해서, 먼저 아래 항목들에 대한 정답이 필요했고, 다음과 같은 방법으로 구했습니다.
>+ 트랜잭션(tx)의 수: '블록구조 정보페이지의 Number Of Transactions 값'
>+ 평균 트랜잭션의 값(value) : '블록구조 정보페이지의 Output Total 값'을 '트랜잭션 수'로 나눔
>+ 평균 트랜잭션의 수수료(fee) : 'json data의 fee 값'을 '트랜잭션 수'로 나눔
>+ 평균 트랜잭션의 크기(size) : 'json data의 size 값'을 '트랜잭션 수'로 나눔


2. 트랜잭션(tx)의 수 구하기
>+ json data의 n_tx값을 가져왔습니다.
>+ 결과: 1376

3. 평균 트랜잭션의 값(value) 구하기
>+ value가 각 트랜잭션의 output과 input각각에 있는데, output의 합산으로 '총 트랜잭션의 값'을 계산하였습니다. 
>+ output값은 fee를 제외한 값이고, 첫 번째 트랜잭션인 경우에는 새로 생성된 비트코인이므로 input은 없고 output만 있었기 때문입니다.
>+ 그렇게 나온 '총 트랜잭션의 값 / 트랜잭션의 수'로 평균 트랜잭션 값을 계산했습니다.
>+ 다만, 블록구조 정보페이지의 Number Of Transactions 값으로 나온 '트랜잭션의 총합'이 json data의 값과 단위가 달라 가중치 100,000,000을 곱해서 비교했습니다.
>+ 결과: 272280909.11628 -> 2.7228090911628 BTC

4. 평균 트랜잭션의 수수료(fee) 구하기
>+ 'json data의 fee 값 / 트랜잭션의 수'로 평균 트랜잭션의 수수료를 계산했습니다.
>+ 결과: 110197.07922 -> 0.0011019707922 BTC

5. 평균 트랜잭션의 크기(size) 구하기
>+ json data의 각 트랜잭션의 size키에 들어있는 트랜잭션 크기 값들을 합산하여 총 트랜잭션의 크기을 계산했습니다. 
>+ 그러나, 'json data의 size 값'과 오차가 생겼습니다. 3개 정도의 블록해쉬를 더 분석해본 결과 계속 오차가 생긴다는 것을 파악했지만, 원인이 무엇인지는 알아내지 못했습니다.
>+ 그래서 오차범위를 1kb로두고, 계산값이 오차범위 내이면 맞는 것으로 처리했습니다.
>+ '총 트랜잭션의 크기 / 트랜잭션의 수'로 평균 트랜잭션 크기를 계산했습니다.
>+ 결과: 725.39172 -> 0.72539172 KB

## 두 번째 과제 수행 : "블록 해쉬 값과 input or output"을 Argument로 받아서, input 혹은 output의 정보만 출력하라.
1. 테스트케이스 만들기
>+ 첫번째 과제와 달리, 결과 값이 단순 수치로 나오는 것이 아니라 데이터 묶음으로 나오는 것이기에 어떻게 테스트케이스를 만들어야 할까 고민이 되었습니다.
>+ 나온 결과가 input묶음이냐 output묶음이냐를 구별하고, 정확한 데이터를 가져왔는지 검증할 수 있는 방법을 생각하다가, 세가지 방법을 사용했습니다.
>+ 첫번째 방법은 "input과 output 데이터 각각의 갯수로 구별해보자"였습니다. 그러나 input, output 갯수가 같았으므로, 이것은 데이터를 잘 가져왔는지 아닌지 가볍게 검증하는 기준으로 남겨두고 다른 방법을 더 찾아야 했습니다.
>+ 두번째 방법은 input에만 sequnce값이 있다는 것에서 생각했습니다. 그래서 결과데이터의 요소에 sequnce가 있느냐 없느냐를 input과 output을 구별하는 기준으로 사용했습니다.
>+ 세번째 방법은 output의 첫번째의 addr이 일치하는지 확인하는 것이었습니다.

2. input 혹은 output 정보만 출력하기
>+ input 혹은 output이 각각 여러 개의 값이 있었으므로, 결과 값을 받기위해 result_input_data라는 배열을 선언한후 그 타입(input 혹은 output)에 해당하는 정보들을 저장했습니다.

