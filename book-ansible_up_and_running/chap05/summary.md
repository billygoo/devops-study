# 매자닌 소개: 테스트 애플리케이션
## 매자닌
- django 프레임웍을 활용한 CMS framework이다.
  - https://github.com/stephenmcd/mezzanine

## 패브릭 배포 스크립트
- SSH로 연결해 Shell 명령어를 실행할 수 있는 라이브러리
  - https://www.fabfile.org/

## WSGI
- 웹 서버 게이트웨이 인터페이스(WSGI, Web Server Gateway Interface)는 웹서버와 웹 애플리케이션의 인터페이스를 위한 파이썬 프레임워크다.
  - https://ko.wikipedia.org/wiki/%EC%9B%B9_%EC%84%9C%EB%B2%84_%EA%B2%8C%EC%9D%B4%ED%8A%B8%EC%9B%A8%EC%9D%B4_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4


## Gunicorn
- UNIX에서 WSGI HTTP 서버로 `Green Unicorn`이라 불린다. 
  - https://gunicorn.org/
- Ruby의 [Unicorn](https://en.wikipedia.org/wiki/Unicorn_(web_server)) 프로젝트에서 활용된 `pre-fork worker` 모델을 사용
  - https://docs.gunicorn.org/en/latest/design.html#server-model
  - 하나의 마스터 프로세스가 워크 프로세스들을 관리한다. 
  - 하지만 마스터 프로세스는 개별 워커 프로세스들이 동작은 모른다. 
  - 모든 리퀘스트나 리스폰스는 개별 워크 프로세스들이 다룬다.
- 여러 웹 프레임워크와 호환이 되며, 간단히 구현되어 서버 리소스가 가볍고, 빠르다. 
- NGINX와 상호 보완적인 기능들을 가지고 있다고 한다. (?)

