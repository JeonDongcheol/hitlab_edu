# [스터디] 파이썬 :: 4. 함수와 모듈
> 작성일: 2018.7.16 , 작성자: 전동철

이번 장에서는 파이썬의 함수와 모듈에 대해서 알아보도록 한다.

### 목차
1. [모듈이란?](#m1)
2. [모듈의 장점](#m2)
3. [모듈 검색 순서](#m3)
4. [모듈 사용 방법](#m4)
5. [참고자료](#m5)

---
 
## 1. 모듈이란? <a name="m1"/>

**모듈(Module)** 이란 기능별로 나뉘어진 프로그램을 말한다. 전체 프로그램을 구성하는 **일부** 로서, 그 일부와 관련된 데이터 및 함수들이 묶여서 형성된 것을 의미한다. 모듈은 파일 단위로 나뉘며, 모듈 안에서는 일반 파이썬 코드들과 같이 함수, 클래스, 변수 등 모두 사용이 가능하다.

## 2. 모듈의 장점 <a name="m2"/>

모듈의 장점은 다음과 같다.

1. **높은 효율성** : 사용 용도에 따라 구분이 가능하고 일부분만 사용이 가능하다.
2. **타인의 코드를 활용 가능** : 타인이 만든 프로그램 중 전부를 가져다 쓰는 것이 아닌 일부를 가져가서 활용이 가능하다.
3. **유지보수의 용이함** : 기능별로 구분해서 관리가 편리하다.

모듈을 사용하게 되면 몇 천줄이나 되는 코드의 줄이 기능 별로 파일 단위로 구분이 되어 알기가 쉽고 관리하는 것에 편리함이 있다. 그렇기 때문에 전체 프로그램을 모듈화 해서 관리하는 것이 매우 중요하다.

## 3. 모듈의 검색 순서 <a name="m3"/>

### 모듈의 검색 순서

파이썬 모듈이 있는 경로들은 **시스템 경로** 에서 리스트 형태로 취합이 되고 사용할 때 시스템 경로에서 검색해서 가져오는데, 기본적으로 모듈의 검색 순서는 다음과 같다.

1. 현재 디렉토리
2. 환경변수 PYTHONPATH에 지정된 경로
3. 파이썬이 설치된 경로 및 그 밑의 라이브러리 경로

위의 경로가 모두 취합되어 시스템 경로인 **sys.path** 에 저장이 된다. 모듈을 사용하려고 하는데, sys.path 경로 안에 없다면 에러가 발생한다.

### 모듈 경로의 추가 방법

만약에 파이썬 모듈이 있는 디렉토리의 경로를 추가하고 싶다면 다음과 같은 방법을 사용한다.

```python
#sys 모듈을 추가해야 함.
import sys

sys.path.append('C:/myLib')
```

## 모듈의 사용 <a name="m4"/>

### 다른 파이썬 파일 내에서의 사용

다른 파이썬 파일 내에서 모듈을 사용하는 방법은 모듈 사용의 가장 기본적인 방법으로 `import` 를 사용하는 것이다. 모듈을 import 하는 방법은 다음과 같다.

1. 현재 디렉토리에서 모듈을 불러오는 방법

```python
#원래는 <module name>.py 이지만 사용할 때는 <module name>만 적어준다.
import <module name>
```

2. 다른 경로의 모듈을 불러오는 방법

```python
#파일 경로 : C:/Users/jeon/Documents/pythonLib (파이썬이 설치된 경로와 다른 경우)
import C:/Users/jeon/Documents/pythonLib/myMod
```

3. 모듈 내에서 특정 요소만 불러오는 경우

```python
#<module name>으로부터 특정 요소를 가져온다.
from <module name> import <function>|<variable>|*
```

4. 여러개의 모듈을 불러오는 경우

```python
#2개 이상의 모듈을 불러온다.
import <module name1>, <module name2>, ...
```

위와 같은 방법으로 모듈을 불러오고 실제 다른 파이썬 코드에서 사용하는 방법은 다음과 같다.

#### 예제: ch4modEx.py, ch4Ex1, ch4Ex2 - 모듈 사용 예제 코드

```python
#ch4modEx.py
def sum(a,b):
    result = a + b
    return result
    
def minus(a,b):
    if a > b:
        print("a가 b보다 큽니다.")
        result = a-b
    else:
        print("b가 a보다 큽니다.")
        result = b-a
    return result
```

```python
#ch4Ex1.py
import ch4modEx

ch4modEx.sum(3,5) # 결과값 : 8
ch4modEx.minus(4,5) # 결과값 : 'b가 a보다 큽니다.' 출력, 1
```

```python
#ch4Ex2.py
from ch4modEx import minus

minus(10,3) # 결과값 : 'a가 b보다 큽니다.' 출력, 7
```

ch4Ex1.py의 코드는 **모듈 자체를 import** 했을 때 사용 방법이다. 모듈 자체를 import를 하게 되면 `<module name>.`이 붙고 클래스, 함수, 변수 등이 사용한다. 반면 ch4Ex2.py의 코드는 모듈 내에서 minus라는 함수를 import 하는 즉, 위에서 3번의 방법인 모듈 내에서 **특정 요소만 불러오는 방법** 이다. 사용하지 않는 불필요한 요소 없이 모듈 내에서도 필요한 요소들만 불러와서 사용하면 되기 때문에 편리하고 모듈 import 하는 시간을 많이 소모하지 않는다.

### 모듈 자체 사용

모듈을 자체적으로 사용하는 방법은 스크립트 전체를 바로 실행시키는 방법을 말한다. 여기서 중요한 것은 `__name__` 의 사용이다. `__name__`이 `__main__` 이면 해당 모듈 안에서 참인 부분을 실행하고 거짓일 경우에는 import 만 일어날 뿐 아무 일도 일어나지 않게 하는 방법이다.

#### 예제 ch4modEx2.py - 모듈 안에서 __name__ 사용 코드

```python
#ch4modEx2.py
def sum(a,b):
    result = a + b
    return result
    
def minus(a,b):
    if a > b:
        print("a가 b보다 큽니다.")
        result = a-b
    else:
        print("b가 a보다 큽니다.")
        result = b-a
    return result

#명령 프롬프트에서 직접 실행시켰을 때 참
if __name__ == "__main__":
    a = int(input("a : "))
    b = int(nput("b : "))
    print(sum(a,b))
```

만약에 명령 프롬프트에서 직접 ch4modEx2.py를 실행시켰을 때 조건문이 참이 되어 참인 조건문 안에서의 문장을 실행하고 import 만 시켰을 경우에는 거짓이 되어 import 할 뿐 아무 문장도 실행하지 않는다.

## 참고자료 <a name="m5"/>

1. 점프 투 파이썬, https://wikidocs.net/29
2. 코딩 도장, https://dojang.io/mod/page/view.php?id=1147
