# Interface

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


# Abstract Class
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

## Abstrct class vs Interface
1. 메서드 타입 interface는 오직 abstract method만, abstract class는 non-abstract method도 가능
2. varialbe -> abstract class는 non-final 변수도 가능하다


여러 클래스에 공통된 코드가 있어야 한다면 abstract class를 사용

무조건 구현되어야 하는 부분이 있다면 interface를 사용


# Comparator interface
user-defined object의 order를 정의(?) 하고 싶을 때 사용한다.
매번 새로운 클래스에 대해 새로운 sort 함수를 정의하는 대신 인터페이스만 구현하도록 하여
기존에 있던 standard sort function을 사용할 수 있다는 장점이 있다.

``` java
public int compare(Object obj1, Object obj2):
```

Collections class의 sort 함수는 정렬시 매번 두개의 원소를 잡아 compare 함수에 전달하고
그것의 반환 값에 따라 순서를 정하게 된다.
반환 값은 -1, 0, 1이 될 수 있는데 각각 less than, equal, greater를 의미한다.




# Collections
![](java-collection.jpg)


# List Interface

ordered colletions of object이며 중복된 원소를 저장할 수 있다.
ArrayList, LinkedList, Vector, Stack 클래스가 List의 구현체이다.

제공하는 연산
1. Positional Access -> numeric index로 접근이 가능하다
2. Search -> 특정 원소를 찾아 해당 위치를 index로 반환해준다.
3. Iteration -> iterator를 이용하여 순회할 수 있다.
4. Range-view -> 리스트의 일부분만을 리스트로써 다룰 수 있게 해준다.


## ArrayList
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

ArrayList와 유사한 것으로는 Vector class가 있다. 이는 legacy 클래스라고 볼 수 있지만 여전히 사용된다. 마찬가지로 dynamic array로 구현되어있지만 synchronised되어 있다.


### Vector vs ArrayList
둘다 dynamic array를 이용하여 List interface를 구현한 구현체 클래스이다.
가장 큰 차이점은 다음과 같다.
1. Synchronization 
	- Vector는 synchronized되어 한 순간에 오직 단 하나의 스레드만이 접근이 가능하며, 반대로 ArrayList는 그렇지 않아 여러개의 스레드가 동시에 동일한 ArrayList를 이용하여 작업을 진행할 수 있다. 그러므로 만약 여러 스레드가 ArrayList를 사용할 일이 생긴다면 lock 등을 이용하여 그 부분을 적절히 처리해줄 필요가 있다.
2. Performance
	- ArrayList가 더 빠르다. 왜냐면 vector와 다르게 thread-safe가 아니기 때문에

그 밖에도 array가 resize되는 방식이나 iterate하는 방식이 조금씩 다르다.


# Set
unordered collections of object이며 중복된 원소를 저장할 수 없다.
HashSet, LinkedSet, TreeSet이 Set interface를 구현한다.
집합을 표현할 때 유용하게 사용할 수 있다.

## Hashset
hashtable을 이용하여 Set interface를 구현한다. insert한 순서대로 들어가는 것이 아니며,
object의 hash code값을 이용하여 들어가게 된다. 	

hashtable을 사용하기 때문에 load factor가 성능에 영향을 준다. -> load factor가 75% 이상이 되면 size를 두 배로 늘려 줄여준다.

HashSet은 내부적으로 HashMap을 이용하여 원소를 저장한다. Map은 key ,value pair로 저장하는데
내부적으로 HashSet은 항상 동일한 value값을 이용하여 모든 key값을 저장한다고 한다.

## LinkedHashSet
hashset의 ordered version이라고 할 수 있다. hashset을 iterate하게 되면 그 순서는 예측할 수
없는데 LinkedHashSet은 doubly-linkedlist로 구현하여 삽입된 순서대로 원소를 순회할 수 있게 해준다.

insertion order를 유지하기 때문에 기존의 hashset, hashmap보다 더 많은 cpu cycle과 메모리를 사용한다는 단점이 있다.

## TreeSet
순서를 유지하진 않지만 key값에 따라 정렬된 상태로 원소를 저장한다. 그렇지만 동일한 원소의 삽입이 불가능하다. 내부적으로는 Red-black tree와 같은 self-balancing binary search tree로 구현된다. 그래서 삽입, 삭제, 검색을 O(logN)에 해결할 수 있다.




# Map
HashMap vs HashTable
둘다 Map 인터페이스의 구현체임
근데 HashMap은 not thread-safe임 여러 스레드에 의해 공유될수 없음
반면에 hashtable은 thread-safe!
-> 만약 synchronization이 필요없다면 hashmap이 더 선호된다.
hashtable에 비해 hashmap은 key나 value값으로 null을 받을 수 있음 (물론 key로 null을 쓰는것은 단 하나만 가능하겠지)