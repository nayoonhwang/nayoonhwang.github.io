---
layout: post
title:  "golang slice"
date:   2019-10-10 14:22:22 +0900
categories: golang slice
---

slice에서 

~~~go
var bSlice []string
aSlice, err := getSlice(param)
// getSlice에서는 slice를 make하고 return한다
bSlice = aSlice
~~~
이렇게 하면 bSlice와 aSlice는 같은 Slice를 보고 있게 된다.

~~~go
bSlice[0] = "first"
fmt.Print(aSlice[0])
// first
~~~

즉, 두 개의 slice 포인터가 내부적으로는 같은 slice를 바라보게 되는것이다.
주의해서 사용해야하지만, 메모리를 약간이라도 아낄수있다. 메모리 할당을 두번할걸 한번으로 줄였기 때문이다.

~~~go
func IntersectionString(left, right []string) []string {
	m := make(map[string]bool)

	for _, value := range left {
		m[value] = true
	}

	var res []string
	for _, value := range right {
		if _, ok := m[value]; ok {
			res = append(res, value)
		}
	}

	return res
}
~~~

이는 두 []string slice의 교집합을 찾는 함수이다. 일단 bool 타입을 값을 가지는 map으로 만든후, 왼쪽 slice의 값을 모두 key로 만든후 value에 true를 대입한다.

아래는 해당 교집합 함수를 각각 O(n^2)와 O(n)의 시간복잡도를 가지는 두 테스트 함수로 시간을 비교하는 테스트 함수이다.
~~~go
func TestIntersectionSpeed(t *testing.T) {
	var a []int
	for i := 0; i < 10000000; i++ {
		a = append(a, i)
	}

	var b []int
	var c int

	for i := 0; i < 300000; i++ {
		c = rand.Intn(100000)
		b = append(b, c)
	}

	boolMap := make(map[int]bool)

	t.Run("test double loop", func(t *testing.T) {
		var commonId []int
		for _, val := range b {
			for _, valA := range a {
				if valA == val {
					a = append(commonId, val)
				}
			}
		}
	})
	
	t.Run("test single loop", func(t *testing.T) {
		for _, id := range b {
			boolMap[id] = true
		}
		var commonId []int
		for _, val := range a {
			if _, ok := boolMap[val]; ok {
				commonId = append(commonId, val)
			}
		}
	})
}
~~~

하지만 놀랍게도, 

~~~
=== RUN   Test
--- PASS: Test (0.27s)
=== RUN   Test/test_double_loop
    --- PASS: Test2/test_double_loop (0.01s)
=== RUN   Test/test_single_loop
    --- PASS: Test2/test_single_loop (0.02s)
	PASS
~~~
밑에 꺼가 더 오래걸리게 나온다.

Slice는 참조시에 메모리가 붙어서 더 빠르게 가능하다는 이야기를 하는데, 아래 글을 참조해야한다고 한다.

https://jacking75.github.io/go_PerformanceTuning/


https://medium.com/swlh/golang-tips-why-pointers-to-slices-are-useful-and-how-ignoring-them-can-lead-to-tricky-bugs-cac90f72e77b