---
comments: true
tags: javascript
layout: post
title: [javascript] 기본형과 참조형의 종류 및 차이점



---



## [javascript] 기본형과 참조형의 종류 및 차이점

javascript data type에는 기본형과 참조형이 존재한다.

기본형(primitive Type) : 값을 그대로 할당한다.

ex) Number, String, Boolean, null, undefined

참조형(reference Type) : 값이 저장된 주소값을 할당한다(참조)

ex) Object, Array, Function, RegExp



##### [기본형]

```javscript
var a;
a = 10;
b = 'abc'
```

| 변수명 | a    | b    | c    |
| ------ | ---- | ---- | ---- |
| 주소   | @313 | @314 |      |

| 주소   | 313  | 314   | 315  | 316  |
| ------ | ---- | ----- | ---- | ---- |
| 데이터 | 10   | 'abc' |      |      |



변수 a를 선언하게 되면 해당 변수를 저장할 주소값을 할당한다. 이 경우에는 313 주소를 할당 후, a = 10에 따라 313 주소공간에 10을 할당하였다.

var b = 'abc'를 선언하면 동일하게 해당 변수를 저장할 주소값(314)를 할당하고, 해당 주소에 'abc'를 할당하였다.



```javscript
b = false
c = b
```

| 변수명 | a    | b    | c    |
| ------ | ---- | ---- | ---- |
| 주소   | @313 | @314 | @315 |

| 주소   | 313  | 314   | 315   | 316  |
| ------ | ---- | ----- | ----- | ---- |
| 데이터 | 10   | false | false |      |

b=false라고 값을 넣을 경우, 314의 주소를 찾아가서 해당 값을 false로 변경한다. 그 후, c = b에 의해 c는 315 주소값을 할당받고 해당 값인 false를 넣는다. 

이 경우 b와 c는 같지만, c = 20 인 값을 넣게 되는 경우, b와 c는 같지 않게 된다.



```javscript
c=20
```

| 변수명 | a    | b    | c    |
| ------ | ---- | ---- | ---- |
| 주소   | @313 | @314 | @315 |

| 주소   | 313  | 314   | 315  | 316  |
| ------ | ---- | ----- | ---- | ---- |
| 데이터 | 10   | false | 20   |      |



##### [참조형]

```javscript
var obj = {
	a: 1,
	b: 'b'
}
```

| 변수명 | obj  |      |      |
| ------ | ---- | ---- | ---- |
| 주소   | 413  |      |      |

| 주소   | 413   | 414  | 1011                                   | 1012 | 1013 |
| ------ | ----- | ---- | -------------------------------------- | ---- | ---- |
| 데이터 | @1011 |      | {<br />a : @1012,<br />b: @1013<br />} | 1    | 'b'  |

obj를 선언 후, 주소공간을 할당(413)을 한다. 해당 값을 넣을려고 보니, 참조형이고, 참조형 변수의 경우, property와 data 형태로 구성되어 있으며, property는 변수와 비슷한 성질을 지닌다. 따라서 property인 a를 할당하기 위한 주소를 할당(1012)받고, 1012 주소에 해당 값을 저장한다. 동일하게 b를 할당하기 위한 주소(1013)을 할당받고, 1013 주소에 값을 저장한다.



```javscript
var obj2 = obj
obj2.a = 10
console.log(obj.a) // 10
```

| 변수명 | obj  | obj2 |      |
| ------ | ---- | ---- | ---- |
| 주소   | 413  | 414  |      |

| 주소   | 413   | 414   | 1011                                   | 1012 | 1013 |
| ------ | ----- | ----- | -------------------------------------- | ---- | ---- |
| 데이터 | @1011 | @1011 | {<br />a : @1012,<br />b: @1013<br />} | 10   | 'b'  |

obj2를 선언 후, obj를 할당하게 되면 obj2는 obj가 가리키는 주소값을 할당한다. 

이 후, obj2.a에 값 10을 할당하게 되면 obj2는 @1011 주소를 찾아가고 해당 property인 a의 주소인 1012의 위치로 가서 해당 값을 10으로 바꾸어 버린다.

그래서 obj.a를 찍어보면 해당 값은 10이 된걸 확인할 수 있다.



##### [nested]

```javscript
var obj3 = {
	a: [4,5,6]
}
```

| 변수명 | Obj3 |      |      |
| ------ | ---- | ---- | ---- |
| 주소   | @547 |      |      |

| 주소   | 547   | 1184                    | 1185                                         | 1326 | 1327 | 1328 |
| ------ | ----- | ----------------------- | -------------------------------------------- | ---- | ---- | ---- |
| 데이터 | @1184 | {<br />a : @1185<br />} | [<br />@1326,<br />@1327,<br />@1328<br />}] | 4    | 5    | 6    |

obj3를 선언 후, 주소(547)를 할당 받고, a를 저장하기 위해 주소공간(1185)를 할당 받는다. 할당할 값을 봤더니 해당 값 또한 참조형이므로 해당 값을 저장하기 위해 새로운 주소공간(1326, 1327, 1328)을 할당받고 해당 값을 저장한다.



```javscript
obj3.a = 'new'
```

| 변수명 | Obj3 |      |      |
| ------ | ---- | ---- | ---- |
| 주소   | @547 |      |      |

| 주소   | 547   | 1184                    | 1185  | 1326 | 1327 | 1328 |
| ------ | ----- | ----------------------- | ----- | ---- | ---- | ---- |
| 데이터 | @1184 | {<br />a : @1185<br />} | 'new' | 4    | 5    | 6    |

obj3.a에 new라는 값을 할당해버리면, a가 가리키는 주소(1185)에 'new'를 할당한다. 기존에 해당 값을 참조하고 있는 1326, 1327, 1328은 참조관계를 잃어버리므로, 추후 GC(garbage collector)에 의해 해당 값은 정리가 된다.



[참조]


