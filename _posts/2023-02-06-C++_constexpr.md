
---
layout: single
title:  "constexpr"
categories: Unreal
tag : [c++]
toc : true
author_profile : false
sidebar:
    nav: "docs"

# search : false
---

**1. constexpr**

http://en.cppreference.com/w/cpp/language/constexpr

기존의 const 보다 훨씬 더 상수성에 충실하며, 컴파일 타임 함수를 이용한 성능 향상 등 충분히 깊게 이해할만한 가치가 있는 녀석이라 할 수 있으니, 확실히 이해하고 활용할 수 있는 것이 중요하다.

C++11부터 지원되는 한정자 **constexpr**는 **일반화된 상수 표현식(Generalized constant expression)**을 사용할 수 있게 해주며, 일반화된 상수 표현식을 통해 **변수**나 **함수, 생성자 함수**에 대하여 **컴파일 타임에 평가**될 수 있도록 처리해 줄 수 있다.

(C++17부터는 **람다 함수**에도 constexpr 키워드 사용이 가능하다)

**constexpr 변수 또는 함수의 반환값은 반드시 LiteralType이어야** 하며, **LiteralType은 컴파일 타임에 해당 레이아웃이 결정될 수 있는 타입**을 의미한다. 다음은 리터럴 타입들의 유형이다.

- void
- 스칼라 값
- 참조
- void, 스칼라 값, 참조의 배열
- trivial 소멸자 및 이동 또는 복사 생성자가 아닌 constexpr 생성자를 포함하는 클래스. 또한 해당 비정적 데이터 멤버 및 기본 클래스가 모두 리터럴 타입이고 volatile이 아니어야 함

코드 작업 중 해당 타입이 리터럴 타입인지 확인하고 싶을 땐, **std::is_literal_type**을 사용하면 된다.

**1) 변수에서의 사용**

const와 constexpr의 주요 차이점은 **const 변수의 초기화를 런타임까지 지연시킬 수 있는 반면, constexpr 변수는 반드시 컴파일 타임에 초기화가 되어 있어야 한다.**

초기화가 안 되었거나, 상수가 아닌 값으로 초기화 시도시 컴파일이 되지 않는다.



```c++
1. constexpr float x = 42.f;  // OK
2. constexpr float y { 108.f }; // OK
3. constexpr int i;       // error C2737: 'i': 'constexpr' 개체를 초기화해야 합니다.
4. int j = 0;
5. constexpr int k = j + 1;   // error C2131 : 식이 상수로 계산되지 않았습니다.
```

**변수에 constexpr 사용시 const 한정자를 암시**한다.



**2) 함수에서의 사용**

constexpr을 함수 반환값에 사용할 때는 다음의 제약들이 따른다.

- 가상으로 재정의된 함수가 아니어야 한다.
- 반환값의 타입은 반드시 LiteralType이어야 한다.

**함수에 constexpr을 붙일 경우 inline을 암시**한다.

즉, 컴파일 타임에 평가하기 때문이며, **inline 함수들과 같이 컴파일**된다.

C++11에서는 함수 본문에 지역변수를 둘 수 없고, **하나의 반환 표현식만**이 와야 하는 제약이 있었으나, C++14부터는 이러한 제약이 사라졌다.

```c++
1. // C++11/14 모두 가능
2. constexpr int factorial(int n)
3. {
4.   // 지역 변수 없음
5.   // 하나의 반환문
6.   return n <= 1 ? 1 : (n * factorial(n - 1));
7. }
8.  
9. // C++11에서는 컴파일 에러 발생
10. // C++14부터 가능
11. constexpr int factorial(int n)
12. {
13.   // 지역 변수
14.   int result = 0;
15.  
16.   // 여러 개의 반환문
17.   if (n <= 1)
18. ​    result = 1;
19.   else
20. ​    result = n * factorial(n - 1);
21.  
22.   return result;
23. }
```



