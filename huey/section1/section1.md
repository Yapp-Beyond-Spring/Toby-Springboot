
> ✅ **강의 개요**
> 
> 스프링 부트가 어떻게 동작하는지, 어떤 원리로 만들어졌는지를 이해하기 위해 노력
> 
> 스프링 부트가 스프링을 어떻게 이용해서 동작하는지를 코드를 통해서 설명할 수 있는 방법

# 스프링 부트 소개

## 스프링 부트 정의

스프링 부트는 스프링을 기반으로 실무 환경에 사용 가능한 수준의 독립실행형 애플리케이션을 복잡한 고민 없이 빠르게 작성할 수 있게 도와주는 여러가지 도구의 모음이다.

2023년 안에 Spring6을 지원하는 Spring Boot 3.0.0이 나올 듯

## 스프링 부트 핵심 목표

- 매우 빠르고 광범위한 영역의 스프링 개발 경험을 제공
- 강한 주장을 가지고 즉시 적용 가능한 기술 조합을 제공하면서, 필요에 따라 원하는 방식으로 손쉽게 변형 가능
- 프로젝트에서 필요로 하는 다양한 비기능적인 기술 제공 (내장형 서버, 보안, 메트릭, 상태 체크, 외부 설정 방식 등)
- 코드 생성이나 XML 설정을 필요로 하지 않음

## Containerless

> 컨테이너리스 웹 애플리케이션 아키텍처

- Serverless라고 볼 수 있음.
- 스프링을 Containerless로 만든게 스프링 부트!

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/e1393774-1f67-42c7-8bf8-1b860b561e15)

client는 component에 요청을 보내 받은 응답으로 사용자와 인터랙션한다.

component는 독립적으로 존재할 수 없고 container가 필요하다.

container는 하나 또는 다수의 component를 관리해주며 생명주기, 메모리 등에 관여한다. 또한 client의 요청을 어떤 component가 처리할지 판단해서 전달하는 라우팅(또는 매핑) 역할도 수행한다.

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/397f6c7a-19b6-4a67-a8b0-2cdc73c5a157)

자바에서 web component는 servlet이며, container는 servlet container이다.

제일 유명한 servlet container는 톰캣이다.

이 모습이 가장 전통적인 90년대의 자바의 웹 애플리케이션 구조이다.

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/846e72d0-7be2-4c85-b406-4f415d96e4e3)

Spring container를 추가하여 servlet에 들어온 웹 요청을 전달받아 처리하고 다시 servlet을 통해 응답을 하는 구조이다.

따라서 Spring container 역시 어떤 Bean이 처리해야할지 판단하여 전달하는 역할을 수행한다.

즉, 스프링을 쓰기 위해서는 servlet container를 띄워야 하는데 이게 간단하지 않다.

해당 작업은 servlet container 종류별로 다 다르며 노력에 비해 꼭 필요한 학습이 아니었다.

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/46026a66-3e18-4bce-b6f6-64742fc31cc2)

따라서 servlet container를 없앤 방식이 containerless이다.

이를 독립실행형 애플리케이션(standalone application) 방식이라 한다.

## Opinionated

> 개발자는 개발에만 집중 (자기 주장이 강한, 자기 의견을 고집하는, 독선적인)
> 
- 사용 기술과 의존 라이브러리 결정
    - 업계에서 검증된 스프링 생태계 프로젝트, 표준 기술, 오픈소스 기술의 종류와 의존관계, 사용 버전을 정해줌.
    - 각 기술을 스프링에 적용하는 방식(DI 구성)과 디폴트 설정값 제공
- 유연한 확장
    - 디폴트 구성을 커스터마이징 하는 자연스러운 방법 제공
    - 스프링 부트가 스프링을 사용하는 방식을 이해한다면 언제라도 스프링 부트를 제거하고 원하는 방식으로 재구성 가능
    - 스프링 부트처럼 기술과 구성을 간편하게 제공하는 나만의 모듈 작성 가능

## 강의의 목표

- 스프링 부트로 만든 스프링 애플리케이션의 기술과 구성 정보를 직접 확인할 수 있다.
- 적용 가능한 설정 항목을 파악할 수 있다.
- 직접 만든 빈 구성 정보를 적용하고, 그에 따른 변화를 분석 할 수 있다.
- 스프링 부트의 기술을 꼼꼼히 살펴볼 수 있다.