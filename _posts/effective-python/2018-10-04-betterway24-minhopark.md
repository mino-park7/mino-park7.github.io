---
layout: "post"
title: "이펙티브 파이썬 - 24. 객체를 범용으로 생성하려면 @classmethod 다형성을 이용하자"
date: "2018-10-04 01:30"
category: effective python Study
tag: [python, effective,]
---

# BetterWay24. 객체를 범용으로 생성하려면 @classmethod 다형성을 이용하자

- 파이썬에서 객체가 다형성을 지원할 뿐만 아니라 클래스도 다형성을 잘 지원한다.

---

- 다형성은 계층 구조에 속한 여러 클래스가 자체의 메서드를 독립적인 버전으로 구현하는 방식
- 다형성을 이용하면 여러 클래스가 같은 인터페이스나 추상 기반 클래스를 충족하면서도 다른 기능을 제공할 수 있다.

- ex) 맵리듀스 구현을 작정할 때 입력 데이터를 표현할 공통 클래스가 필요하다고 하자.
   - 다음은 서브클래스에서 정의해야하는 `read`메서드가 있는 입력 데이터 클래스다.


```python
class InputData(object):

    def read(self):
        raise NotImplementedError
```


```python
# 디스크에 있는 파일에서 데이터를 읽어오도록 구현한 InputData의 서브클래스
class PathInputData(InputData):

    def __init__(self, path):
        #super.__init__()
        self.path = path

    def read(self):
        return open(self.path).read()
```

- `PathInputData` 같은 `InputData` 서브클래스가 몇 개든 있을 수 있고, 각 서브 클래스에서는 처리할 바이트 데이터를 반환하는 표준 인터페이스인 `read`를 구현할 것이다.
- 다른 `InputData` 서브클래스는 네트워크에서 데이터를 읽어오거나 데이터의 압축을 해제하는 기능 등을 할 수 있다.
- 표준 방식으로 입력 데이터를 처리하는 맵리듀스 작업 클래스에도 비슷한 추상 인터페이스가 필요하다.


```python
class Worker(object):
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError
```


```python
# 적용하려는 특정 맵 리듀스 함수를 구현한 Worker의 구체 서브클래스다(간단한 줄바꿈 카운터)
class LineCountWorker(Worker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')

    def reduce(self, other):
        self.result += other.result
```

- 이렇게 구현하면 잘 동작할 것처럼 보이지만 결국 커다란 문제에 직면
- 이 모든 코드 조각을 무엇으로 연결할 것인가? 적절히 인터페이스를 설계하고 추상화한 클래스들이지만 일단 객체를 생성한 후에나 유용. 무엇으로 객체를 만들고 맵리듀스를 조율할까?

- 가장 간단한 방법은 헬퍼 함수로 직접 객체를 만들고 연결하는 것
   - 다음은 디렉터리의 내용을 나열하고 그 안에 있는 각 파일로 `PathInputData` 인스턴스를 생성하는 코드


```python
import os

def generate_inputs(data_dir):
    for name in os.listdir(data_dir):
        yield PathInputData(os.path.join(data_dir, name))
```


```python
# 다음으로 generate_inputs 함수에서 반환한 InputData 인스턴스를 사용하는 LineCountWorker 인스턴스를 생성
def create_workers(input_list):
    workers = []
    for input_data in input_list:
        workers.append(LineCountWorker(input_data))
    return workers
```


```python
from threading import Thread
# map 단계를 여러 스레드로 나눠서 이 Worker 인스턴스를 실행. 그런다음 reduce를 반복적으로 호출해서 결과를 최종값 하나로 합친다
def excute(workers):
    threads = [Thread(target=w.map) for w in workers]
    for thread in threads: thread.start()
    for thread in threads: thread.join()

    first, rest = workers[0], workers[1:]
    for worker in rest:
        first.reduce(worker)
    return first.result

```


```python
# 마지막으로 단계별로 실행하려고 mapreduce 함수에서 모든 조각을 연결
def mapreduce(data_dir):
    inputs = generate_inputs(data_dir)
    workers = create_workers(inputs)
    return excute(workers)
```


```python
# 테스트용 입력 파일로 이 함수를 실행해보면 잘 동작
from tempfile import TemporaryDirectory

def write_test_files(tmpdir):

    for i in range(1000):
        with open(os.path.join(tmpdir, str(i) + '.txt'), 'w') as f:
            f.write('*\n' * i)

with TemporaryDirectory() as tmpdir:
    write_test_files(tmpdir)
    result = mapreduce(tmpdir)

print('There are', result, 'lines')

```

    There are 499500 lines


