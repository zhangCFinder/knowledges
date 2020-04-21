
> 【强制】 在对Map进行遍历的时候不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 Iterator 方式，如果并发操作，需要对 Iterator 对象加锁。
  
HashMap的迭代器(Iterator)是fail-fast迭代器，而HashTable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，具体原因见  <u>Java·ConcurrentModificationException的具体原因</u>
但HashTable的enumerator迭代器的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别
  
  
``` JAVA
public static void main(String[] args) {

  Map<String, String> map = new HashMap<String, String>();
  map.put("1", "value1");
  map.put("2", "value2");
  map.put("3", "value3");
//第一种：普遍使用，二次取值
  System.out.println("通过Map.keySet遍历key和value：");
  for (String key : map.keySet()) {
   System.out.println("key= "+ key + " and value= " + map.get(key));
  }

 //第二种
 /**
  该情况对map执行remove操作不会导致报异常，map.entrySet():返回此映射中包含的映射关系的 Set 视图。该 set 受映射支持，所以对映射的更改可在此 set 中反映出来，反之亦然。如果对该 set 进行迭代的同时修改了映射（通过迭代器自己的 remove 操作，或者通过对迭代器返回的映射项执行 setValue 操作除外），则迭代结果是不确定的。set 支持元素移除，通过 Iterator.remove、Set.remove、removeAll、retainAll 和 clear 操作可从映射中移除相应的映射关系。它不支持 add 或 addAll 操作。
  */
  System.out.println("通过Map.entrySet使用iterator遍历key和value：");
  Iterator<Map.Entry<String, String>> it = map.entrySet().iterator();
  while (it.hasNext()) {
   Map.Entry<String, String> entry = it.next();
   if ("1".equals(entry.getKey())) {     
             it.remove();
     }
   System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
  }
  

//第三种：推荐，尤其是容量大时
  System.out.println("通过Map.entrySet遍历key和value");
  for (Map.Entry<String, String> entry : map.entrySet()) {
   System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
  }


 //第四种
  System.out.println("通过Map.values()遍历所有的value，但不能遍历key");
  for (String v : map.values()) {
   System.out.println("value= " + v);
  }

 }

```

强烈建议使用`EntrySet`进行遍历。

`EntrySet`可以把 key value 同时取出，`keySet`还得需要通过 key 取一次 value，效率较低。


