#Interface

interface는 class가 무엇을 해야 하는지를 명시한다. (그렇지만 how는 알려주지 않는다) -> 즉 class의 blueprint라고 할 수 있다.

만약 인터페이스를 구현한 클래스가 주어진 모든 메서드를 구현하지 않는다면 abstract class로 지정선언되어야 한다.

대표적인 예시로는 Comparator interface가 있다. 이를 구현한 클래스는 sort의 대상이 될 수 있다.

``` java
interface <interface_name>{
    
    // declare constant fields -> 모든 필드는 public, static, final
    // declare methods that abstract  -> 모든 메서드는 public, static, abstract
    // by default.
}
```

인터페이스의 instance를 생성할 순 없지만 reference를 생성하여 해당 인터페이스를 구현한 클래스를
가리킬 수 있다.

사용하는 이유

1. abstraction !!
2. 자바가 multiple inheritance를 지원하지 않지만 여러개의 interface를 구현하는 것은 가능함


#Abstract Class
c++에선 최소 하나 이상의 virtual function이 존재하면 해당 class가 abstract가 되었지만
자바에선 명시적으로 abstract keyword를 지정해 주어야 한다.

``` java
// An example abstract class in Java
abstract class Shape {
    int color;
 
    // An abstract function (like a pure virtual function in C++)
    abstract void draw(); 
}
```

abstract class의 instance는 생성할 수 없음 하지만 해당 타입의 reference는 생성할 수 있다.

##Abstrct class vs Interface
1. 메서드 타입 interface는 오직 abstract method만, abstract class는 non-abstract method도 가능
2. varialbe -> abstract class는 non-final 변수도 가능하다


여러 클래스에 공통된 코드가 있어야 한다면 abstract class를 사용

무조건 구현되어야 하는 부분이 있다면 interface를 사용


#Comparator interface
user-defined object의 order를 정의(?) 하고 싶을 때 사용한다.
매번 새로운 클래스에 대해 새로운 sort 함수를 정의하는 대신 인터페이스만 구현하도록 하여
기존에 있던 standard sort function을 사용할 수 있다는 장점이 있다.

``` java
public int compare(Object obj1, Object obj2):
```

Collections class의 sort 함수는 정렬시 매번 두개의 원소를 잡아 compare 함수에 전달하고
그것의 반환 값에 따라 순서를 정하게 된다.
반환 값은 -1, 0, 1이 될 수 있는데 각각 less than, equal, greater를 의미한다.




#Collections
![](java-collection.jpg)

##ArrayList
List interface를 구현한 구현체이며 c++의 vector class와 비슷하다.

기존의 Array와의 차이점은 dynamic size를 제공한다는 점이다.
따라서 기존에는 fixed size를 생성시에 명시해야 하지만 ArrayList는 그렇지 않아도 된다.

Array는 모든 타입을 담을 수 있지만 (생성된 타입에 따라) ArrayList는 object entry에 대해서만 지정이 가능하다. 만약 우리가 arraylist.add(1)을 한다면 이것은 Wrapper class로 autoboxing이 되어 Integer type으로 들어가는 것이다.

또한 ArrayList는 primitive type에 대해 저장하지 않기 때문에 항상 reference로 값을 저장한다.
따라서 각 원소가 저장되는 공간은 Array의 경우 연속적이지만 ArrayList는 그렇지 않다


``` java
// A Java program to demonstrate differences between array
// and ArrayList
import java.util.ArrayList;
import java.util.Arrays;
 
class Test
{
    public static void main(String args[])
    {
        /* ........... Normal Array............. */
        int[] arr = new int[3];
        arr[0] = 1;
        arr[1] = 2;
        System.out.println(arr[0]);
 
        /*............ArrayList..............*/
        // Create an arrayList with initial capacity 2
        ArrayList<Integer> arrL = new ArrayList<Integer>(2);
 
        // Add elements to ArrayList
        arrL.add(1);
        arrL.add(2);
 
        // Access elements of ArrayList
        System.out.println(arrL.get(0));
    }
}
```



