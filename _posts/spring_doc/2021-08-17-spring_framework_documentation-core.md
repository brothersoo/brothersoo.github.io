---
title: "Spring Framework Documentation(한글번역) - Core"
categories:
  - Spring doc
tags:
  - spring
toc: true
permalink: /spring_doc/:title
---

<br>

해당 포스트는 spring에 대한 이해 및 영어 공부를 위해 [spring framework의 공식 문서 v5.3.9](https://docs.spring.io/spring-framework/docs/current/reference/html/)를 번역한 포스팅입니다.

오역이 많을 수 있으니 발견하신다면 댓글로 말씀 부탁드리겠습니다 감사합니다!

> Rod Johnson, Juergen Hoeller, Keith Donald, Colin Sampaleanu, Rob Harrop, Thomas Risberg, Alef Arendsen, Darren Davison, Dmitriy Kopylenko, Mark Pollack, Thierry Templier, Erwin Vervaet, Portia Tung, Ben Hale, Adrian Colyer, John Lewis, Costin Leau, Mark Fisher, Sam Brannen, Ramnivas Laddad, Arjen Poutsma, Chris Beams, Tareq Abedrabbo, Andy Clement, Dave Syer, Oliver Gierke, Rossen Stoyanchev, Phillip Webb, Rob Winch, Brian Clozel, Stephane Nicoll, Sebastien Deleuze, Jay Bryant, Mark Paluch
>
> Copyright © 2002 - 2021 Pivotal, Inc. All Rights Reserved.
>
> Copies of this document may be made for your own use and for distribution to others, provided that you do not charge any fee for such copies and further provided that each copy contains this Copyright Notice, whether distributed in print or electronically.

<br>
<br>
<br>

**5.3.9**

이번 문서는 Spring Framework의 필수적인 모든 기술들에 대해 다룹니다.

그중에서도 가장 핵심은 Inversion of Control(IoC, 역전의 제어)입니다. Spring Framework의 IoC container은 Spring의 Aspect-Oriented Programming(AOP) 기술의 포괄적인 범위를 따라 성장했습니다(? A thorough treatment of the Spring Framework’s IoC container is closely followed by comprehensive coverage of Spring’s Aspect-Oriented Programming (AOP) technologies.). Spring Framework는 쉬운 컨셉과, Java enterprise programming 내 AOP 요구사항의 핵심적인 80%를 훌륭히 준비한 자체 AOP 프레임워크를 가지고 있습니다.

AspectJ(현재로써 기능면에서 가장 풍부하고, 확실히 Java enterprise 세계관에서 가장 완성도있는 AOP 구현체)를 통한 Spring의 완성도 확인 또한 제공됩니다.


# 1. The IOC Container

이번 절은 Spring의 역전의 제어(이하 IoC) container에 대해 다룹니다.

## 1.1. Spring IoC Container와 Beans에 대한 설명

이번 절은 Spring Framework의 `역전의 제어` 원칙의 구현을 다룹니다. IoC는 의존성 주입(Dependency injection, 이하 DI)라고도 불립니다. 이는 객체들이 그들의 의존성(: 함께 작동할 다른 객체)을 오직

생산자 인자(Constructor argument), factory method로의 인자, 혹은 생성되거나 factory method로부터 반환된 객체에 설정된 속성값

을 통해서만 정의하는 과정입니다. Container가 의존성을 가질 Bean 객채를 생성 후 해당 의존성을 주입합니다. 이 과정은 근본적으로

클래스의 direct construction와 `Service Locator pattern`

과 같은 메커니즘을 통해 bean이 역으로 그의

인스턴스화와 의존성의 위치

를 역으로 제어하는 것입니다(역전의 제어라는 이름을 보면 알다시피).

<br>

`org.springframework.beans`와 `org.springframework.context` 패키지는 Spring Framework의 IoC Container를 위한 기초입니다. [`BeanFactory`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/beans/factory/BeanFactory.html) interface는 어떤 타입의 객체든 관리하기 용이한 고급 설정 메커니즘을 제공합니다. [`Application Context`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/context/ApplicationContext.html)는 `BeanFactory`의 sub-interface입니다. `Application Context`에는 다음과 같은 항목들이 추가되었습니다.
- Spring의 AOP 특성들과의 쉬운 통합
- Internationalization에 사용될 메세지 자원 관리
- 이벤트 게시(Event publisher)
- `WebApplicationContext`와 같은 웹 애플리케이션에서 사용할 수 있는 `Application-layer specific contexts`

다시말해, `BeanFactory`는 설정 프레임워크와 기본 기능을 제공하고, `ApplicationContext`는 엔터프라이즈에 특화된 기능이 추가된 것입니다. `ApplicationContext`는 `BeanFactory`의 완전한 상위 대체자(superset)이고, 이번 절에서 Spring의 IoC Container를 설명하는데 항상 사용될 것입니다. `ApplicationContext` 대신 `BeanFactory`의 사용에 대한 설명은 [The `BeanFactory`](#beans-beanfactory)를 확인하세요.

Spring에서 당신의 애플리케이션의 중추를 이루고 Spring IoC container에 의해 관리될 객체들은 `Bean`이라 불립니다. `Bean`은 Spring IoC container에 의해 인스턴스화(이하 구현)되고, 조립되고, 관리될 객체입니다. 그렇지 않다면, bean은 당신의 애플리케이션 안에 있는 수많은 평범한 객체중 하나에 불과하지 않을 것입니다. Bean과 그에 대한 의존성들(dependencies)은 container이 사용하는 configuration metadata에 반영되어있습니다.

## 1.2. Container 개요

`org.springframework.context.ApplicationContext` 인터페이스는 Spring IoC container를 대표하고, bean들을 구현, 구성, 그리고 조립하는 역할을 가지고 있습니다. Container는 configuration metadata를 읽음으로써 어떤 객체를 구현, 구성, 그리고 조립할지에 대한 설명을 얻습니다. Configuration metadata는 XML, Java 어노테이션, 혹은 Java 코드로 쓰여져있습니다. 이 metadata는 당신의 애플리케이션을 구성하는 객체들과 그 객체들 사이의 깊은 내부 의존성을 표현할 수 있게 해줍니다.

`ApplicationContext` 인터페이스의 몇몇 구현체들은 Spring로부터 제공받습니다. Stand-alone 애플리케이션에서는 [`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) 혹은 [`FileSystemXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html)를 생성하는 경우가 많습니다. 지금까지 configuration metadata를 정의하는데 보편적으로 쓰여왔던 형식이었지만, Java annotation 혹은 code를 사용한 metadata 형식을 지원하도록 해주는 약간의 XML 설정을 제공하면 container가 Java annotation 혹은 code를 metadata 형식으로 사용할 수 있도록 설정할 수 있습니다.

대부분의 애플리케이션 시나리오에서는 explicit 사용자 코드는 하나 이상의 인스턴스를 인스턴스화하는데 요구되지 않습니다. 에를 들어, 웹 애플리케이션 시나리오에서 애플리케이션의 'web.xml' 안의  간단한 여덟줄 가량의 boilerplate(찍어내듯이 재사용하는 간단한 코드) web descriptor XML면 충분합니다([웹 애플리케이션을 위한 간편한 ApplicationContext 인스턴스화](#context-create)를 확인하세요). 만약 당신이 [Spring Tools for Eclipse](https://spring.io/tools)(Eclipse 개발 환경)를 사용한다면, 몇번의 마우스 클릭 혹은 단축키 입력으로 이 간단한 boilerplate configuration을 생성할 수 있습니다.

아래 다이어그램은 Spring이 어떻게 작동하는지 상위 단계의 관점으로 보여줍니다. 당신의 애플리케이션 클래스들은 configuration metadata로 결합되어있으므로, `ApplicationContext`가 생성되고 초기화된다면, 완전히 구성되고, 실행 가능한 시스템, 혹은 애플리케이션을 사용할 준비가 된 것입니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/spring_doc/core/container-magic.png)\
*그림 1. The Spring IoC container*

### 1.2.1. Configuration Metadata

이전에 보았던 다이어그램에서 볼 수 있듯이, Spring IoC 컨테이너는 configuration metadata(이하 설정 메타데이터)를 받습니다. 이 설정 메타데이터는 당신이 애플리케이션 개발자로써 당신의 애플리케이션을 어떻게 인스턴스화 할것인지, 구성할 것인지, 그리고 그 안의 객체들을 어떻게 조립할 것인지를 Spring container에게 알려줍니다.

설정 메타데이터는 전통적으로, 이 장에서 Spring IoC 컨테이너의 주요 컨셉과 요소들을 전달하는데 주로 사용될 XML 형식으로 제공됩니다.

> **_INFO:_** XML 기반 메타데이터는 configuration metadata를 작성하는데 허용된 유일한 형식이 아닙니다. Spring IoC 컨테이너는 이 설정 메타데이터가 실제 어떤 형식으로 쓰여있는지와 아무런 관계가 없습니다. 요즘 많은 개발자들은 그들의 Spring 애플리케이션에 [Java 기반 설정](#beans-java)을 선택합니다.

Spring 컨테이너를 다른 형식의 메타데이터로 사용하는 것에 대한 정보를 얻으려면 다음을 확인하세요:

- [어노테이션 기반 설정](#beans-annotation-config): Spring 2.5는 어노테이션 기반 설정 메타데이터 지원을 발표했습니다.
- [Java 기반 설정](#beans-java): Spring 3.0부터, Spring JavaConfig 프로젝트가 제공하는 다양한 기능들이 Spring 프레임워크의 핵심이 되었습니다. 게다가, XML 파일 대신 Java를 사용하여 당신의 애플리케이션 클래스들에다 외부적으로 beans를 정의할 수 있습니다. 이 새로운 기능슬을 사용하려면 [`@Configuration`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html), [`@Bean`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html), [`@Import`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html), 그리고 [`@DependsOn`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html) 어노테이션을 확인하세요.

이와같은 bean 정의문은 당신의 애플리케이션을 구축하는 실제 객체들과 대응합니다. 통상적으로 우리는 계층적 객체들, 데이터 접근 객체들(DAOs), Struts(아파치 스트럿츠) `Action` 인스턴스들, Hibernate `SessionFactories`와 같은 인프라 구조 객체들, JMS(Java Message Service) `Queues` 등을 정의합니다. 도메인 객체를 생성하고 불러오는것은 보통 DAO들과 비즈니스 로직의 책임이므로, 일반적으로 우리는 컨테이너 안에 세부적인 도메인 객체를 설정하지 않습니다. 그러나, IoC container의 제어권 밖에서 생성된 객체들을 AspectJ를 통해 Spring의 integration으로 생성할 수 있습니다. [AspectJ를 통해 Spring에서 도메인 객체들을 의존성 주입하기](#aop-atconfigurable)를 확인해보세요.

아래의 예제는 기초적인 XML 기반의 설정 메타데이터를 보여줍니다:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="..."> 1️⃣ 2️⃣
        <!-- 해당 bean의 collaborators와 설정들을 입력 -->
    </bean>

    <bean id="..." class="...">
        <!-- 해당 bean의 collaborators와 설정들을 입력 -->
    </bean>

    <!-- bean에 대한 더 많은 정의들을 이어서 입력 -->

</beans>
```

&nbsp;&nbsp;&nbsp;`1️⃣` `id` 속성은 개별의 bean 정의 식별하는 문자열입니다.

&nbsp;&nbsp;&nbsp;`2️⃣` `class` 속성은 bean의 유형을 정의하고, 클래스의 완전한 이름을 사용합니다.

`id`의 값은 함께 사용될 객체들을 나타냅니다. 위 예제는 함께 사용하는 객체들을 참조하는 XML를 포함하지 않고 있습니다. 해당 정보는 [의존성(Dependencies)](#beans-dependencies)에서 확인하세요.

### 1.2.2. Container 인스턴스화하기

`ApplicationContext` 생성자로 전달될 한개 이상의 경로는,

로컬 파일 시스템, Java `CLASSPATH`, 등과 같은 다양한 외부 자원으로부터

컨테이너가 설정 메타데이터를 불러올 수 있도록 하는 문자열 자원입니다.

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

> **_INFO:_**: IoC 컨테이너에 대한 학습을 진행하였다면, URI 구문에 정의된 위치로부터 InputStream을 읽는데 편리한 메커니즘을 제공하는 스프링의 `Resource`([자원들(Resources)](#resources)에 설명된) 추상화에 관심이 생길 수 있습니다. 보통, `Resources` 경로들은 [Application Contexts와 Resource Paths](#resources-app-ctx)에 설명된 것과 같이 application contexts를 생성하는데 사용됩니다.

아래의 예제는 서비스 계층 객체`(services.xml)`들의 설정 파일을 나타냅니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 서비스들 -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- 해당 bean의 추가적인 collaborator들과 설정들을 입력 -->
    </bean>

    <!-- 서비스를 위한 추가 bean들을 정의 -->

</beans>
```

아래의 예제는 데이터 접근 객체들`(daos.xml)` 파일을 나타냅니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- 해당 bean의 추가적인 collaborator들과 설정들을 입력 -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- a해당 bean의 추가적인 collaborator들과 설정들을 입력 -->
    </bean>

    <!-- 서비스를 위한 추가 bean들을 정의 -->

</beans>
```

위 예제를 보면, 서비스 계층은 `PetStoreServiceImpl` 클래스와 두개의 데이터 객체들,`JpaAccountDao`와 `JpaItemDao`,로 이루어져있습니다(JPA 객체-관계형 매핑 표준을 기반으로 한). `property name` 원소는 JavaBean 속성을 나타내고, `ref` 원소는 두개의 협력 객체들 사이의 의존성을 나타냅니다. 객체의 의존성 설정에 세부적인 내용은 [Dependencies](#beans-dependencies)를 확인하세요.

**XML 기반의 설정 메타데이터 구성하기**
...

### 1.2.3. Container 사용하기

## 1.3. Bean 개요

### 1.3.1. Bean 이름 짓기

### 1.3.2. Bean 인스턴스화하기

## <a name="beans-dependencies"></a>1.4. Dependencies (의존성)

### 1.4.1. Dependency Injection (의존성 주입)

### 1.4.2. Dependencies와 Configuration 세부 내용

### 1.4.3. `depends-on` 사용하기

### 1.4.4 Lazy-initialized Beans (지연 초기화)

### 1.4.5. Autowiring Collaborators

### 1.4.6. Method Injection

## 1.5. Bean Scopes (Bean 영역)

### 1.5.1. The Singleton Scope (싱글톤 영역)

### 1.5.2. The Prototype Scope (프로토타입 영역)

### 1.5.3. 프로토타입 빈 의존성을 가진 싱글톤 빈

### 1.5.4. Request, Session, Application, 그리고 WebSocket Scopes

### 1.5.5. Custom Scopes




<!-------------------------------------------------------------->



## <a name="beans-annotation-config"></a>1.9. 어노테이션 기반 컨테이너 설정


## <a name="beans-java"></a>1.12. 자바 기반의 컨테이너 설정

### <a name="context-create"></a>1.15.5. 웹 애플리케이션을 위한 간편한 ApplicationContext 인스턴스화

## <a name="beans-beanfactory"></a>1.16. `Bean Factory`



### <a name="aop-atconfigurable"></a>5.10.1. AspectJ를 이용한 Spring에서 도메인 객체 의존성 주입하기


# <a name="resources"></a>2. Resources

## <a name="resources-app-ctx"></a>2.8 Application Contexts와 Resource Paths