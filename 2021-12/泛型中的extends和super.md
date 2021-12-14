## extends
`List<? extends Number>` 列表里的元素都是Number或Number的子类型，不能add新元素，但可以被子类列表赋值，取元素时只能取出Number类型元素

```java
List<? extends Number> list1 = new ArrayList<>();
list1.add(1); //会编译错误

List<Integer> list2 = new ArrayList<>();
list2.add(1);
list1 = list2;
Integer i1 = list1.get(0); // 会编译错误
Number number = r.get(0); // 编译正常
```

## super

`List<? super Integer>` 列表里的元素都是Integer或Integer的父类型，可以add新元素，可以被Integer父类型赋值，但取值时只能取Object类型，需要强转

```java
List<? super Integer> list1 = new ArrayList<>();
list1.add(1); // add正常
Integer i1 = list1.get(0); // 会编译报错
Object object = list1.get(0); // 编译正常
Integer i2 = (Integer) list1.get(0);
```

PECS: Producer Extends, Consumer Super