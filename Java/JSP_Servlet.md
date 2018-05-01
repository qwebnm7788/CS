# Expression Language

``` java
<%-- 표현식 --%>
<%= member.getAddress() %>

<%-- EL --%>
${member.address}
```

기존의 표현식에 비해 EL을 사용하면 코드가 간결해지고 이해하기 쉬워진다.

EL은 JSP의 스크립트 요소(스크립틀릿, 표현식, 선언부)를 제외한 모든 부분에서 사용이 가능하다.

JSP2.1부터는 $이 아닌 #{}도 존재함 -> 차이는 EL의 값을 언제 생성하느냐에 있음

EL은 값이 존재하지 않을 때 null을 출력하는 것이 아닌 아무것도 출력하지 않는 방식을 갖는다.

값을 참조할때 영역을 표시하지 않으면 page -> request -> session -> application영역을
순서대로 검사하여 해당 값을 찾는다.