constexpr 함수는 컴파일러에게 가능하다면, 상수시간에 컴파일해 달라고 요청하는 것이지만 상황에 따라 컴파일 타임에 미리 실행될 수도 있고, 그렇지 못하고 런타임에 실행될 수도 있다.

(마치 inline 키워드가 그러하듯이 말이다)

**constexpr의 함수 인자들이 constexpr 규칙에 부합하지 못하는 경우엔 컴파일 타임에 실행되지 못하고 런타임에 실행된다.**

런타임 실행 여부는 여러 가지 방식으로 검증해 볼 수 있다.

- breakpoint 걸어서 중단되는지 확인
- 아래 예제의 constN 같은 테스팅 템플릿 작성
- 정수일 경우 배열의 dimension으로 작성



```c++
1. // 템플릿 인자 N이 컴파일 타임 상수인지 테스트하기 위한 템플릿 구조체
2. template<int n>
3. struct constN
4. {
5.   constN() { std::cout << N << '**\n**'; }
6. };
7. 
8. constexpr int factorial(int n)
9. {
10.   return n <= 1 ? 1 : (n * factorial(n - 1));
11. }
12.  
13. int main()
14. {
15.   // 4는 리터럴 타입이므로 상수 타임에 컴파일 성공
16.   constN<factorial(4)> out1;
17.  
18.   // ab는 4의 값을 가지지만, 리터럴 타입이 아니므로 컴파일 에러 발생
19.   // error C2975: 'N': 'constN'의 템플릿 인수가 잘못되었습니다. 컴파일 타임 상수 식이 필요합니다.
20.   int ab = 4;
21.   constN<factorial(ab)> out2;
22.  
23.   // 리터럴 타입이 아니므로, 이 함수는 런타임에 실행된다.
24.   // 하지만 정상 동작한다.
25.   int cd = factorial(ab);
26.  
27.   return 0;
28. }
```



25라인에서 factorial 함수의 인자가 constexpr로 평가될 수 없기에, 함수이지만 런타임에 실행되는 것을 확인할 수 있다. 

이처럼 constexpr 함수는 인자가 constexpr에 부합한지에 따라, 컴파일 타임 또는 런타임에 실행되기에 범용적으로 사용되는 함수이고, 실행의 복잡도가 낮지 않다면, 가급적 constexpr 키워드를 붙이는 것도 괜찮은 습관이 되지 않을까 생각한다.

위의 예제에서는 함수가 어느 시점에 실행되는지 여부를 살펴보았지만, 무작정 constexpr 키워드를 붙일 수 있는 것도 아니다.

함수가 절대 상수표현식으로써 평가받지 못하는 문맥을 가지는 경우엔 컴파일 에러가 발생한다.



```c++
1. constexpr int rand_short()
2. {
3.   // 절대 상수화가 될 수 없는 문맥
4.   // error C3615: constexpr 함수 'randshort'의 결과가 상수 식이면 안 됩니다.
5.   return rand() % 65535;
6. }
```

컴파일 에러니까 함수 작성 후 바로 확인이 가능하기에 심각한 오류를 사전에 만나는 일은 없을 것이다.



**3) 생성자 함수에서의 사용**

LiteralType 클래스를 생성할 때 constexpr 생성자를 사용할 수 있다.

이 때의 제약은 다음과 같다.

- 모든 생성자 함수의 매개변수들 역시 LiteralType들이어야 한다.
- 어떤 클래스로부터 상속받지 않아야 한다

constexpr이 적용된 함수의 매개변수는 반드시 LiteralType이어야 한다고 했다.

이를 만족시키기 위해 constexpr 생성자 함수를 이용, LiteralType 클래스를 활용하는 예제를 살펴보도록 하자.



