---
layout: post
title:  "gorm-postgresql 사용 주의"
date:   2019-12-20 10:45:22 +0900
categories: gorm postgresql
---

gorm 을 통해 go 에서 orm을 사용하고 있었다. gorm은 mysql, postgresql 등 메인 dbms를 지원하면서도 꽤 편리하게 데이터베이스 operation들을 관리할 수 있었기 때문에 잘 사용하고 있었다. 

계속 조회만 했었어서 Update 로직을 딱히 사용할 일이 없다가, 이번에 한번 사용할 일이 있어서 사용해보았다.
~~~go
rdb.Debug().Table("test").Where("user_id = $1", user.UserID).UpdateColumn("test_column", postgres.Jsonb{RawMessage: resBytes})
~~~

뭐 이런 코드였는데, 자꾸 에러가 발생하길래 뭐지싶어서 디버그 옵션을 키고 돌려봤다.
~~~bash
pq: column "test_column" is of type jsonb but expression is of type text
UPDATE "testcases" SET "test_column" = '[123 34 99 111 ~ ]' WHERE (user_id = '[123 34 99 111 ~ ]'
~~~
쿼리가 저렇게 찍혀있었다. 즉, 앞에 있는 where 절의 user_id 에 test_column 에 넣을 데이터가 덮어쓰기 되어있었다. 

~~~go
rdb.Debug().Table("test").UpdateColumn("test_column", postgres.Jsonb{RawMessage: resBytes}).Where("user_id = $1", user.UserID)
~~~
혹시 몰라서 where 절의 위치를 바꿔봤는데 아예 where 절이 무시된다.

http://gorm.io/docs/update.html 공식 documentation에서
~~~go
db.Table("users").Where("id IN (?)", []int{10, 11}).Updates(map[string]interface{}{"name": "hello", "age": 18})
//// UPDATE users SET name='hello', age=18 WHERE id IN (10, 11);
~~~
위 코드가 정상 실행되는 걸 보니 mysql에서는 위와 같은 방식의 update가 정상적으로 실행되는 듯하다. 아마도 postgresql 의 placeholder 관련되서 버그가 있어서 안먹는것으로 추정된다.

Table에 Operation을 날리는게 아니라 Model 기준으로 Update을 수행하게 되면 정상적으로 동작하는 것을 확인했다.
~~~go
rdb.Model(&user).UpdateColumn("test_column", postgres.Jsonb{RawMessage: resBytes})
~~~

Model은 gorm에서는 특정 Row를 가르킨다고 보면 된다. 따라서 Model 의 파라미터로 들어가는 user 객체에는 Id 컬럼이 반드시 채워져있어야한다. 

~~~go
type User struct {
	ID string `gorm:"id"`
	Name string `gorm:"name"`
	TestColumn postgres.Jsonb `gorm:"test_column"`
	IsEnabled bool `gorm:"is_enabled"`
}

var users []User
if err := rdb.Where("is_enabled = $1", true).Find(&users).Error; err != nil{
	return err
}
~~~

뭐 이런식으로 하면 user들을 query로 땡겨와서 slice 안에 넣어놓고 각 user의 특정 컬럼값을 Update하는 로직을 작성할 수 있을 것이다. 
그동안은 Table 에 직접 Operation하는 것이 좀 더 명시적이라고 생각해서 그런식으로 코드를 작성했었는데, 위와 같이 Model기준으로 하는게 코드상 버그도 없고 공식 Documentation이나 gorm usage들에서 많이 사용하는 방식인 듯하다.

물론 Explict하게도 해결할 수 있다. 무식하지만 쿼리를 그냥 때리는 것이다. 
~~~go
rdb.Exec("UPDATE test SET test_column=$1 WHERE user_id=$2", postgres.Jsonb{RawMessage: resBytes}, user.UserID).Error
~~~
쿼리를 그냥 execute(실행)하면 사실 위와같은 문제는 해결 할 수 있다.

Model 기준으로 많이 사용하고 있으니, 한번 Model기준으로 앞으로 작성해보려고한다.