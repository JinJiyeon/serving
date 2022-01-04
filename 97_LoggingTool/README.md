# 97_Logging Tool

## 기본정보

- 목적 : 로그는 대표적인 스트림 데이터이다. 로그를 남기는 Logging Tool의 구조와 사용법을 알자

- 기간 : 2021/10/27 ~ 2021/10/28

- Ref : 

  - [데이터 분석가 입장에서 로그 분석하기 : 로그 분석의 방향성과 주제](https://techblog.woowahan.com/2536/)

  - [개발자+데분가+기획자 입장에서 로그 기획하기 : 전사. 로그 룰과 틀 잡기](https://speakerdeck.com/devinjeon/jamag-ndc19-joheun-rogeuran-mueosinga-joheun-rogeureul-wihae-goryeohaeya-hal-geosdeul?slide=10)

  - [개발자 입장에서 로그 시스템 구축하기 : 로그 아키텍쳐](https://www.slideshare.net/ssuser380e9c/ndc18-2-95522893)

  - [로그 관련 글 모음](https://zzsza.github.io/data/2021/06/13/data-event-log-definition/)

  - [파이썬 logging docs](https://zzsza.github.io/data/2021/06/13/data-event-log-definition/)

  - [자바 logback docs](http://logback.qos.ch)



## 로그 데이터 수집

로그는 클라이언트 서버에서도, 백엔드 서버에서도 수집할 수 있다 

핸드폰, PC >>> 웹 브라우저, 앱 >>> 프론트서버 >>> 백엔드 서버

- 웹 브라우저, 앱에서 클릭을 수집하는 경우

  - 웹 브라우저에 이벤트를 모아뒀다가 로그를 전송한다

  - 클릭할 때마다 서버를 호출해서 기록을 쌓는 건 비효율적이므로 버퍼링처럼 모아둔다. 
  - 단점은 갑자기 앱이나 브라우저가 꺼졌을 때 로그가 몇일 뒤에 들어온다

- Api 서버

  - 어떤 파라미터로 호출했는지 기록한다

- 클라이언트 서버

  - 실제 api로 다시 redirect하면서, 로그를 기록한다

  - HTTP 302

  - Cf) 프록시 : 서로 다른 서버에서 요청을 대신 전달해준다

    

### 로그 데이터 저장

- Elastic search

  - 로그데이터의 경향성만 보는 경우에 ES에 저장한다
  - 키바나로 시각화할 수 있어서 편하다

  - 조인이 어려워서 로그를 활용한 프로젝트를 하기에 제약이 있다

- HDFS

  - 로그데이터를 활용해 복잡한 서비스를 하려는 경우에 HDFS에 저장한다.
  - rsync 같은 프로그램을 이용하여 HDFS에 옮긴다



## python : logging

- 초급
  - DEBUG, INFO, WARNING, ERROR, CRITICAL.. 등의 상태에 따라 출력할 메세지를 설정할 수 있다.

```python
# myapp.py
import logging
import mylib

def main():
    logging.basicConfig(filename='myapp.log', level=logging.INFO)
    logging.info('Started')
    mylib.do_something()
    logging.info('Finished')

if __name__ == '__main__':
    main()

# mylib.py
import logging

def do_something():
    logging.info('Doing something')

# myapp.log
INFO:root:Started
INFO:root:Doing something
INFO:root:Finished
```

- 고급

  - 로깅은 로거, 핸들러, 포매터로 구성된다.

  - 로거는 모듈 별로 생성할 것을 권장한다. 로거는 이름, 상태별 메시지를 설정한다

  - 핸들러는 로그 데이터를 어떻게 출력한지 정한다. 로그 파일, 표준 출력, 전자 메일 등..

    *예를 들어, 응용 프로그램은 모든 로그 메시지를 로그 파일로 보내고, 에러(error)와 그 이상의 모든 로그 메시지를 표준 출력으로 보내고, 모든 심각한 에러(critical) 메시지를 전자 메일 주소로 보낼 수 있습니다. 이 시나리오에서는 각 처리기가 특정 심각도의 메시지를 특정 위치로 보내는 3개의 개별 처리기가 필요합니다.*

  - 포매터는 로그 메시지의 순서, 구조, 내용을 지정한다

```python
import logging

# create logger
logger = logging.getLogger('simple_example')
logger.setLevel(logging.DEBUG)

# create console handler and set level to debug
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)

# create formatter
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# add formatter to ch
ch.setFormatter(formatter)

# add ch to logger
logger.addHandler(ch)

# 'application' code
logger.debug('debug message')
logger.info('info message')
logger.warning('warn message')
logger.error('error message')
logger.critical('critical message')

>>>
2005-03-19 15:10:26,618 - simple_example - DEBUG - debug message
2005-03-19 15:10:26,620 - simple_example - INFO - info message
2005-03-19 15:10:26,695 - simple_example - WARNING - warn message
2005-03-19 15:10:26,697 - simple_example - ERROR - error message
2005-03-19 15:10:26,773 - simple_example - CRITICAL - critical message
```

- 변수 넣기

```python
import logging
logging.warning('%s before you %s', 'Look', 'leap!')

>>> 
WARNING:root:Look before you leap!
```



### Django와 logging

view 함수 아래에 로거를 추가한다. 기본 로거는 base.py에 셋팅해둔다.

```python
# views.py
import logging

# Get an instance of a logger
# 여기가 아니라 my_view 내여도 된다
logger = logging.getLogger(__name__)

def my_view(request, arg1, arg):
    ...
    if bad_mojo:
        # Log an error message
        logger.error('Something went wrong!')
# settings > base.py
LOGGING = {
    (... 생략 ...)
    'handlers': {
        'console': {
            'level': 'INFO',
            'filters': ['require_debug_true'],
            'class': 'logging.StreamHandler',
        },
        'django.server': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'django.server',
        },
        'mail_admins': {
            'level': 'ERROR',
            'filters': ['require_debug_false'],
            'class': 'django.utils.log.AdminEmailHandler'
        },
        'file': {
            'level': 'INFO',
            'filters': ['require_debug_false'],
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': BASE_DIR / 'logs/mysite.log',
            'maxBytes': 1024*1024*5,  # 5 MB
            'backupCount': 5,
            'formatter': 'standard',
        },
    },
    (... 생략 ...)
}
```





## Java : logback

- 초급

```java
package chapters.introduction;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld1 {

  public static void main(String[] args) {

    Logger logger = LoggerFactory.getLogger("chapters.introduction.HelloWorld1");
    logger.debug("Hello world.");

  }
}

>>>
20:49:07.962 [main] DEBUG chapters.introduction.HelloWorld1 - Hello world.
```

- 고급

  - 파이썬이 로거, 핸들러, 포매터로 구성되었듯이 자바도 로거, 어펜더, 레이아웃으로 구성된다.

  - 로거의 이름은 com.foo, com.foo.Bar로 표기하며 부모 자식 관계가 있다

  - 어펜더는 로그를 어디에 출력할지 정한다. 

    *an output destination is called an appender. Currently, appenders exist for the console, files, remote socket servers, to MySQL, PostgreSQL, Oracle and other databases, JMS, and remote UNIX Syslog daemons.*

```java
// 얘를 오버라이드하면 로깅을 좀더 자세하게 구축할 수 있다
package org.slf4j; 
public interface Logger {

  // Printing methods: 
  public void trace(String message);
  public void debug(String message);
  public void info(String message); 
  public void warn(String message); 
  public void error(String message); 
}

import ch.qos.logback.classic.Level;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

ch.qos.logback.classic.Logger logger = 
        (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("com.foo");
logger.setLevel(Level. INFO);

Logger barlogger = LoggerFactory.getLogger("com.foo.Bar");

// This request is enabled, because WARN >= INFO
logger.warn("Low fuel level.");

// This request is disabled, because DEBUG < INFO. 
logger.debug("Starting search for nearest gas station.");

// barlogger, named "com.foo.Bar", inherit from "com.foo" 
// Thus, the following request is enabled because INFO >= INFO. 
barlogger.info("Located nearest gas station.");

// This request is disabled, because DEBUG < INFO. 
barlogger.debug("Exiting gas station search");
```