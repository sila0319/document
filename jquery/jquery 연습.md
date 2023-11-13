# Jquery Selectors



## 1. 특정 태그 다 감추기

``` javascript
$("p").hide();
```



* $("p") 이 부분에 들어간 p는 html에 p 태그를 전부다 선택한다.
* hide 는 css display 속성을 none으로 설정해서 요소를 다 숨긴다.



## 2. 특정 id 감추기

``` javascript
$("#id값").hide();
```

* 특정 id를 가진 태그를 숨기기 위해서는 "#id값"을 입력하고 .hide()를 붙이게 되면 해당 아이디를 가진 태그가 숨겨지게 된다.



## 3. 모든 요소 다 감추기

``` js
$("*").hide();
```



## 4. 특정 속성 감추기

``` js
$("[attribute]").hide()
```

* 태그와 다르게 속성은 "[attribute]" 이런 방식으로 작성을 해야한다. 



# 5. 