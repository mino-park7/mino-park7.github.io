---
layout: "post"
title: "파이썬 디자인 패턴 - 생성 디자인 패턴 - 팩터리 메서드 패턴"
date: "2018-08-13 23:08"
category: python Study
tag: [python, designpattern, 생성디자인패턴]
---


## 3. 팩터리 메서드 패턴

***

* **팩터리 메서트** 패턴은 객체 생성을 요청할 때 하위 클래스가 인스턴스화할 클래스를 지정할 수 있게끔 고안된 패턴
* 생성하려는 클래스가 어떤 것인지 모를 때도 활용할 수 있다.

***

* 특정 게임판을 생성하는 하위 클래스의 바탕이 되는 추상 보드 클래스 생성
* 게임에 따라 각 말이 자체 클래스를 갖게 하고싶음

***

게임판을 인스턴스화하고 출력하는 최상위 코드

![main코드](http://mino-park7.github.io/assets/images/2018/08/main코드.png)

* 이 함수는 프로그램 모든 버전에서 동일. 각 게임판을 생성하고 콘솔에 출력

***
### gameboard1.py
![AbstractBoard class](http://mino-park7.github.io/assets/images/2018/08/abstractboard-class.png)

* `___str___()` 메서드를 통해 게임판 내부 표현을 문자열로 변환
* 게임판은 한 글자로 된 문자열 리스트로 표현
  - 말이없는 칸은 None
  - console() 함수는 배경색 위에 배치할 말이 담긴 문자열을 반환
* AbstractFormBuilder와 마찬가지로 `AbstractBoard` 클래스에 `abc.ABCMeta` 메타클래스를 전달해 추상 클래스를 만들 수 있지만, 본 코드에서는 `raise NotImplementedError()` 를 사용 (하위 클래스에서 implement하지 않으면 에러 발생)

***
![ChechersBoard class](http://mino-park7.github.io/assets/images/2018/08/chechersboard-class.png)

* `AbstractBoard` 를 구현한 concrete class인 `CheckersBoard()` class 는 10x10 국제 규격 체커판을 생성
* `poppulate_board()` 메서드는 하드코딩된 클래스(`BlackDraought`, `WhiteDraught`)를 활용하기 때문에 **팩터리 메서드**가 아니다!
* 이 메서드를 어떻게 **팩터리 메서트**로 만들지 보여주기 위해서 첨부

***

![ChessBoard class 1](http://mino-park7.github.io/assets/images/2018/08/chessboard-class-1.png)

* `ChessBoard` 클래스도 마찬가지로 `poppulate_board()` 메서드가 어떻게 **팩터리 메서드**로 바뀌는지 보여주기 위해 첨부

***

![Piece class](http://mino-park7.github.io/assets/images/2018/08/piece-class.png)

* `Piece()` 클래스는 말에 대한 기반 클래스
* `str` 변수만 사용할 수 도 있지만, 그렇게 하면 어떤 객체가 말인지 판단할 수 없음
* `__slots__=()`를 사용해 인스턴스가 어떤 데이터도 가실 수 없도록 보장 (2.6에서 살펴볼 예정)

***

![Piece concrete classes](http://mino-park7.github.io/assets/images/2018/08/piece-concrete-classes.png)

* 모든 말 클래스가 공통적으로 활용하는 패턴을 볼 수 있음
  - 각 말은 변경 불가능한 `Piece`의 하위 클래스(자체가 str의 하위 클래스)
  - 각 말을 표현하는 유니코드 글자가 담긴 1글자짜리 문자열로 초기화
* 소규모 하위 클래스가 **14** 개가 있음
* 중복이 굉장히 많다!

***
### gameboard2.py
![create_piece using populate_board](http://mino-park7.github.io/assets/images/2018/08/create-piece-using-populate-board.png)

* 위의 gameboard1.py 의 `populate_board()` 와 다르게 `create_piece()` 팩터리 메서드를 활용한다.
* `creae_piece()` 함수는 인자에 따라 적절한 타입의 객체(`BlackDraought` 나 `WhiteDraught` 같은)를 반
* `ChessBoard` class에도 똑같이 존재 (색깔과 말 이름을 인자로 전달)

***

![create_piece factory method with eval()](http://mino-park7.github.io/assets/images/2018/08/create-piece-factory-method-with-eval.png)

* 이 **팩터리 메서트**는 내장 함수인 `eval()`을 활용해 클래스 인스턴스를 생성
  - ex) 인자가 "knight", "black"이면 `eval("{}Chess{}")~`을 통해 `BlackChessKnight()`가 됨
  - 하지만 `eval()`은 위험 (어떤것이든 생성 가능)

***

![code generation in gameboard2.py](http://mino-park7.github.io/assets/images/2018/08/code-generation-in-gameboard2-py.png)

* gameboard1.py 의 소규모 하위 클래스 **14**개를 단 한 블록의 코드로 클래스를 모두 생성했다.
  - `itertools.chain()` : 하나 이상의 `iterable`을 받아 전체를 순회할 수 있는 단일 `iterable`을 반환
  - 여기서는 두 개의 `iterable`을 전달
    1. 검은색과 하얀색 체커 말의 유니코드 포인트에 대한 2-튜플
    2. 검은색과 하얀색 체스 말에 대한 range 객체
* 각 유니코드 포인트에 대해 한 글자로 된 문자열을 생성해 이 문자의 유니코드 명칭에 따라 클래스 이름을 생성 ("black chess knight"는 `BlackChessKnight`가 된다)
* `exec()` 을 사용
  - 너무 위험함

***
### gameboard3.py
![gameboard3.py tuple decl](http://mino-park7.github.io/assets/images/2018/08/gameboard3-py-tuple-decl.png)
![gameboard3 CheckersBoard class](http://mino-park7.github.io/assets/images/2018/08/gameboard3-checkersboard-class.png)

* gameboard2.py 의 populate_board() 메서드와는 다르게, 잘못 입력하기 쉬운 문자열 리터럴 대신 상수 값을 사용해 말과 색깔을 지정한다.
* 각 말을 생성하기 위해 새로운 `creae_piece()` **팩터리**를 활용

***
![gameboard3.py AbstractBoard class](http://mino-park7.github.io/assets/images/2018/08/gameboard3-py-abstractboard-class.png)

* 위는 `ChessBoard`와 `CheckersBoard`가 상속 할 추상 클래스이다
  - 두 클래스는 `create_piece()`를 상속한다.
    - `create_piece()`는 ~Board 클래스의 `__classForPiece` dict를 참조하여 kind, color를 받아 말 이름을 반환한다.\
    - gameboard2.py 와 다르게 `exec()`이나 `eval()`을 사용하지 않고 안전하게 동적으로 생성할 수 있다.

***

![gameboard3.py code generate](http://mino-park7.github.io/assets/images/2018/08/gameboard3-py-code-generate.png)

* gameboard2.py 와 다르게 `eval()`이나 `exec()`을 활용하는 대신 안전한 방법 사용
  - 일단 `code`에서 글자와 그에 대한 명칭을 가지고 있다면 `maek_new_method()` 메서드를 호출해 해로운 함수 (`new()`)를 생성
  - 내장함수인 `type()`을 활용해 새로운 클래스를 생성
    - `type()`을 이용해 클래스를 생성하려면
      - 클래스 타입
      - 기반 클래스들에 대한 튜플 (Piece 하나만 넘김)
      - 클래스 애트리뷰트에 대한 딕셔너리
    - 방금 생성한 `new()` 함수를 `__new__` 메서드로 설정
  - 현재 모듈(`sys.modules[__name__]`)에 새로 생성한 클래스를 name이라는 애트리뷰트로 추가하기 위해 `setattr()`를 호출
* `make_new_method()` 는 `new()` 함수를 생성
* `super()`는 활용 할 수 없음, 왜냐면 `new()`가 생성될 시기에는 `super()`가 접근할 클래스가 아직 없기 때문
***
### gameboard4.py

![gameboard4.py](http://mino-park7.github.io/assets/images/2018/08/gameboard4-py.png)

* gameboard3.py 에서 `setattr()` 를 사용한 것과 달리, 전역변수가 저장된 딕셔너리의 참조를 얻어, name에 있는 값이 key이고 새로 생성한 Class가 값인 새 아이템을 추가 -> `setattr()`와 똑같이 동작
* `new()` 메서드 생성의 경우에도, `lambda`를 활용하여 해결
* `create_piece()` 가 gameboard4.py 의 **팩터리 메서드**이다.
  - 사용되는 상수값은 gameboard3.py 와 같지만, 클래스 객체의 dict 대신 내장 함수인 `globals()`를 통해 얻은 dict에서 해당 클래스를 동적으로 찾는다.
  - 이렇게 얻은 클래스 객체를 바로 호출해서 원하는 말 인스턴스를 얻는다.