- 무엇이 문제인가? 큰 문제는 `mapreduce` 함수가 전혀 범용적이지 않다는 점
   - 다른 `InputData`나 `Worker` 서브클래스를 작성한다면 `generate_inputs`, `create_workers`, `mapreduce`함수를 알맞게 다시 작성해야 함

---

- 이 문제는 결국 객체를 생성하는 범용적인 방법의 필요성으로 귀결
- 다른 언어에서는 이 문제를 생성자 다형성으로 해결함
   - 이 방식을 따르려면 각 `InputData` 서브클래스에서 맵리듀스를 조율하는 헬퍼 메서드가 범용적으로 사용할 수 있는 특별한 생성자를 제공해야함
   - 문제는 파이썬이 단일 생성자 메서드 `__init__` 만을 허용한다는 점
   - 격국 모든 `InputData` 서브클래스가 호환되는 생성자를 갖춰야 한다는 터무니 없는 요구사항 임

- 이 문제를 해결하는 가장 좋은 방법은 `@classmethod` 다형성을 이용하는 것
    - `@classmethod` 다형성은 생성된 객체가 아니라 전체 클래스에 적용된다는 점만 빼면 `InputData.read`에 사용한 인스턴스 메서드의 다형성과 똑같다

- 이 발상을 맵리듀스 관련 클래스에 적용해보자. 여기서는 공통 인터페이스를 사용해 새 `InputData` 인스턴스를 생성하는 범용 클래스 메서드로 `InputData` 클래스를 확장한다


```python
class GenericInputData(object):
    def read(self):
        raise NotImplementedError

    @classmethod
    def genrate_inputs(cls, config):
        raise NotImplementedError
```

- `generate_inputs` 메서드는 `GenericInputData`를 구현하는 서브클래스가 해석할 설정 파라미터들을 담은 딕셔너리를 받는다.
- 다음 코드에서는 입력 파일들을 얻어올 디렉터리를 config로 알아낸다.


```python
class PathInputData(GenericInputData):
    def __init__(self, path):
        self.path = path

    def read(self):
        return open(self.path).read()

    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name))

```

- 비슷하게 `GenericWorker` 클래스에 `create_workers` 헬퍼를 작성한다.
- 여기서는 `input_class` 파라미터(`GenericInputData`의 서브클래스여야함)로 필요한 입력을 만들어냄
- `cls()`를 범용 생성자로 사용해서 `GenericWorker`를 구현한 서브클래스의 인스턴스를 생성


```python
class GenericWorker(object):
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError

    @classmethod
    def create_workers(cls, input_class, config):
        workers = []
        for input_data in input_class.generate_inputs(config):
            workers.append(cls(input_data))
        return workers
```

- 위의 `input_class.generate_inputs` 호출이 바로 여기서 보여주려는 클래스 다형성이다.
- 또한 `create_workers`가 `__init__` 메서드를 직접 사용하지 않고 `GenericWorker`를 생성하는 또 다른 방법으로 `cls`를 호출함을 알 수 있다

---

- `GenericWorker`를 구현할 서브클래스는 부모 클래스만 변경하면 된다.


```python
class LineCountWorker(GenericWorker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')

    def reduce(self, other):
        self.result += other.result
```


```python
# mapreduce 함수를 완전히 범용적으로 재작성
def mapreduce(worker_class, input_class, config):
    workers = worker_class.create_workers(input_class, config)
    return excute(workers)
```

- 테스트용 파일로 새로운 작업 클래스 객체를 실행하면 이전에 구현한 것과 같은 결과가 나옴
- 차이는 `mapreduce` 함수가 범용적으로 동작하려고 더 많은 파라미터를 요구한다는 점


```python
with TemporaryDirectory() as tmpdir:
    write_test_files(tmpdir)
    config = {'data_dir': tmpdir}
    result = mapreduce(LineCountWorker, PathInputData, config)
    print(result)
```

    499500


- 이제 `GenericInputData`와 `GenericWorker`의 다른 서브클래스를 원하는 대로 만들어도 glue code를 작성할 필요가 없다

## 핵심 정리

- 파이썬에서는 클래스별로 생성자를 한개(`__init__` 메서드) 만 만들수 있다.
- 클래스에 필요한 다른 생성자를 저으이하려면 `@classmethod`를 사용하자.
- 구체 서브클래스들을 만들고 연결하는 범용적인 방법을 제공하려면 클래스 메서드 다형성을 이용하자