```c++
1. // 아래 CountLowercast 함수의 인자로 사용하기 위한 LiteralType class
2. class ConstString
3. {
4. private:
5.   const char* p = null;
6.   std::size_t sz = 0;
7.  
8. public:
9.   template<std::size_t N>
10.   constexpr ConstString(const char(&a)[N])
11.   : p(a), sz(N - 1)
12.   {
13.   }
14.  
15. public:
16.   constexpr char operator[](std::size_t n) const
17.   {
18. ​    return n < sz ? p[n] : throw std::out_of_range("");
19.   }
20.  
21. public:
22.   constexpr std::size_t size() const { return sz; }
23. };
24.  
25. // 소문자가 몇개인지 수를 세는 함수
26. constexpr std::size_t CountLowercase(ConstString s, std::size_t n = 0, std::size_t c = 0)
27. {
28.   return n == s.size() ?
29. ​          c :
30. ​          s[n] >= 'a' && s[n] <= 'z' ?
31. ​            CountLowercase(s, n+1, c+1) :
32. ​            CountLowercase(s, n+1, c);
33. }
34.  
35. // 컴파일타임 상수를 테스트해 보기 위한 템플릿
36. template<int n>
37. struct ConstN
38. {
39.   ConstN()
40.   {
41. ​    std::cout << n << '**\n**';
42.   }
43. };
44.  
45. int main()
46. {
47.   std::cout << "Number of lowercase letters in **\"**Hello, world!**\"** is ";
48.  
49.   // "Hello, world!"가 ConstString으로 암시적 형반환되어, CountLowercase 함수의 인자로 넘어감.
50.   // CountLowercase는 컴파일 타임에 평가되었고, 그 결과를 가지고 ConstN 역시 컴파일 타임에 결정됨.
51.   ConstN<CountLowercase("Hello, world!")> out;
52.  
53.   return 0;
54. }
```



**2. 템플릿 메타 프로그래밍 vs constexpr**

constexpr은 컴파일 타임에 평가되기 때문에 템플릿 메타 프로그래밍(템플릿 함수가 인스턴스화될 때 값 계산)과 비교될 수 있다.

예를 들어, 똑같이 배열의 크기나 enum 열거형의 값과 같은 곳에서 상수로써 사용이 가능하다.



```c++
1. // 템플릿 함수 방식의 Factorial
2. template <int N>
3. struct Template_Factorial
4. {
5.   enum { value = N * Template_Factorial<N - 1>::value; }
6. };
7.  
8. template <>
9. struct Template_Factorial<0>
10. {
11.   enum { value = 1; }
12. }
13.  
14. // constexpr 방식의 Factorial
15. **constexpr** int Constexpr_Factorial(int n)
16. {
17.   return n <= 0 ? 1 : n  * Constexpr_Factorial(n - 1);
18. }
19.  
20. // constexpr 함수의 결과를 enum의 값으로 사용 가능
21. enum FACTORIAL
22. {
23.   first  = Constexpr_Factorial(1),
24.   second = Constexpr_Factorial(2),
25.   third  = Constexpr_Factorial(3),
26. };
```





위 예제에서 보듯이, 기존의 TMP에서 0과 같은 특수값을 사용하기 위해 템플릿 특수화를 했던 것에 비해, constexpr 함수는 훨씬 더 직관적인 방법을 제공할 수 있다.

위 예제에 피보나치까지 살짝 추가해, 조금 더 비교해 보면 기존 코드를 죄다 다시 작성하고 싶어질 것이다.



```c++
1. template<unsigned n>
2. struct Fibonacci
3. {
4.   static const unsigned value = Fibonacci<n - 1>::value + Fibonacci<n - 2>::value;
5. };
6.  
7. template<>
8. struct Fibonacci<0>
9. {
10.   static const unsigned value = 0;
11. };
12.  
13. template<>
14. struct Fibonacci<1>
15. {
16.   static const unsigned value = 1;
17. };
18.  
19. int main()
20. {
21.   return Fibonacci<5>::value; 
22. }
23.  
24. ///////////////////////////////////////////////////////////////////////////////////////////////
25. 
26. constexpr unsigned fibonacci(const unsigned x)
27. {
28.   return x <= 1 ? 1 : fibonacci(x - 1) + fibonacci(x - 2);
29. }
30.  
31. int main()
32. {
33.   return fibonacci(5);
34. }
```



[참조사이트] http://egloos.zum.com/sweeper/v/3147813