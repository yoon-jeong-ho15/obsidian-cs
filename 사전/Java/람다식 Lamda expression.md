
# 자바
자바를 포함한 객체 지향 프로그래밍 언어에서는 객체 없이 함수만 사용할 수 없다. 
그러나 람다 표현식을 통해서 함수형 프로그래밍 방식을 구현할 수 있다.

기존의 익명 내부 클래스의 기능을 사용하면서 더 간편하게 작성할 수 있다.

## 용례
List 객체의 sort() 메소드
### 1. 람다식의 도입 배경
- 익명 내부 클래스를 더 간단하게 표현하기 위해 도입
- 특히 함수형 인터페이스(추상 메서드가 하나만 있는 인터페이스)를 구현할 때 사용
### 2. Comparator와 람다식

```java
// 정렬을 위한 Comparator 인터페이스 
public interface Comparator<T> {
	int compare(T o1, T o2);  // 추상 메서드 딱 하나
} 
// 사용 방법 비교 
// 1. 익명 내부 클래스 
Collections.sort(list, new Comparator<Integer>() {
@Override
public int compare(Integer o1, Integer o2) {
	return o1 - o2;
	} 
}); 
// 2. 람다식 - 훨씬 간단! 
Collections.sort(list, (a, b) -> a - b);
```
### 3. Collections.sort() 메서드
두 가지 버전이 오버로딩되어 있음:
```java
// 1. 기본 정렬 
sort(List<T> list)  // T는 Comparable을 구현해야 함 
// 2. 커스텀 정렬 
sort(List<T> list, Comparator<? super T> c)
```
### 4. 람다식으로 다양한 정렬 구현
```java
// 점수 기준 정렬
(a, b) -> a.getScore() - b.getScore() 
// 이름 기준 정렬 (a, b) -> a.getName().compareTo(b.getName()) 
// 복합 조건 정렬 
(a, b) -> {
	if(a.getSubject().equals(b.getSubject())) {
		return b.getScore() - a.getScore();
	 }
	return a.getSubject().compareTo(b.getSubject()); 
}
```
### 5. 제네릭 T의 중요성
- Object 대신 제네릭 T를 사용하는 이유:
    - 컴파일 시점 타입 검사 가능
    - IDE 자동완성 지원
    - 타입 안정성 보장
    - 형변환 불필요
### 6. compare() 메서드의 반환값 의미

- 음수: 첫 번째 객체가 더 작음 (앞으로)
- 0: 두 객체가 같음
- 양수: 첫 번째 객체가 더 큼 (뒤로)
### 7. 주의사항
- compare() 구현 시 의미 있는 비교 로직 사용해야 함
- 매개변수는 반드시 2개여야 함 (Comparator 인터페이스 규칙)
- 정렬 결과의 일관성을 위해 비교 규칙 준수 필요

## 개념