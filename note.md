로그를 마이크로 서비스 단위로 각자 남기고 싶다.

REC
Up
CT
AB

import microservice => 테라폼이 마이크로서비스로 만들어준다(cpu 몇개 / 리소스 몇개)

kinesis 
stream
(aws 서비스)
https://aws.amazon.com/ko/kinesis/

kinesis firehose

mls-firehose-flo
=> client_id=flo인것 만 따로 로그를 남긴다

(kafka비슷)
mls-stream



modules
- microservice
kinesis firehose 에서 로그 필터 이런거를 하기 위해 람다를 설정할 수도 있음
=> rec에 맞는 로그를 넣고 이런 후처리

s3 terraform tfstate 에서 시점을 관리함.

s3 에 있는 테라폼의 state 와 실제 state가 다르면 사단이 나는거다 => conflict


dynamodb 
mls-tfstate-stg
업데이트 중이다/
동시에 테라퐆 deploy => 
MR 가 두개가 동시에 

terraform 입문기
http://woowabros.github.io/tools/2019/09/20/terraform.html

 5986  ssh -N -L 5432:mls-rdb.cabsm2hsndfe.ap-northeast-2.rds.amazonaws.com:5432 ec2-user@ec2-52-78-200-57.ap-northeast-2.compute.amazonaws.com -i /Users/nayoon/Downloads/mnodt-dev-key.pem
 5987  ssh -i /Users/nayoon/Downloads/mnodt-dev-key.pem ec2-user@ec2-52-78-200-57.ap-northeast-2.compute.amazonaws.com
 5988  ifconfig
 5989  ssh -i /Users/nayoon/Downloads/mnodt-dev-key.pem ec2-user@ec2-52-78-200-57.ap-northeast-2.compute.amazonaws.com
 5990  chmod 600 ~/Downloads/mnodt-dev-key.pem
 5991  chmod 600 ~/Downloads/mnodt-stg-key.pem
 5992  ssh -i /Users/nayoon/Downloads/mnodt-dev-key.pem ec2-user@ec2-52-78-200-57.ap-northeast-2.compute.amazonaws.com
 5993  exit
 5994  ssh -N -L 5432:mls-rdb.cabsm2hsndfe.ap-northeast-2.rds.amazonaws.com:5432 ec2-user@ec2-52-78-200-57.ap-northeast-2.compute.amazonaws.com -i /Users/nayoon/Downloads/mnodt-dev-key.pem