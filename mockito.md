# Mockito

虛擬物件

用來驗證是否如預期的互動行為，例如:

```java
void add(List<String> list, String item) {
   list.add(item);
}

void testAddList() {
    List<String> mockedList = mock(List.class);
    add(mockedList, "one");
    verify(mockedList).add("one");
}
```

也可以偽裝行為，例如：

```java
LinkedList mockedList = mock(LinkedList.class);

// 如果有人呼叫 get(0) ，就回傳 "first"
when(mockedList.get(0)).thenReturn("first");

assertEquals("first", mockedList.get(0));
```

## 安裝

dependencies { testCompile "org.mockito:mockito-core:1.+" }

## See Also

* Mockito - https://github.com/mockito/mockito http://mockito.org/
* EasyMock - https://github.com/easymock/easymock http://easymock.org/
* PowerMock - https://github.com/jayway/powermock
