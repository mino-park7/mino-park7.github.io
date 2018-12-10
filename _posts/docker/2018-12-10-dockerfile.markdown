---
layout: "post"
title: "Dockerfile 작성방법 및 읽는 법"
date: "2018-12-10 11:06"
category: docker
tag: [docker, study]
---

# Dockerfile의 작성 방법 및 읽는 법
## rasa-nlu dockerfile
```Dockerfile
FROM python:3.6-slim

ENV RASA_NLU_DOCKER="YES" \
    RASA_NLU_HOME=/app \
    RASA_NLU_PYTHON_PACKAGES=/usr/local/lib/python3.6/dist-packages

# Run updates, install basics and cleanup
# - build-essential: Compile specific dependencies
# - git-core: Checkout git repos
RUN apt-get update -qq \
    && apt-get install -y --no-install-recommends build-essential git-core openssl libssl-dev libffi6 libffi-dev curl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

WORKDIR ${RASA_NLU_HOME}

# use bash always
RUN rm /bin/sh && ln -s /bin/bash /bin/sh

COPY . ${RASA_NLU_HOME}

RUN pip install -r alt_requirements/requiements_full.txt

RUN pip install -e .

RUN apt-get update -qq \
    && apt-get install -y --no-install-recommends wget \
    && wget -P data/ https://s3-eu-west-1.amazonaws.com/mitie/total_word_feature_extractor.dat \
    && apt-get remove -y wget \
    && apt-get autoremove -y

RUN pip install https://github.com/explosion/spacy-models/releases/download/en_core_web_md-2.0.0.0.tar.gzz --no-cache-dir > /dev/null \
    && python -m spacy link en_core_web_md en \
    && pip install https://github.com/explosion/spacy_models/releases/download/de_core_news_sm-2.0.0/de_core_news_sm-2.0.0.tar.gz --no-cache-dir > /dev/null \
    && python -m spacy link de_core_news_sm de

COPY sample_configs/config_spacy_duckling.yml ${RASA_NLU_HOME}/config.yml

#VOLUME ["/app/projects", "/app/logs", "/app/data"]

EXPOSE 5000

ENTRYPOINT ["./entrypoint.sh"]
CMD ["start", "-c", "config.yml", "--pach", "/app/projects"]
```

### FROM
- 빌드 할 이미지가 어떤 이미지를 기반으로 하고있는지를 나타냄
- rasa-nlu docker의 경우 python 이미지, 특히 3.6-slim 버전을 기반으로 만들어진 이미지임

### ENV
- 해당 이미지의 환경변수를 지정해주는 옵션, 이는 `RUN, CMD, ENTRYPOINT`에 적용
- rasa-nlu의 경우 `$RASA_NLU_HOME, $RASA_NLU_DOCKER, $RASA_NLU_PYTHON_PACKAGES`을 지정해 줌

### RUN
- FROM에서 설정한 이미지 위에서 명령을 실행하는 것, 쉽게 말해 shell sciprt와 같다고 보면 됨
- 예제를 자세히 풀어서 설명하자면
  - `apt-get update -qq` : ubuntu에서 패키지 목록 업데이트, -qq는 quiet 2단계를 의미, logging을 없애줌
  - `&& apt-get ~` : 해당 이미지의 app을 구동하기 위한 우분투 패키지 설치
  - `rm /bin/sh && ln -s /bin/bash /bin/sh` : 해당 이미지의 shell 실행파일을 bash로 강제
  - `pip install -r alt_requirements/requiements_full.txt` : rasa-nlu_full 구동에 필요한 파이썬 패키지 목록이 `alt_requirements/requiements_full.txt` 에 저장되어 있고, 해당 pip 명령어를 통해 설치
  - 이후 RUN : spacy 모듈 설치를 위한 명령어들

### COPY
- 파일을 이미지에 추가함, 뒤에 설명할 ADD와는 약간 다름
- 사용법 :
  - `COPY <복사할 파일> <이미지에서 파일이 복사될 경로>` : 예제에서는 `.` 즉 컨텍스트에 있는 모든 파일을 `${RASA_NLU_HOME}`에 복사함
  - `<복사할 파일>`은 컨텍스트 아래만 가능, `../foo/bar`, `/home/minhopark2115/` 이런거 불가능
  - 인터넷 url 사용불가
  - `<이미지에서 파일이 복사될 경로>`는 언제나 절대경로
  - 여러 파일을 복사할 때, `dockerignore`에 설정한 내역은 제외 됨

### WORKDIR
- `RUN, CMD, ENTRYPOINT`의 명령어가 실행되는 디렉터리를 설정함
- 예제에서는 `${RASA_NLU_HOME}`로 설정

### EXPOSE
- host와 연결할 포트 번호를 설정
- rasa-nlu는 5000번 사용

### ENTRYPOINT
- 컨테이너 (이미지의 인스턴스)가 시작되었을 때 실행되는 명령어
- Dockerfile에서 한번만 사용 가능
- 예제에서는 COPY 명령어를 통해 넘겨진 `entrypoint.sh`라는 쉘스크립트를 실행
  - `entrypoint.sh` 내용 : 간단하게 설명하면 rasa_nlu.server 실행하는 내용
  - 바로 뒤에 있는 `CMD`의 내용을 매개변수로 받는 듯

### CMD
- `ENTRYPOINT`와 비슷 하게 쓰이지만, 앞에 `ENTRYPOINT`가 있다면, 거기에 매개변수를 넘겨주는 목적으로 쓰임
- 위의 `entrypoint.sh`에 `start, -c, ...`등의 매개별수를 넘겨준다

### VOLUME
- 해당 디렉터리의 내용을 컨테이너에 저장하지 않고 호스트에 저장
- `#VOLUME ["/app/projects", "/app/logs", "/app/data"]` 처럼 배열로 선언 가능
- 근데 host어디에 저장될 것인지는 설정 불가능
- host의 특정 디렉터리와 연결하려면 `docker run -v <host directory>:<container directory>` 사용
- 
