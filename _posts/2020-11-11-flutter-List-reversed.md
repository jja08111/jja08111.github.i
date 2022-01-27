---
title: "[Flutter] List 뒤집기(reverse)"
date: 2020-11-11 17:14:00 -0400
categories: flutter
tags:
- flutter
- List
- reversed
---

# List를 뒤집자 
`Flutter`에서 `List`를 뒤집은, 즉 `인덱스 순서가 뒤집힌 List`를 얻으려면 어떻게 해야하는가? 

단순히 List에 있는 reversed를 이용하려 했으나 실패.. [찾아보니](https://www.tutorialkart.com/dart/dart-reverse-list/) 방법은 다음과 같다. 
```dart
void main(){
    //a list
    List<int> myList = [24, 56, 84, 92];
    
    // myList를 뒤집어 순서가 바뀐 iterable을 형성하고 그것으로 새로운 리스트 객체를 만든다.
    List<int> reversedList = List.from(myList.reversed);
    
    print(reversedList);
}
```

위의 코드에서 눈여겨 볼 부분은 `List.from(myList.reversed)`이다.  
이와 같은 방법으로 List를 reverse 할 수 있다.
