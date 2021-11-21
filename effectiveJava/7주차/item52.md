## 아이템 52. 다중정의는 신중히 사용하라

- 다음은 컬렉션을 집합, 리스트, 그 외로 구분하고자 만든 프로그램이다.

```java
public class CollectionClassifier {
	public static String classify(Set<?> s) {
		return "Set";
	}
	public static String classify(List<?> lst) {
		return "List";
	}
	public static String classify(Collection<?> c) {
		return "Unknown Collection";
	}

	public static void main(String[] args) {
		Collection<?>[] collections = {
			new HashSet<String>,
			new ArrayList<BigInteger>(),
			new HashMap<String, String>().values()
		};

		for (Collection<?> c : collections) {
			System.out.println(classify(c));
		}
	}
}
```



작성자가 오버로딩을 구현한 의도와 다르게 하나의 메서드만이 사용된다.

```
Unknown Collection
Unknown Collection
Unknown Collection
```

여러개의 오버로딩된 메서드 중에서 **어떤 메서드를 호출할지에 대한 선택은 컴파일 시점에 결정된다.**

따라서, `Collection<?>` 타입의 배열을 순회한다면, `Collection<?>` 타입으로 모두 인지한다. 

왜냐하면 컴파일러는 `Collection<?>[]` 안에 항목이 `HashSet` 인지, `ArrayList` 인지는 **런타임 시점에 알 수 있기 때문이다.**



**메서드 오버라이드의 메서드 선택은 오버로딩과 다르게 런타임 시점에 결정된다.**

다음 코드를 보고 출력 결과를 예상해보자.

```java
public class Overridings {
	public static void main(String[] args) {
		List<Wine> wineList = List.of(new Wine(), new SparklingWine(), new Champagne());

		for (Wine wine : wineList) {
			System.out.println(wine.name());
		}
	}
}

class Wine {
	String name() { return "wine"; }
}
class SparklingWine extends Wine {
	@Override String name() { return "Sparkling Wine"; }
}
class Champagne extends Wine {
	@Override String name() { return "Champagne"; }
}
```

출력 결과는 다음과 같다.

```
wine
Sparkling Wine
Champagne
```



앞에서 살펴본 `CollectionClassifier` 예제를 의도한 대로 수정하려면 어떻게 해야할까?

```java
public static String classify(Collection<?> c) {
  return c instanceof Set ? "Set" :
  c instanceof List ? "List" : "Unknown Colleciton";
}
```



책의 저자는 오버로딩을 사용할 때 가장 보수적인 방법으로 다음과 같이 권한다.

- 같은 개수의 파라미터를 갖는 오버로딩 메서드 2개 이상을 갖지 마라

이러한 보수적 정책의 가장 좋은 예는 `ObjectOutputStream` 이다.

ObjectOutputStream은 write 메서드를 기본 자료형마다 오버로딩하지 않고 메서드에 자료형 이름을 붙였다. 예를 들어, `writeInt(int)` , `writeBoolean(boolean)`, `writeLong(long)` 로 정의되어 있다.



다음 코드를 실행해보자. 어떤 결과가 나올까?

```java
public static void main(String[] args) {
  Set<Integer> set = new TreeSet<>();
  List<Integer> list = new ArrayList<>();

  for (int i = -3; i < 3; i++) {
    set.add(i);
    list.add(i);
  }
  for (int i = 0; i < 3; i++) {
    set.remove(i);
    list.remove(i);
  }

  System.out.println(set + " " + list);
}
```

Set과 List 각각에서 양수만 제거된 결과가 도출될 것이라고 예상할 것이다.

```
[-3, -2, -1] [-3, -2, -1]
```

하지만, 실제 결과는 다음과 같다.

```
[-3, -2, -1] [-2, 0, 2]
```



`set.remove(i)` 는 오버라이딩 메서드인 `remove(E)` 를 호출한다. 

여기서 E는 제너릭 타입이므로 오토박싱이 발생한다.

반면, `list.remove(i)` 는 인덱스를 지정하여 삭제하는 메서드다. 

그렇기 때문에 list의 0번, 1번, 2번 인덱스에 해당되는 -3이 지워지고, [-2, -1, 0, 1, 2] 중에서 1번 인덱스인 -1이 제거된다. 마지막으로 list는 [-2,0,1,2] 가 되어, 2번 인덱스인 1이 제거된다. 따라서 결과는 [-2, 0, 2]가 된다.



그렇다면, 원래 우리가 예상한 방식으로 동작되게 하려면 어떻게 해야할까?

remove() 메서드를 호출할 때, 파라미터를 int를 기초형이 아닌 박싱 타입인 Integer로 넘겨야 한다. 

```java
for (int i = 0; i < 3; i++ ){
  set.remove(i);
  list.remove((Integer) i); // remove(Object o) 가 호출됨
}
```


> 오버로딩 메서드를 안전하게 사용하는 가장 최적의 방법은 다른 개수의 파라미터에만 오버로딩을 정의하는 것이다. 오버로딩 대신, 메서드의 이름을 달리하여 다중정의를 흉내내자.