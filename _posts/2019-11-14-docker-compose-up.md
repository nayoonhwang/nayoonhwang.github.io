---
layout: post
title: "docker-compose up"
date: 2019-11-14
categories: docker
tags: docker
---

[큰 참고가 된 게시물](https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose#ports)

# 개발환경 구성
여러가지 다른 환경에 동일한 개발 환경을 구성할 수 있도록 하기 위해 Docker를 사용한다

# docker-compose 의 필요성
Docker 로 개발환경을 구성할때 예를 들어 Web Application 과 Database 두 개의 서버를 띄워야한다고 하면,
일단 App을 하나의 컨테이너로 띄우고, Database를 또다른 컨테이너로 띄워서 둘이 통신할 수 있도록 해야한다.

그런데 컨테이너와 다른 컨테이너가 통신할 수 있게 하려면 docker run 에서 몇가지 옵션이 더 들어가야한다.

예시. Django application 
~~~bash
docker run -it --rm \
    -p 8000:8000 \
    django-sample \
    ./manage.py runserver 0:8000
~~~

django 에서 사용할 데이터베이스 설정을 담은 settings.py 파일
~~~python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mls-admin-local',
        'USER': 'postgres',
        'HOST': 'db',
        'PORT': '5432',
        'PASSWORD': 'postgres',
    }
}
~~~

ex. Postgresql database
~~~bash
docker run --rm \
    --name db \
    -e POSTGRES_DB=sampleDatabase \
    -e POSTGRES_USER=sampleuser \
    -e POSTGRES_PASSWORD=samplesecret \
	--volume=$(pwd)/docker/data:/var/lib/postgresql/data \
    postgres
~~~
> 이렇게 환경변수를 설정해서 Postgresql 컨테이너를 실행시키면, default 값 대신 해당 값으로 초기화한다.
[참고](https://hub.docker.com/_/postgres). 예를 들어 default database 대신에 djangosample 이라는 database를 create한다. POSTGRES_DB의 경우에 정의되지 않으면 USER와 같은 이름의 database가 만들어진다. *단, 서로 다른 container가 통신을 해야하는 상황에서는 password는 반드시 필요하므로 설정해주어야한다* 

단순하게 이렇게 구성해서는 두 컨테이너는 서로 통신할 수 없고, 
> could not connect to server: Connection refused

이 에러만 발생할 것이다. 

이를 해결하기 위해선
~~~bash
docker run -it --rm \
    -p 8000:8000 \
    --link db \
    -e DJANGO_DB_HOST=db \
    -e DJANGO_DEBUG=True \
    django-sample \
    ./manage.py runserver 0:8000
~~~

이런식으로 --link 옵션을 통해 db 컨테이너와 app 컨테이너를 연결해줘야한다. 

하지만, 이런식으로 하게 되면 조금 불편한 것이 사실이다. 이렇게 순차적으로 컨테이너를 실행시키는 것보단 한번 주루룩 설치하는 것이 아무래도 편할 것이다.

> 게다가 link 옵션은 deprecate 예정이다.

이를 위해서 docker-compose를 이용할 수 있다. 이름에서 떠오르는 composer 와 비슷하게 컨테이너의 관리도구 같은 느낌이다.
docker-compose.yml 파일을 이용하여 컨테이너 실행에 필요한 옵션을 기술할 수 있고, 컨테이너간 실행 순서나 의존성도 관리할 수 있다.

## docker-compose.yml

~~~bash
version: "3"

services:
  db:
    image: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      POSTGRES_DB: sampleDatabase
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
  web:
    build:
      context: .
      dockerfile: Dockerfile
    image: sampleApplication
    stdin_open: true
    tty: true
    command: >
      bash -c "python manage.py migrate --noinput 
      && python manage.py createsu 
      && python manage.py collectstatic --noinput 
      && python manage.py runserver 0.0.0.0:8000"
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on: # depend_on 옵션을 주면 순서를 조정가능하다 (db가 올라간 후에 web이 올라가도록)
      - db
    environment:
      DJANGO_SETTINGS_MODULE: sampleApplication.settings.default
~~~

위에 좀 복잡하게 했던걸 위와 같은 compose 파일을 작성하면 쉽게 동작시킬 수 있다. 

compose 에서는 link같은 것 필요없이도, 서비스 끼리 이름으로 **통신가능**하다. 즉 위 예시에는 web에서 데이터베이스에 접근할때 단순히 다른 서비스인 db로 별다른 조치 없이 접근가능하다. Application 컨테이너가 뜰때 django의 settings.py에 database 설정의 host를 'db'로 해놓았기 때문에 이에 접근을 시도할 것이다. 예를 들어 settings.py 에 database host를 localhost라고 해놓았다면 접속이 안될것이다.

db라는 이름이 마음에 들지 않는다면 또는 compose 내에 'links'라는 옵션을 통해 db 외에 다른 이름을 alias시켜줄 수도 있다. mydatabase라던지...?

첨에 좀 헤맸던건, 자꾸 데이터베이스가 없다느니, role 이 존재하지 않는다느니 하는 문제였는데, 

~~~bash
docker volumn prune
docker ps -a // 도커 컨테이너 모두 조회(stop된것도)
rm -rf /var/lib/postgresql/data/

docker-compose down
~~~
컨테이너를 모두 조회해서 모두 삭제하고 마운트된 볼륨도 모두 삭제한 후 다시해보니 정상적으로 잘 작동했다.

기존 컨테이너와 뭔가가 충돌했던 모양이다. 

이제 실행시켜보면 된다. --force-recreate옵션을 통해 무조건 컨테이너를 다시 만드는 옵션을 줄 수있다.

~~~bash
docker-compose up --force-recreate
~~~