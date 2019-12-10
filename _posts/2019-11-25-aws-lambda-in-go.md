---
layout: post
title: "aws-lambda-in-go"
date: 2019-11-25
categories: aws
tags: aws lambda golang
---

~~~go
func Handler(ctx context.Context) error {
	file, err := os.Open("config.json")
	if err != nil{
		return err
	}
	defer file.Close()

	conf := Configuration{}
	err = json.NewDecoder(file).Decode(&conf)
	if err != nil {
		return err
	}

	// Initialize connection string
	host, user, password, database := conf.Host, conf.User, conf.Password, conf.Database

	var connectionString = fmt.Sprintf("host=%s user=%s password=%s dbname=%s", host, user, password, database)

	db, err := sql.Open("postgres", connectionString)
	if err != nil {
		return err
	}
	defer db.Close()

	// TODO: 유저 프로파일 찌르기
	rows, err := db.Query("select * from experiments limit 5")
	if err != nil {
		return err
	}
	defer rows.Close()

	cols, err := rows.Columns()

	rowValues := make([][]interface{}, 0)

	vals := make([]interface{}, len(cols))

	// pointer of interface
	for i := range cols{
		vals[i] = new(interface{})
	}

	for rows.Next() {
		rowValue := make([]interface{}, len(cols))
		err := rows.Scan(vals...)
		if err != nil {
			return err
		}
		for i := range cols{
			rowValue = append(rowValue, *(vals[i]).(*interface{}))
		}

		rowValues = append(rowValues, rowValue)
	}

	fmt.Print(strings.Join(cols, ","))
	for i := range rowValues{
		fmt.Print(rowValues[i])
	}

	// testcases
	// request에 있는걸로 / response 비교
	// 매번 connnection (1분 1번)
	// TODO: Close!
	// Request -> user profile api

	return nil
}
~~~

https://www.calhoun.io/using-postgresql-with-go/

// 매번 connnection (1분 1번)

> *json.compact*
> json을 비교하는데 똑같은 json이 자꾸 bytes.Compare 시에 다르다고 나와서 뭔가 했더니, 공백때문에 5개 byte가 더 길게 나왔다. 공백을 제거하기 위해 bytes.TrimSpace 이런거를 써도 JSON string 안에 공백이 있거나 하면 동작이 제대로 안된다. 오히려 데이터를 요상하게 만들 수 있으므로, JSON 내의 공백을 제거하거나 할때 json.Compact를 사용하도록 한다. buffer 를 만들고 해당 buffer에 공백을 제거할 json string을 넣는 방식으로 하면된다.

~~~go
package main

import "bytes"
import "fmt"
import "encoding/json"

func main() {
	var jsonb = []byte(`{
        "foo": "bar"
    }`)
	buffer := new(bytes.Buffer)
	if err := json.Compact(buffer, jsonb); err != nil {
		fmt.Println(err)
	}
	
	fmt.Println(buffer)
}
~~~

> Buffer 을 만들때
> new(Buffer) (or just declaring a Buffer variable) is sufficient to initialize a Buffer

위의 예시에서처럼 new(bytes.Buffer)를 하던지, 
~~~go
buffer := &bytes.Buffer{}
~~~
이런식으로 생성한뒤 주소값을 넣어주면된다. 
~~~go
var buffer *bytes.Buffer
~~~
이런식으로 빈 값을 가르키는 포인터만 만들면 메모리 참조 panic이 날수있으니 조심해야한다.

