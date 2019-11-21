---
layout: post
title: "aws-lambda"
date: 2019-11-19
categories: aws
tags: aws lambda
---

AWS lambda를 이용하여 serverless application을 생성할 수 있다. 다른말로, 백엔드 서버를 구축할 필요없이 핸들러 함수만 등록하면 Lambda가 해당 코드를 실행시켜주는 서비스이다. Trigger 시에만 코드를 실행하며 사용한 컴퓨팅 시간에 대해서만 요금을 지불할 수 있다. 다른 AWS 서비스와의 연동이 잘 되있고 이는 UI/cli 상으로 조정가능하다. 다만, Role, Network 설정을 잘 해주어야한다.

[참고](https://aws.amazon.com/ko/lambda/)

실습해보기 위해 간단하게 vpc내 rds instance에 접근해서 간단한 쿼리를 날리는 핸들러를 파이썬으로 작성했다.

지원하는 언어별로 작업 방법이 다르고, 일단 Python을 기준으로 진행한다. 
~~~python
def create_conn():
    conn = None
    try:
        conn = psycopg2.connect(conn_string)
    except psycopg2.DatabaseError as e:
        logger.error(
            "ERROR: Unexpected error: Could not connect to Postgres instance.")
        logger.error(e)
        sys.exit()

    logger.info("SUCCESS: Connection to RDS MySQL instance succeeded")

    return conn


def fetch(conn, query):
    result = []
    logger.info("Now executing: {}".format(query))
    cursor = conn.cursor()
    cursor.execute(query)

    raw = cursor.fetchall()
    for line in raw:
        result.append(line)

    return result


def lambda_handler(event, context):
    query_cmd = "SELECT * FROM PG_TABLES LIMIT 5"
    logger.info(query_cmd)

    conn = create_conn()

    result = fetch(conn, query_cmd)
    conn.close()

    return result
~~~

이때 Postgresql 에 접근하기 위한 드라이버로 psycopg2라는 외부 라이브러리를 import해주어야했기 때문에,
Lambda Runtime에 해당 라이브러리를 따로 첨부해주어야했다. 

따로 첨부해주기 위해서는 소스코드 + 라이브러리 를 한꺼번에 압축하여 업로드하는 방법이 있고, layers라는 걸 활용해서 lambda function들이 사용할 외부 라이브러리를 그때그때 매번 업로드하지 않고 layer만 추가해서 사용할 수 있도록 해놓은 것도 있었는데, 한꺼번에 업로드하는것은 동작을 확인했고, layers를 실습해보고자 하였다. 

* [한꺼번에 압축하는 법](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html)

Layers로 작업할때는 몇 가지만 주의하면 된다. 물론 이 몇가지 때문에 꽤 힘든 시간을 보내긴했다(ㅠㅠ)

1. Lambda의 cpu 기준으로 컴파일된 라이브러리를 포함할것 (ex. \_psycopg.cpython-36m-x86_64-linux-gnu.so)
[람다용으로 컴파일된 라이브러리 레포지토리](https://github.com/jkehler/awslambda-psycopg2)

2. python 폴더 밑에 라이브러리 폴더를 두고 python 폴더를 압축하여 업로드할것.
- *이 구조가 아니면 Lambda에서 import가 안된다!*(이것때문에 시간오래걸림)

ex.
- python
- - library1
- - library2
