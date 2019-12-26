---
layout: post
title:  "Passing Slice to a function"
date:   2019-12-26 10:45:22 +0900
categories: memory golang
---

절차지향 프로그래밍의 가장 흔한 특징은 배열 개념이다. 배열은 간단한 것처럼 보이지만 우리가 언어에 배열을 적용할때에는 몇가지 답변해야할 문제들이 있다.

* 고정된 사이즈인가 변수인가?
* 사이즈가 타입의 부분인가?
* 다차원 배열은 어떻게 생겼는가?
* 빈 배열이 의미가 있는가?

이 문제들의 답변은 배열이 그저 언어의 특징인지, 디자인의 핵심부분인지에 영향을 끼친다.

Go에서는 고정 길이 배열을 기반으로 만들어져 유동적이고 확장가능한 데이터 구조를 가지는 Slice가 이 문제의 답변의 핵심이다. 

# 배열

배열은 고에서 중요한 부분이지만, 어떤 건물의 토대처럼 눈에 보이는 컴포넌트들 아래 숨겨져있다. 우리는 좀 더 흥미로운 주제인 Slice를 이야기하기 전에 배열에 대해서 간단히 짚고 넘어가야한다.

배열은 사이즈가 타입의 일부이고, 이는 표현성을 한정시키기 떄문에 Go 프로그램에서 그닥 잘 보이지 않는다.

~~~go
var buffer [256]byte
~~~
이는 buffer라는 256바이트짜리 변수를 선언한다. 이 변수 buffer의 타입은 256이라는 사이즈를 포함한다. 이 배열과 관련된 데이터는 딱 원소들의 배열일 것이다. 설계상으로 아마 이렇게 메모리에 있을 것이다.

~~~bash
buffer: byte byte byte ... 256 times ... byte byte byte
~~~
즉, 이 변수는 256 바이트의 데이터"만" 저장하고 있다. 이 원소들은 buffer[0], buffer[1] 같은 친숙한 인덱싱 문법으로 접근할 수 있다. 배열은 보통 slice를 위한 저장소로서의 목적으로 가장 많이 사용된다.

# slice
slice를 쓰기위해서는 그것들이 무엇이고 무엇을 하는지 정확히 이해를 하고 있어야한다.

slice는 slice 변수 그 자체와는 별개로 저장되는 배열의 연속적인 섹션을 표현하기 위한 자료구조이다. slice는 배열이 아니다. slice는 배열의 일부를 나타낸다. 우리 buffer 배열이 있다고 할때, 배열을 slicing 함으로써 우리는 100에서 150까지의 원소를 나타내는 slice를 만들 수 있다. 

~~~go
var slice []byte = buffer[100:150]
// var slice = buffer[100:150]
// slice := buffer[100:150]
~~~

slice 변수는 정확히 무엇일까? 정확히 그런건 아니지만, 일단 slice를 길이와 배열의 원소에 대한 포인터, 두 개 원소를 가지는 자료구조라고 생각해보자. 

~~~go
type sliceHeader struct {
    Length        int
    ZerothElement *byte
}

slice := sliceHeader{
    Length:        50,
    ZerothElement: &buffer[100],
}
~~~

물론 이건, 그냥 예시다. 사실 sliceHeader는 프로그래머들에게 보이는 영역이 아니고, 원소의 포인터의 타입이 원소 타입에 의존적이지만, 이 예시는 slice의 원리를 설명하고 있다.

~~~go
slice2 := slice[5:10]
// 밑에 sliceHeader 구조체는 이런식으로 생겼을것이다.
// slice2 := sliceHeader {
// 	 Length: 5,
// 	 ZerothElement: &buffer[105],
// }
~~~
이 헤더가 buffer 변수에 저장되 있는 같은 배열을 가르키고 있다는 걸 주의해야한다.

또한 reslice할 수 있는데, slice를 slice해서 기존의 slice 구조체로 결과를 다시 저장하는 것을 의미한다.
~~~go
slice = slice[5:10]
~~~
이것의 sliceHeader 구조는 slice2 변수꺼와 아예 같을 것이다. slice를 truncate할때와 같이, reslicing을 하는 경우는 종종 볼 수 있다. 

~~~go
slice = slice[1:len(slice)-1]
// sliceHeader {
//	 Length: 3,
//	 ZerothElement: &buffer[106],
// }
~~~

짬 찬 go 프로그래머가 slice header 에 대해서 말하는 걸 들은 적 있을 것이다. slice header는 사실 실제로 slice 변수가 가지고 있는 값이다. slice를 argument로 가지는 함수를 호출할때, 함수에 실제로 전달되는 건 sliceheader이다. slice header에는 사실 한가지 아이템이 더 있는데, 일단 슬라이스 헤더의 존재여부가 slice를 다루는 프로그램에 어떤 영향을 주는 지부터 알아본다.

# 함수에 slice를 전달할때

Slice가 포인터를 포함하고 있지만 그자체로서는 "값"이라는 걸 이해하는 게 중요하다. Slice는 포인터와 길이를 가지고 있는 구조체 값이다. 구조체를 가르키는 포인터가 아니다. 

이전 예시에서 IndexRune을 호출할때

Go 에서 파라미터를 전달할때 Reference가 아닌, Value로 전달한다. Slice의 경우에도 마찬가지이다. 이는 Pointer와 길이를 가진 구조체 값이라고 할 수 있다.