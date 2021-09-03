---
title: "Spring Framework Documentation(한글번역) - Core"
categories:
  - Spring doc
tags:
  - spring
toc: true
toc_sticky_top: true
permalink: /spring_doc/:title
---

<style>
    .blue-text-box {
        color: #242424;
        background-color: #AEC0BF;
        border-radius: 3px;
        padding: 20px;
        height: auto;
    }
    .blue-text-box a {
        color: #00473E;
    }
    .blue-text-box code {
        background-color: #4C4C69;
        color: white;
    }
    .info-box {
        color: #242424;
        background-color: #AEC0AE;
        border-radius: 3px;
        padding: 20px;
    }
    .info-box a{
        color: #00473E;
    }
    .info-icon {
        text-align: center;
        width: 10%;
        float: left;
    }
    .icon-div {
        display: table-call;
        vertical-align: middle;
    }
    .icon {
        content: url({{ site.url }}{{ site.baseurl }}/assets/images/spring_doc/info-icon.png);
        width: 50%;
        height: 50%;
    }
    .info-content {
        width: 90%;
        float: right;
    }
</style>

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

이번 장은 Spring의 역전의 제어(이하 IoC) container에 대해 다룹니다.

## 1.1. Spring IoC Container와 Beans에 대한 설명

이번 장은 Spring Framework의 `역전의 제어` 원칙의 구현을 다룹니다. IoC는 의존성 주입(Dependency injection, 이하 DI)라고도 불립니다. 이는 객체들이 그들의 의존성(: 함께 작동할 다른 객체)을 오직

생산자 인자(Constructor argument), factory method로의 인자, 혹은 생성되거나 factory method로부터 반환된 객체에 설정된 속성값

을 통해서만 정의하는 과정입니다. Container가 의존성을 가질 Bean 객채를 생성 후 해당 의존성을 주입합니다. 이 과정은 근본적으로

클래스의 direct construction와 `Service Locator pattern`

과 같은 메커니즘을 통해 bean이 역으로 그의

인스턴스화와 의존성의 위치

를 역으로 제어하는 것입니다(역전의 제어라는 이름을 보면 알다시피).

<br>

`org.springframework.beans`와 `org.springframework.context` 패키지는 Spring Framework의 IoC Container를 위한 기초입니다. [`BeanFactory`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/beans/factory/BeanFactory.html) interface는 어떤 타입의 객체든 관리하기 용이한 고급 구성 메커니즘을 제공합니다. [`Application Context`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/context/ApplicationContext.html)는 `BeanFactory`의 sub-interface입니다. `Application Context`에는 다음과 같은 항목들이 추가되었습니다.
- Spring의 AOP 특성들과의 쉬운 통합
- Internationalization에 사용될 메세지 자원 관리
- 이벤트 게시(Event publisher)
- `WebApplicationContext`와 같은 웹 애플리케이션에서 사용할 수 있는 `Application-layer specific contexts`

다시말해, `BeanFactory`는 configuration 프레임워크와 기본 기능을 제공하고, `ApplicationContext`는 엔터프라이즈에 특화된 기능이 추가된 것입니다. `ApplicationContext`는 `BeanFactory`의 완전한 상위 대체자(superset)이고, 이번 장에서 Spring의 IoC Container를 설명하는데 항상 사용될 것입니다. `ApplicationContext` 대신 `BeanFactory`의 사용에 대한 설명은 [The `BeanFactory`](#beans-beanfactory)를 확인하세요.

Spring에서 당신의 애플리케이션의 중추를 이루고 Spring IoC container에 의해 관리될 객체들은 `Bean`이라 불립니다. `Bean`은 Spring IoC container에 의해 인스턴스화(이하 구현)되고, 조립되고, 관리될 객체입니다. 그렇지 않다면, bean은 당신의 애플리케이션 안에 있는 수많은 평범한 객체중 하나에 불과하지 않을 것입니다. Bean과 그에 대한 의존성들(dependencies)은 container이 사용하는 configuration metadata에 반영되어있습니다.

## 1.2. Container 개요

`org.springframework.context.ApplicationContext` 인터페이스는 Spring IoC 컨테이너를 대표하고, bean들을 구현, 구성, 그리고 조립하는 역할을 가지고 있습니다. 컨테이너는 configuration metadata를 읽음으로써 어떤 객체를 구현, 구성, 그리고 조립할지에 대한 설명을 얻습니다. Configuration metadata는 XML, Java 어노테이션, 혹은 Java 코드로 쓰여져있습니다. 이 metadata는 당신의 애플리케이션을 구성하는 객체들과 그 객체들 사이의 깊은 내부 의존성을 표현할 수 있게 해줍니다.

`ApplicationContext` 인터페이스의 몇몇 구현체들은 Spring로부터 제공받습니다. Stand-alone 애플리케이션에서는 [`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) 혹은 [`FileSystemXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.9/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html)를 생성하는 경우가 많습니다. 지금까지 configuration metadata를 정의하는데 보편적으로 쓰여왔던 형식이었지만, Java annotation 혹은 code를 사용한 metadata 형식을 지원하도록 해주는 약간의 XML configuration을 제공하면 container가 Java annotation 혹은 code를 metadata 형식으로 사용할 수 있도록 설정할 수 있습니다.

대부분의 애플리케이션 시나리오에서는 explicit 사용자 코드는 하나 이상의 인스턴스를 인스턴스화하는데 요구되지 않습니다. 에를 들어, 웹 애플리케이션 시나리오에서 애플리케이션의 'web.xml' 안의  간단한 여덟줄 가량의 boilerplate(찍어내듯이 재사용하는 간단한 코드) web descriptor XML면 충분합니다([웹 애플리케이션을 위한 간편한 ApplicationContext 인스턴스화](#context-create)를 확인하세요). 만약 당신이 [Spring Tools for Eclipse](https://spring.io/tools)(Eclipse 개발 환경)를 사용한다면, 몇번의 마우스 클릭 혹은 단축키 입력으로 이 간단한 boilerplate configuration을 생성할 수 있습니다.

아래 다이어그램은 Spring이 어떻게 작동하는지 상위 단계의 관점으로 보여줍니다. 당신의 애플리케이션 클래스들은 configuration metadata로 결합되어있으므로, `ApplicationContext`가 생성되고 초기화된다면, 완전히 구성되고, 실행 가능한 시스템, 혹은 애플리케이션을 사용할 준비가 된 것입니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/spring_doc/core/container-magic.png)\
*그림 1. The Spring IoC container*

### 1.2.1. Configuration Metadata (구성 메타데이터)

이전에 보았던 다이어그램에서 볼 수 있듯이, Spring IoC 컨테이너는 configuration metadata(이하 구성 메타데이터)를 받습니다. 이 구성 메타데이터는 당신이 애플리케이션 개발자로써 당신의 애플리케이션을 어떻게 인스턴스화 할것인지, 구성할 것인지, 그리고 그 안의 객체들을 어떻게 조립할 것인지를 Spring container에게 알려줍니다.

구성 메타데이터는 전통적으로, 이 장에서 Spring IoC 컨테이너의 주요 컨셉과 요소들을 전달하는데 주로 사용될 XML 형식으로 제공됩니다.

<div class="info-box" style="height: 120px;">
    <div class="info-icon">
        <div class="icon-div"><div class="icon"></div></div>
    </div>
    <div class="info-content">
        XML 기반 메타데이터는 구성 메타데이터를 작성하는데 허용된 유일한 형식이 아닙니다. Spring IoC 컨테이너는 이 구성 메타데이터가 실제 어떤 형식으로 쓰여있는지와 아무런 관계가 없습니다. 요즘 많은 개발자들은 그들의 Spring 애플리케이션에 <a href="#beans-java">Java 기반 configuration</a>을 선택합니다.
    </div>
</div>

<br>

Spring 컨테이너를 다른 형식의 메타데이터로 사용하는 것에 대한 정보를 얻으려면 다음을 확인하세요:

- [어노테이션 기반 configuration](#beans-annotation-config): Spring 2.5는 어노테이션 기반 구성 메타데이터 지원을 발표했습니다.
- [Java 기반 configuration](#beans-java): Spring 3.0부터, Spring JavaConfig 프로젝트가 제공하는 다양한 기능들이 Spring 프레임워크의 핵심이 되었습니다. 게다가, XML 파일 대신 Java를 사용하여 당신의 애플리케이션 클래스들에다 외부적으로 beans를 정의할 수 있습니다. 이 새로운 기능들을 사용하려면 [`@Configuration`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html), [`@Bean`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html), [`@Import`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html), 그리고 [`@DependsOn`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html) 어노테이션을 확인하세요.

Spring configuration은 컨테이너가 관리해야하는 하나, 혹은 보통 하나 이상의 bean definition으로 이루어져 있습니다. XML 기반의 구성 메타데이터는 이 bean들을 상위 단계 `<beans/>` 원소 내부에 `<beans/>` 원소로써 구성합니다. Java configuration은 보통 `@Configuration` 클래스 안에서 `@Bean` 어노테이션이 붙은 메서드들을 사용합니다.

이와같은 bean 정의문은 당신의 애플리케이션을 구축하는 실제 객체들과 대응합니다. 통상적으로 우리는 계층적 객체들, 데이터 접근 객체들(DAOs), Struts(아파치 스트럿츠) `Action` 인스턴스들, Hibernate `SessionFactories`와 같은 인프라 구조 객체들, JMS(Java Message Service) `Queues` 등을 정의합니다. 도메인 객체를 생성하고 불러오는것은 보통 DAO들과 비즈니스 로직의 책임이므로, 일반적으로 우리는 컨테이너 안에 세부적인 도메인 객체를 설정하지 않습니다. 그러나, IoC container의 제어권 밖에서 생성된 객체들을 AspectJ를 통해 Spring의 integration으로 생성할 수 있습니다. [AspectJ를 통해 Spring에서 도메인 객체들을 의존성 주입하기](#aop-atconfigurable)를 확인해보세요.

아래의 예제는 기초적인 XML 기반의 구성 메타데이터를 보여줍니다:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="..."> 1️⃣ 2️⃣
        <!-- 해당 bean의 collaborators와 구성들을 입력 -->
    </bean>

    <bean id="..." class="...">
        <!-- 해당 bean의 collaborators와 구성들을 입력 -->
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

컨테이너가 구성 메타데이터를 불러올 수 있도록 하는 문자열 자원입니다.

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

<br>

<div class="info-box" style="height: 150px">
    <div class="info-icon">
        <div class="icon"></div>
    </div>
    <div class="info-content">
        IoC 컨테이너에 대한 학습을 진행하였다면, URI 구문에 정의된 위치로부터 InputStream을 읽는데 편리한 메커니즘을 제공하는 스프링의 `Resource`(<a href="#resources">자원들(Resources)</a>에 설명된) 추상화에 관심이 생길 수 있습니다. 보통, `Resources` 경로들은 <a href="#resources-app-ctx">Application Contexts와 Resource Paths</a>에 설명된 것과 같이 application contexts를 생성하는데 사용됩니다.
    </div>
</div>

<br>

아래의 예제는 서비스 계층 객체`(services.xml)`들의 구성 파일을 나타냅니다.

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
        <!-- 해당 bean의 추가적인 collaborator들과 구성들을 입력 -->
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
        <!-- 해당 bean의 추가적인 collaborator들과 구성들을 입력 -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- a해당 bean의 추가적인 collaborator들과 구성들을 입력 -->
    </bean>

    <!-- 서비스를 위한 추가 bean들을 정의 -->

</beans>
```

위 예제를 보면, 서비스 계층은 `PetStoreServiceImpl` 클래스와 두개의 데이터 객체들,`JpaAccountDao`와 `JpaItemDao`,로 이루어져있습니다(JPA 객체-관계형 매핑 표준을 기반으로 한). `property name` 원소는 JavaBean 속성을 나타내고, `ref` 원소는 두개의 협력 객체들 사이의 의존성을 나타냅니다. 객체의 의존성 설정에 세부적인 내용은 [Dependencies](#beans-dependencies)를 확인하세요.

#### XML 기반의 구성 메타데이터 구성하기
...

### 1.2.3. Container 사용하기

## 1.3. Bean 개요

Spring IoC 컨테이너는 하나 이상의 bean을 관리합니다. 이 bean들은 당신이 컨테이너에 제공한 구성 메타데이터를 통해 생성됩니다(예를 들어, XML `<bean/>` 정의들 형식과 같이.)

컨테이너 자체 안에서, 이러한 bean definitions는 `BeanDefinition` 객체로 표현되고, 아래와 같은 메타데이터를 가지고 있습니다(다른 정보들과 함께).

* Package-qualified 클래스명: 대게, bean이 정의되는 실제 구현 클래스.
* 컨테이너 내에서 bean이 어떤 행동을 할지 정하는 Bean 행위 구성 원소들(스코프, 생명주기 콜백들 등).
* bean이 그의 역할을 하기 위해 필요한 다른 bean으로의 참조. 이러한 챀조들은 collaborators, 혹은 dependencies라고도 불립니다.
* 새로 생성되는 객체들에 적용된 다른 구성 설정. 예를 들어, pool의 크기 제한, 혹은 connection pool을 관리하는 bean에서 사용된 connection의 수.

해당 메타데이터는 각각의 bean definition을 생성하는 성질의 집합으로 번역됩니다. 아래의 표는 이 성질들에 대한 설명입니다.

**표 1. Bean definition**

|성질|설명 링크...|
|---|---|
|클래스 (Class)|[Bean 인스턴스화하기 (Instantiating Beans)](#beans-factory-class)|
|이름 (Name)|[Bean 이름짓기 (Naming Beans)](#beans-beanname)|
|범위 (Scope)|[Bean 범위 (Bean Scopes)](#beans-factory-scopes)|
|생성자 인자 (Constructor arguments)|[의존성 주입 (Dependency Injection)](#beans-factory-collaborators)|
|성질들 (Properties)|[의존성 주입 (Dependency Injection)](#beans-factory-collaborators)|
|자동 연결 모드 (Autowiring mode)|[협력자 자동연결 (Autowiring Collaborators)](#beans-factory-autowire) |
|지연 초기화 모드 (Lazy initialization mode)|[지연 초기화된 Bean (Lazy-initialized Beans)](#beans-factory-lazy-init)|
|초기화 방법 (Initialization method)|[초기화 콜백 (Initialization Callbacks)](#beans-factory-lifecycle-initializingbean)|
|파괴 방법 (Destruction method)|[파괴 콜백 (Destruction Callbacks)](#beans-factory-lifecycle-disposablebean)|

특정 bean을 생성하는 방법에 대한 설명을 가지고 있는 bean definition에 대해 추가적인 정보로, `ApplicationContext` 구현체들은 사용자들에 의해 컨테이너 외부에서 생성되어 존재하는 객체들의 등록을 허용합니다. 이는 BeanFactory의 `DefaultListableBeanFactory` 구현체를 반환하는 `getBeanFactory()` 메서드를 통해 ApplicationContext의 BeanFactory에 접근하는 것으로 행해집니다. `DefaultListableBeanFactory`는 `registerSingleton(..)`과 `registerBeanDefinition(..)` 메서드를 통해 해당 등록을 지원합니다. 그러나 일반적으로 애플리케이션들은 표준 bean definition 메타데이터를 통해 정의된 beans으로만 작동합니다.

<div class="info-box" style="height: 200px;">
    <div class="info-icon">
        <div class="icon-div"><div class="icon"></div></div>
    </div>
    <div class="info-content">
        Bean 메타데이터와 수동으로 제공된 싱글톤 객체들은, 컨테이너가 autowiring과 다른 내부 조사(introspect) 단계들에서 컨테이너가 그들을 원활히 이해(reason about)하기 위해 최대한 빨리 등록되어야 합니다. 어느 정도에서는 이미 존재하는 메타데이터와 싱글톤 인스턴스들을 오버라이딩 하는 것이 지원되지만, 런타임 환경에서 새로운 bean 등록은 공식적으로 지원되지 않고, 동시적 접근 예외(concurrent access exceptions), bean 컨테이너 내의 일관적이지 않은 상태, 혹은 둘 다를 야기할 수 있습니다.
    </div>
</div>


### <a name="beans-beanname"></a>1.3.1. Bean 이름 짓기

모든 bean은 하나 이상의 식별자를 가지고 있습니다. 이 식별자들은 bean을 관리하는 컨테이너 내에서 유일해야 합니다. 그러나, 하나 이상이 존재한다면, 나머지 식별자들은 별칭(alias)으로써 고려될 수 있습니다.

XML 기반 구성 메타데이터에서, `id` 속성, `name` 속성, 혹은 둘 다를 사용해 bean 식별자를 지정할 수 있습니다. `id` 속성은 정확히 한개의 아이디를 지정할 수 있도록 합니다. 이 이름들은 관습적으로 알파벳만을 포함하지만('myBean', 'someService', 등), 특수 문자 또한 포함할 수 있습니다. 만약 bean에게 다른 별칭들을 부여하고 싶다면, 쉼표(`,`), 세미콜론`(;)`, 혹은 공백으로 구분하여 `name` 속성에 지정할 수 있습니다. 이에 대한 역사 이야기를 잠깐 하자면, 스프링 3.1 미만의 버전에서는 `id` 속성이 사용 가능한 문자를 제한하는 `xsd:ID` 타입으로써 정의되었습니다. 3.1 부터 이는 `xsd:string` 타입으로 정의되었습니다. `id`의 유일성은 더이상 XML parser가 아닌 컨테이너에 의해 강제된다는 것을 참고하세요.

반드시 bean에 `name` 혹은 `id`를 제공해야만 하는것은 아닙니다. 만약 `name` 혹은 `id`가 명시적으로 제공되지 않는다면, 컨테이너가 bean을 위해 유일한 이름을 생성할 것입니다. 그러나, `ref` 원소 혹은 Service Locator 형식의 조회를 통해 bean을 이름으로 참조하고 싶다면, 반드시 이름을 직접 제공해야 합니다. 이름을 직접 제공하지 않는 것은 [내부 bean](#beans-inner-beans)의 사용, 그리고 [autowiring collaborators](#beans-factory-autowire)을 위해 존재합니다.

<div class="blue-text-box">
<p style="text-align: center; font-size: 20px">Bean 이름 협약(naming conventions)</p>

<p>해당 협약에서 bean의 이름을 짓는 것은 표준 Java convention에서 인스턴스 변수를 이름짓는 방식과 동일합니다. 이는 bean 이름들은 알파벳 소문자로 시작하고, camel-cased이라는 것을 의미합니다. 예시로 `accountManger` `accountService` `userDao` `loginController`, 등을 들 수 있습니다.</p>

<p>Bean에 일관된 이름을 부여하는 것은 구성 파일을 쉽게 읽고 이해하는데 도움을 줍니다. 또한 Spring AOP를 사용하고 있다면, 이름응로 관계된 bean 집합의 실직적 구현체(AOP advice)를 제공하는데 큰 도움을 줍니다.</p>
</div>

<br>

<div class="info-box" style="height:180px">
    <div class="info-icon">
        <div class="icon"></div>
    </div>
    <div class="info-content">
        Classpath에서 component scanning이 진행될 때, Spring은 이전에 설명된 규칙을 가지고 이름없는 components의 이름을 생성합니다: simple class name의 첫 알파벳만 소문자로 변경한 것. 하지만, (보통 일어나지 않는) 하나 이상의 문자를 가지고, 첫번째와 두번째 문자들이 모두 대문자인 특별한 상황에서는 원래의 대소문자 형식이 유지됩니다. 이는 `java.beans.Introspector.decapitalize`에서 정의된 규칙과 동일합니다 (Spring이 여기서 사용하는 것과).
    </div>
</div>

#### Bean definition 밖에서 Bean 별칭짓기

Bean definition 자체 내에서, `id`속성으로 지정된 하나의 이름과, `name` 속성으로 지정된 원하는 만큼의 이름들의 조합을 통해, bean에게 하나 이상의 이름을 제공할 수 있습니다. 

그러나, bean이 실제로 정의되어있는 곳에 별칭을 부여하는 것이 항상 올바르지는 않을 수 있습니다. 다른 곳에서 정의된 bean의 별칭을 부여해야 하는 상황이 있을 때가 있습니다. 이는 보통, 각각 고유한 객체 정의문을 가지고 있는 서브-시스템을 여러개 가지고 있는 거대한 시스템의 configuration이 각 서브-시스템마다 있는 경우입니다. XML 기반 구성 메타데이터에서는, `<alias/>` 요소로 이를 해결할 수 있습니다.

```xml
<alias name="fromName" alias="toName"/>
```

이 경우에, 위 별칭 정의가 이루어진 후에는 `fromName`이라고 이름지어진 bean(같은 컨테이너 내의)이 `toName`으로 지칭될 수 있습니다.

예를 들어, 서브-시스템 A의 구성 메타데이터는 `subsystemA-dataSource`라는 이름으로 DataSource를 참조할 수 있습니다. 또, 서브-시스템 B의 구성 메타데이터는 `subsystemB-datasource`라는 이름으로 DataSource를 참조할 수 있습니다. 위 서브-시스템 A와 B를 모두 사용하는 메인 애플리케이션을 구성할 때, 이 메인 애플리케이션은 `myApp-dataSource`라는 이름으로 DataSource를 참조합니다. 이와 같이 다른 세가지 이름으로 하나의 객체를 참조하고 싶다면, 구성 메타데이터에 아래와 같은 별칭 정의를 추가하면 됩니다.

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

이제 각 component와 메인 애플리케이션은, 유일하고, 같은 bean을 참조하면서도 다른 정의와 충돌을 일으키지 않을 것임을 보장하는 이름으로, DataSource를 참조할 수 있습니다.

<div class="blue-text-box">
<p style="text-align: center; font-size: 20px">Java-configuration</p>

<p>만약 Javaconfiguration을 사용한다면, 별칭을 제공하기 위해 `@Bean` 어노테이션을 사용할 수 있습니다. 자세한 내용은 <a href="#beans-java-bean-annotation"><code>@Bean</code> 어노테이션 사용하기</a>에서 확인하세요</p>
</div>

### <a name="beans-factory-class"></a>1.3.2. Bean 인스턴스화하기

Bean definition은 하나 이상의 객체를 생성하는데 필수적인 생성 문서입니다. 컨테이너는 이름 있는 bean이 그의 이름으로 요청될 때 이 문서를 읽고, 실제 객체를 생성(혹은 획득)하기 위해 bean definition에 의해 캡슐화된 구성 메타데이터를 사용합니다.

만약 XML 기반 구성 메타데이터를 사용한다면, `<bean/>` 요소의 `class` 속성에 인스턴스화 될 객체의 타입(혹은 클래스)를 지정합니다. 이 `class` 속성(내부적으로 `BeanDefinition`의 `Class` 프로퍼티)은 보통 필수적입니다. (예외 경우는 [인스턴스 팩토리 메서드로 인스턴스화하기](#beans-factory-class-instance-factory-method), 그리고 [Bean definition 상속화](#beans-child-bean-definitions)에서 확인하세요.) `Class` 프로퍼티는 다음 두가지 경우 중 하나의 경우로써 지정될 수 있습니다.

- 보통, (`new` 연산자를 사용하는 Java 코드와 다소 동등하게도), 컨테이너가 스스로 bean의 생성자를 반사적으로 호출하여 bean을 생성할 때, bean 클래스가 생성되기를 지정.
- 컨테이너가 bean을 생성하기 위해 클래스의 `static` 팩토리 메서드를 호출하는 경우보다 적은 경우인, 객체를 생성하기 위해 호출된 `static` 팩토리 메서드를 포함하는 실제 클래스를 지정. `static` 팩토리 메서드의 호출으로부터 반환된 객체 타입은 같은 클래스 일 수도 있고, 완전히 다른 클래스 일 수도 있습니다.

<div class="blue-text-box">
<p style="text-align: center; font-size: 20px">중첩된 클래스명</p>

<p>중첩된 클래스에 대한 bean definition을 구성하고 싶다면, 중첩된 클래스의 이진명(binary name), 혹은 소스명(source name) 중 하나를 사용하시면 됩니다.</p>

<p>예를 들어, <code>com.example</code>안에 <code>SomeThing</code>이라는 클래스가 있고, 이 <code>SomeThing</code> 클래스는 <code>OtherThing</code>이라는 <code>static</code> 중첩 클래스를 가지고 있다면, 그들은 달러 표시(<code>$</code>) 혹은 점(<code>.</code>)으로 분리될 수 있습니다. 따라서 bean definition 안의 <code>class</code> 속성의 값은 <code>com.example.SomeThing$OtherThing</code> 혹은 <code>com.example.SomeThing.OtherThing</code>이 될 것입니다.</p>

</div>

#### 생성자로 인스턴스화하기

생성자를 사용하여 bean을 생성할 경우에, 모든 일반 클래스들은 Spring이 사용 가능하고, 호환이 가능하게 됩니다. 그 말인 즉슨, 개발되는 클래스는 아무런 인터페이스를 상속받지 않아도 되고, 특정한 방식으로 코드가 짜여야 한다는 법도 없습니다. 단순히 bean 클래스를 지정하는것만으로 충분합니다. 그러나, 해당 bean에 어떠한 타입의 IoC를 사용하냐에 따라, default(비어있는) 생성자가 필요할 수도 있습니다.

Spring IoC 컨테이너는 사실상 원하는 어떠한 클래스든 관리할 수 있습니다. True JavaBean들만 한정되게 관리할 수 있는 것이 아닙니다. 대부분의 Spring 사용자들은 컨테이너 안에서 properties를 따라 구상된 default(인자 없는) 생성자와 적절한 setters와 getters만을 가지는 실제 JavaBeans를 선호합니다. 컨테이너는 더 외부적인 non-bean-style의 클래스들을 가질 수 있습니다. 만약 예를 들어, JavaBean 명세를 전혀 지키지 않는 레거시 connection pool을 사용해야 한다 하더라도, Spring은 그것 또한 관리할 수 있습니다.

XML 기반 구성 메타데이터에서 다음과 같이 bean 클래스를 명세할 수 있습니다.

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

생성자에 인자를 전달하는 메커니즘(필요하다면), 그리고 객체가 생성된 후에 객체 인스턴스 속성을 관리하는 것에 대한 자세한 내용은 [의존성 주입](#beans-factory-collaborators)을 확인하세요.

#### <a name="beans-factory-class-static-factory-method"></a>정적 팩토리 메서드로 인스턴스화하기

정적 팩토리 메서드를 통해 생성하는 bean을 정의할때는 아래 두 속성을 사용하세요.

1. `class`: `static` 팩토리 메서드를 가지고 있는 클래스를 지정
2. `factory-method`: 팩토리 메서드 자체의 이름을 지정

해당 메서드를 호출할 수 있어야 하고(추후에 설명될 것과 같이, 필요하다면 인자들과 함께), 생성자를 통해 생성된것 처럼 취급되는, 활성 상태의 객체를 반환해야 합니다. Bean definition의 한가지 활용 용도는 레거시 코드의 `static` 팩토리들을 호출하는 것입니다.

아래의 bean definition은 팩토리 메서드를 호출함에 따라 생성되는 bean을 지정합니다. 해당 definition은 반환될 객체의 타입(클래스)를 지정하지 않고, 오로지 팩토리 메서드를 포함하는 클래스만을 지정합니다. 해당 예제에서, `createInstance()` 메서드는 정적 메서드이어야 합니다. 아래 예제는 팩토리 메서들르 지정하는 방법을 보여줍니다.

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

아래 예제는 위 bean definition과 함께 작동할 수 있는 클래스를 보여줍니다.

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

(선택적으로) 팩토리 메서드에 인자를 넘기는 것과, 팩토리로부터 반환된 객체의 인스턴스 속성을 설정하는 것에 대한 메커니즘의 자세한 내용은 [의존성과 구성의 세부 사항](#beans-factory-properties-detailed)

#### <a name="beans-factory-class-instance-factory-method"></a>인스턴스 팩토리 메서드로 인스턴스화하기

[정적 팩토리 메서드를 통해 인스턴스화](#beans-factory-class-static-factory-method)하는 것과 비슷하게, 인스턴스 팩토리 메서드로 인스턴스화하는 것은 새로운 bean을 생성하기 위해 컨테이너로부터 이미 존재하는 bean의 비정적(non-static) 메서드를 호출합니다. 이 메커니즘을 사용하려면, `class` 속성은 비워둔 채로 두고, `factory-bean` 속성에, 객체를 생성하기 위해 호출될 인스턴스 메서드를 포함하는 현재의(혹은 부모, 혹은 선조) 컨테이너 속의 bean 이름을 지정하세요. 팩토리 메서드 자체는 `factory-method` 속성으로 설정하세요. 아래의 예제는 그러한 bean을 어떻게 구성하나를 보여줍니다.

```xml
<!-- createInstance() 이라는 메서드를 포함하는 팩토리 bean -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- 해당 locator bean에 필요한 어떠한 아무 의존성을 주입 -->
</bean>

<!-- 팩토리 bean에 의해 생성될 bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

아래의 예제는 위 예제에 상응하는 클래스를 보여줍니다.

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

아래 예제에서 볼 수 있듯이, 하나의 팩토리 클래스는 하나 이상의 팩토리 메서드를 가지고 있을 수 있습니다.

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

아래의 예제는 위 예제에 상응하는 클래스를 보여줍니다.

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

이 접근법은 팩토리 bean 자체가 의존성 주입(DI)를 통해 관리되고 구성될 수 있다는 것을 보입니다. [의존성과 구성 세부 사항](#beans-factory-properties-detailed)을 확인하세요

<div class="info-box" style="height:125px">
    <div class="info-icon">
        <div class="icon"></div>
    </div>
    <div class="info-content">
        Spring 문서에서 "factory bean"이란, Spring 컨테이너에서 구성되고, 인스턴스, 혹은 정적 팩토리 메서드를 통해 객체를 생성하는 bean을 뜻합니다. 이에 반해, `FactoryBean`(대문자에 유의하세요)는 Spring-specific `FactoryBean`구현체 클래스를 나타냅니다.
    </div>
</div>

#### Bean의 런타임 타입 검사하기

특정 bean의 런타임 타입을 결정하는 것은 절대 사소한 일이 아닙니다. Bean 메타데이터 정의서의 특정 클래스는 아래 세가지 가능성이 있는 초기 클래스 레퍼런스 입니다.

1. 선언된 팩토리 메서드를 통해 잠재적으로 결합되는 경우
2. 또 다른 타입의 bean 런타임 타입을 가지는 `FactoryBean`이 되는 경우
3. 인스턴스 단계의 팩토리 메서드의 경우, 전혀 설정되지 않을 경우 (대신, 지정된 `factory-bean` 이름을 통해 해결될 것입니다.)

추가적으로, AOP proxying은, 타겟 bean의 실제 타입의 제한된 노출을 가지는 인터페이스 기반 프록시로 bean 인스턴스를 감쌀 것입니다(오직 상속받은 인터페이스들만).

추천되는 특정 bean의 실제 런타임 타입을 알아내는 방법은 특정 bean 이름으로 `BeanFactory.getType`을 호출하는 것입니다. 이는 위 경우들을 모두 고려하고, `BeanFactory.getBean` 호출이 반환할 같은 이름을 가지는 bean 객체의 타입을 반환합니다.

## <a name="beans-dependencies"></a>1.4. Dependencies (의존성)

전형적인 실무 애플리케이션은 하나의 객체(혹은 Spring 형식의 bean)만 가지고 있지 않습니다. 세상에서 가장 간단한 애플리케이션일지라도 엔드 유저가 일관된 애플리케이셔으로써 보게 될 것을 나타내기 위해 몇개의 서로 협력하는 객체들을 가지고 있습니다. 이후 절에서는, 자립하는 bean definition들을 정의하는 것 부터, 목표를 위해 객체들이 협력하는 완전히 구현된 애플리케이션까지의 단계를 설명할 것입니다.

### <a name="beans-factory-collaborators"></a>1.4.1. Dependency Injection (의존성 주입)

의존성 주입(DI)는 객체들이 오직 아래 세가지 방법으로만 그들의 의존성(함께 일할 다른 객체)을 정의하는 과정입니다.

1. 생성자 인자들
2. 팩토리 메서드로 전달되는 인자
3. 팩토리 메서드로부터 생성되거나 반환된 객체 인스턴스에 설정된 속성들

컨테이너는 bean을 생성할 때 이 의존성들을 주입합니다. 이 과정은 근본적으로, 클래스의 직접적인 생성 혹은 Service Locator 패턴을 사용하여 bean이 자체적으로 그의 인스턴스화 혹은 의존성들의 위치(location of its dependencies)를 제어하는 것에 대한 역전(이런 이유로 Inversion of Control이라는 이름이 생겼습니다)입니다. (역자: 즉, 원래는 bean이 스스로 자신의 의존성 등을 제어하는 것이 자연스럽지만 Spring container가 그 제어권을 가져와 bean 들을 제어하는 것이 제어의 역전이 일어났다고 보는 것입니다.)

DI 원칙으로 코드는 깨끗해지고, 객체들이 그들의 의존성들과 함께 제공된다면 탈동조화(decoupling)을 효과적으로 진행할 수 있습니다. 객체들은 그들의 의존성들을 탐색하지 않고, 그 의존성들의 위치 혹은 클래스를 알지 못합니다. 그 결과로, 특히 의존성들이 단위 테스트에 사용될 stub, mock 구현체들을 허용하는 인터페이스 혹은 abstract base class들에 있을 경우, 클래스들은 테스트하기 용이해집니다.

DI는 크게 두가지 형태로 존재합니다: [생성자 기반 의존성 주입](#beans-constructor-injection) 그리고 [Setter 기반 의존성 주입](#beans-setter-injection).

#### <a name="beans-constructor-injection"></a> 생성자 기반 의존성 주입

생성자 기반 DI는 컨테이너가, 각각이 의존성을 나타내는 몇개의 인자들을 가지는 생성자를 호출함으로써, 이루어집니다. 이와, 특정 인자들을 가지고 `Static` 팩토리 메서드를 호출하여 bean을 생성하는 것은 거의 동등하고, 해당 의견에서는 생성자로의 인자와 `static` 팩토리 메서드로의 인자를 비슷하게 취급합니다. 아래 예제는 오직 생성자 주입으로만 의존성이 주입될 수 있는 클래스를 보입니다.

```java
public class SimpleMovieLister {

    // SimpleMovieLister는 MovieFinder를 의존하고 있습니다.
    private final MovieFinder movieFinder;

    // Spring 컨테이너가 MovieFinder를 주입할 수 있도록 하는 생성자
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // 주입된 MovieFinder를 실제 사용할 비즈니스 로직 (생략)
}
```

이 클래스에는 전혀 특별할 것이 없다는 것에 주의하세요. 이것은 컨테이너 특정 인터페이스, 베이스 클래스, 혹은 어노테이션에 아무런 의존성도 없는 POJO(Plain Old Java Object)입니다.

##### <a name="beans-factory-ctor-arguments-resolution"></a>생성자 인자 처리(Resolution)

생성자 인자 처리 매칭은 인자의 타입으로 일어납니다. Bean definition의 생성자 인자에 잠재적 모호성이 없다면, bean definition에 정의된 생성자 인자들의 순서가 곧 해당 bean이 인스턴스화될 때의 적절한 생성자로전달될 순서입니다. 아래의 클래스를 참고하세요:

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

`ThingTwo`와 `ThingThree`는 상속 관계에 있지 않다고 가정한다면, 잠재적 모호성은 존재하지 않습니다. 이에, 아래의 구성은 정상적으로 작동할 것이고, `<constructor-arg/>` 요소에 생성자 인자 인덱스, 혹은 타입을 명시적으로 특정할 필요가 없습니다.

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

하나의 bean이 더 참조되고, 그의 타입이 정해져있다면, 매칭은 이루어질 수 있습니다(이전의 예제에서 이루어진 것과 같이). `<value>true</value>`와 같은 간단한 타입이 사용되었다면, Spring은 값의 타입을 결정할 수 없고, 도움 없이는 타입 매칭이 이루어질 수 없습니다. 아래의 클래스를 참고하세요:

```java
package examples;

public class ExampleBean {

    // 궁긍적 진리를 구하는데 걸리는 해의 수
    private final int years;

    // 인생, 우주, 그리고 모든것에 대한 진리
    private final String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

**생성자 인자 타입 매칭하기**

이전의 예제에서, 아래 예제에서 보이듯이, `type` 요소를 사용하여 생성자 인자를 명시적으로 특정한다면, 컨테이너는 간단한 타입들로 타입 매칭을 할 수 있습니다.

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

**생성자 인자 인덱스**

아래의 예제가 보이듯이, `index` 요소를 사용하여 생성자 인자의 인덱으를 명시적으로 특정할 수 있습니다.

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

다수의 간단한 값의 모호성에 대한 해결의 추가적인 내용으로, 인텍스를 특정함으로써 생성자가 같은 타입의 두개의 인자를 가지는 것에 대한 모호성을 해결할 수 있습니다.

<div class="info-box" style="height:75px">
    <div class="info-icon">
        <div class="icon"></div>
    </div>
    <div class="info-content">
        인덱스는 0을 기준으로 합니다.
    </div>
</div>

<br>

**생성자 인자 이름**

아래의 예제가 보이듯이 값 명확화를 위해 생성자 파라미터 이름을 사용할 수 있습니다.

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

특별한 설정 없이 이가 작동하기 위해서는 debug flag가 enabled된 상태로 코드가 컴파일되어, Spring이 생성자로부터 파라미터 이름을 검색할 수 있게 해야 한다는 것을 명심하세요. 만약 debug flag를 설정하지 못하거나, 하지 않고 싶다면, 생성자 인자들을 명시적으로 이름 짓기 위해[@ConstructorProperties](https://docs.oracle.com/javase/8/docs/api/java/beans/ConstructorProperties.html) JDK 어노테이션을 사용할 수 있습니다. 그렇게 된다면 예제 클래스는 아래와 같아질 것입니다.

```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

#### <a name="beans-setter-injection"></a>Setter 기반 의존성 주입

Setter 기반 DI는 bean을 인스턴스화하기 위한 기본 생성자(no-argument constructor) 혹은 인자 없는 팩토리 메서드의 호출 이후 bean에 있는 setter 메서드들을 컨테이너가 호출함으로써 이루어집니다.

이래의 예제는 순수 setter 주입을 사용해야만 의존성이 주입될 수 있는 클래스를 보입니다. 해당 클래스는 전통적인 Java입니다. 이는 컨테이너 특정 인터페이스, 베이스 클래스, 혹은 어노테이션에 대한 의존성이 없는 POJO입니다.

```java
public class SimpleMovieLister {

    // SimpleMovieLister는 MovieFinder를 의존하고 있습니다.
    private MovieFinder movieFinder;

    // Spring 컨테이너가 MovieFinder를 주입할 수 있도록 하는 setter 메서드
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // 주입된 MovieFinder를 실제로 사용하는 비즈니스 로직 (생략)
}
```

`ApplicationContext`는 자신이 관리하는 bean들을 위해 생성자 기반, 그리고 setter 기반 DI를 지원합니다. 생성자 접근을 통해 이미 몇개의 의존성들이 주입된 후에도 setter 기반 DI를 지원합니다. 속성의 형태를 변환하기 위한 `PropertyEditor` 인스턴스들과의 결합에서 사용하는 `BeanDefinition`의 형태로 의존성들을 구성합니다. 하지만, 대부분의 Spring 사용자들은 이 클래스들을 직접적으로 작업하지 않고(프로그래밍 방식으로), 대신 XML `bean` definitions, 어노테이션 처리된 components(`@Component`, `@Controller` 등으로 어노테이션 처리된 클래스들), 혹은 자바 기반의 `@Configuration` 클래스들 안의 `@Bean` 메서드들을 통해 작업합니다. 해당 소스들은 그 다음에 내부적으로 `BeanDefinition` 속의 인스턴스들로 변환되고, 전체 Spring IoC 컨테이너 인스턴스를 불러오기 위해 사용됩니다.

<div class="blue-text-box">
<p style="text-align: center; font-size: 20px">생성자 기반 vs setter 기반</p>

<p>생성자 기반과 setter 기반 DI를 혼영하여 사용할 수 있기 때문에, 필수 의존성에는 생성자를, 선택적 의존성에는 setter 메서드 혹은 configuration 메서드를 사용하면 좋습니다. Setter 메서드에 <a href="#beans-required-annotation">@Required</a> 어노테이션을 사용하여 해당 특성을 필수적인 의존성으로 만들 수 있습니다; 그러나, 인자의 프로그램적 검증이 있는 생성자 주입이 더 좋은 선택으로 보입니다.</p>

<p>String 팀은 대개 변경 불가(immutable) 객체로써 애플리케이션 요소들을 구현할 수 있게 하고, 필수 의존성들이 <code>null</code>이 아님을 보장하게 해주는 생성자 주입을 추천합니다. 뿐만아니라, 생성자 주입된 요소들은 항상 완전히 초기화된 상태로 클라이언트(호출한) 코드에게 반환됩니다. 추가적으로, 많은 양의 생성자 인자는 클래스가 너무 많은 책임을 가지고 있음을 내포하는 안좋은 코드(bad code smell)이고, 나은 주소 별 분리를 고려하여 리팩토링 되어야 합니다.</p>

<p>Setter 주입은 주로 클래스 내부에서 합리적인 디폴트 값을 할당받을 수 있는 선택적인 의존성에만 사용되어야 합니다. 그렇지 않다면, 해당 의존성을 사용하는 모든 곳에서 not-null 검증이 필요할 것입니다. Setter 주입의 한가지 장점은, setter 메서드는 해당 클래스의 객체들을 재구성 하거나, 재주입 할 수 있도록 합니다. 그러므로 <a href="https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jmx">JMX MBeans</a>을 통한 관리는 setter 주입의 나쁘지 않은 사용 사례입니다.</p>

<p>각각의 클래스들에 가장 알맞은 DI 방식을 선택하세요. 가끔, 소스를 가지고 있지 않은 서드 파티 클래스들을 사용할 때, 선택은 개인의 몫이 됩니다. 예를 들어, 만약 서드 파티 클래스가 어떠한 setter 메서드도 보여주지 않고 있다면, 생성자 주입만이 유일하게 가능한 DI 형식이 될 것입니다.</p>
</div>

#### <a name="beans-dependency-resolution"></a> 의존성 관리 과정 (Dependency Resolution Process)

컨테이너는 다음과 같이 bean 의존성 관리를 이루어냅니다.

* `ApplicationContext`는 모든 bean들을 설명하는 구성 메타데이터를 가지고 생성되고 초기화 됩니다. 구성 메타데이터는 XML, Java 코드, 혹은 어노테이션들로 작성될 수 있습니다.
* 각 bean들의 의존성들은 properties, 생성자 인자들, 혹은 정적 팩토리 메서드(일반적인 생성자 대신에 사용한다면)의 형태로 표현될 것입니다. 이 의존성들은 bean이 실제로 생성될 떄 bean에게 제공됩니다.
* 각 property 혹은 생성자 인자는 사용할 값의 실제 설명, 혹은 컨테이너 안의 또다른 bean으로의 참조입니다.
* 값을 나타내는 각 property 혹은 생성자 인자는 기존의 특정된 형태에서, 해당 property 혹은 생성자 인자의 실제 타입으로 변환됩니다. 기본적으로 Spring은 string 형태로 제공된 값을 `int`, `long`, `String`, `boolean` 등, 모든 내장 타입으로 변환할 수 있습니다.

Spring 컨테이너는 컨테이너가 생성될 때 각 bean의 구성을 검증합니다. 그러나 bean properties 자체는 bean이 실제 생성되기 전까지는 설정되지 않습니다. 싱글톤-스코프(singleton-scoped)의, 그리고 미리 인스턴스화(pre-instantiated: 기본값) 되기로 설정된 bean들은 컨테이너가 생성될 때 생성됩니다. (스코프들은 [Bean 스코프](#beans-factory-scopes)에 정의되어있습니다.) 그렇지 않다면, bean은 오직 그가 요청되었을 때에만 생성됩니다. Bean의 생성은, bean의 의존성, 그리고 그 의존성의 의존성(등등)이 생성되고 할당됨에 따라, 잠재적으로 beans의 그래프(graph of beans)를 발생시킬 수 있습니다. 이와 같은 의존성들 사이의 resolution mismatch는 늦게 나타날 수 있다는 것을 참고하세요-영향 받은 첫번째 bean의 생성에 나타날 것입니다.

<div class="blue-text-box">
<p style="text-align: center; font-size: 20px">순환 의존성 (Circular dependencies)</p>

<p>주로 생성자 주입을 사용한다면, 해결할 수 없는 순환 의존성 시나리오가 생길 수 있습니다.</p>

<p>예를 들어: 클래스 A는 생성자 주입을 통해 클래스 B의 인스턴스를 필요로 하고, 클래스 B 또한 생성자 주입을 통해 클래스 A의 인스턴스를 필요로 합니다. 이때 A와 B를 서로에게 주입하도록 bean을 구성한다면, Spring IoC 컨테이너는 컨타임에 해당 순환 참조를 탐지하고 <code>BeanCurrentlyInCreationException</code>를 던집니다.</p>

<p>한가지 가능한 해결방법은 몇개의 클래스들의 소스코드를 생성자 대신 setter에 의해 구성되도록 소스 코드를 수정하는 것입니다. 혹은, 생성자 주입을 사용하지 말고 setter 주입만 사용하세요. 다른 말로, 추천하는 방법은 아니지만, setter 주입으로 순환 참조를 해결할 수 있습니다.</p>

<p>일반적인 경우 (순환 의존이 없는 경우)와 다르게, bean A와 bean B 간의 순환 의존은 하나의 bean이 상대 bean에 완전히 초기화 된 채로 주입되기를 강제합니다 (고전적인 닭과 달걀의 문제와 같이요).</p>
</div>

<br>

일반적으로 올바른 행위를 하는것에 대해 Spring을 신뢰해도 좋습니다. Spring은 존재하지 않는 bean으로의 참조와 순환 의존과 같은 configuration 문제를, 컨테이너를 불러옴과 함께 탐지합니다. Spring은 bean이 실제 생성될 때 최대한 늦게 property을 설정하고, 의존성을 주입합니다. 이는, 정상적으로 불러진 Spring 컨테이너가, 생성하는데 혹은 의존성을 부여하는데 문제가 있는 객체를 요청할 때 예외를 발생시킬 수 있다는 것입니다 - 예를 들어, 속성이 빠져있거나 유효하지 않은 경우 bean은 예외를 던집니다. 이 잠재적으로 지연된 가시적 구성 문제는, 기본적 `ApplicationContext` 구현체들이 왜 싱글톤 bean들을 미리 인스턴스화하는지에 대한 이유입니다. 실제로 필요하기 이전에 생성된 이 bean들의 앞선 비용과 메모리에 대한 configuration 문제를 `ApplicationContext`가 생성된 이후가 아닌 생성될 때 발견할 것입니다. 하지만 해당 디폴트 행위를 override함으로써 싱글톤 bean들이 미리 즉시 인스턴스화되지 않고, 지연 생성되도록 할 수 있습니다.

순환 의존이 존재하지 않는다면, 협력하는 각 bean들은 상대 bean에게 주입되기 이전에 완전히 구성됩니다. 이는, bean A가 bean B를 의존할 때, Spring IoC 컨테이너는 bean A의 setter 메서드를 호출하기 이전에 bean B를 완전히 구성을 마친다는 의미입니다. 완전히 구성을 마친다는 것은, bean이 인스턴스화되고 (미리 인스턴스화된 싱글톤이 아니라면), 의존성들이 설정되고, 관계된 생명주기(lifecycle) 메서드들([구성된 초기화 메서드](#beans-factory-lifecycle-initializingbean) 혹은 [InitializingBean 콜백 메서드](#beans-factory-lifecycle-initializingbean))이 호출된다는 것입니다.

#### 의존성 주입 예시

아래의 예제는 setter 기반 DI를 위해 XML 기반 구성 메타데이터를 사용합니다. Spring XML configuration 파일의 작은 부분은 다음과 같이 몇개의 bean definition을 명시합니다.

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- 중첩된 ref 요소를 사용한 setter 주입 -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- 조금 더 멋진 ref 요소를 사용한 setter 주입 -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

아래의 예제는 위 예제에 대응하는 `ExampleBean` 클래스입니다.

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

이전 예제에서, setter들은 XML 파일 안에서 properties와 대응하도록 선언되어있습니다. 아래의 예제는 생성자 기반 DI를 보여줍니다.

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- 중첩된 ref 요소를 사용한 생성자 주입-->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- 조금 더 멋진 ref 속성을 사용한 생성자 주입 -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

아래의 예제는 이에 대응하는 `ExampleBean`을 나타냅니다.

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

Bean definition 안에 지정된 생성자 인자들은 `ExampleBean`의 생성자 요소로써 사용됩니다.

이제 생성자를 사용하는 것 대신, Spring이 객체의 인스턴스를 반환하는 `static` 팩토리 메서드를 호출하는 변형된 예시를 생각해보세요.

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

아래의 예제는 대응하는 `ExampleBean` 클래스를 나타냅니다.

```java
public class ExampleBean {

    // private 생성자
    private ExampleBean(...) {
        ...
    }

    // 정적 팩토리 메서드; 해당 메서드의 인자들은 실제
    // 사용되든, 안되든간에, 반환되는 bean의 의존성으로 취급할 수 있습니다.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // 기타 기능들...
        return eb;
    }
}
```

`static` 팩토리 메서드로의 인자들은 마치 생성자가 실제 사용되는 것 처럼 `<constructor-arg>` 요소들로 제공됩니다. 팩토리 메서드가 반환하는 클래스의 타입은 해당 `static` 팩토리 메서드를 포함하고 있는 클래스의 타입과 동일하지 않아도 됩니다 (위 예제는 동일하지만요). 인스턴스 팩토리 메서드(non-static)는 본질적으로 동일한 방식으로 사용될 수 있기 때문에 (`class` 속성 대신 `factory-bean` 속성을 사용하는 것과), 여기에서는 세부 내용을 다루지 않을 것입니다.

### <a name="beans-factory-properties-detailed"></a>1.4.2. Dependencies와 Configuration 세부 사항

#### 확실한 값 (Primitives, Strings, 등...)

#### `idref` 원소

#### 다른 bean으로의 참조 (Collaborators)

#### <a name="beans-inner-beans"></a>내부 bean(inner bean)

### 1.4.3. `depends-on` 사용하기

### <a name="beans-factory-lazy-init"></a>1.4.4 Lazy-initialized Beans (지연 초기화)

### <a name="beans-factory-autowire"></a>1.4.5. Autowiring Collaborators

### 1.4.6. Method Injection

## <a name="beans-factory-scopes"></a>1.5. Bean Scopes (Bean 영역)

### 1.5.1. The Singleton Scope (싱글톤 영역)

### 1.5.2. The Prototype Scope (프로토타입 영역)

### 1.5.3. 프로토타입 빈 의존성을 가진 싱글톤 빈

### 1.5.4. Request, Session, Application, 그리고 WebSocket Scopes

### 1.5.5. Custom Scopes




<!-------------------------------------------------------------->


### 1.6.1. Lifecycle Callbacks

#### <a name="beans-factory-lifecycle-initializingbean"></a>Initialization Callbacks

#### <a name="beans-factory-lifecycle-disposablebean"></a>Destruction Callbacks

## <a name="beans-child-bean-definitions"></a>1.7 Bean definition 상속

## <a name="beans-annotation-config"></a>1.9. 어노테이션 기반 컨테이너 설정

### <a name="beans-required-annotation"></a>1.9.1. @Required

## <a name="beans-java"></a>1.12. 자바 기반의 컨테이너 설정

### <a name="beans-java-bean-annotation"></a>1.12.3. `@Bean` 어노테이션 사용하기

### <a name="context-create"></a>1.15.5. 웹 애플리케이션을 위한 간편한 ApplicationContext 인스턴스화

## <a name="beans-beanfactory"></a>1.16. `Bean Factory`



### <a name="aop-atconfigurable"></a>5.10.1. AspectJ를 이용한 Spring에서 도메인 객체 의존성 주입하기


# <a name="resources"></a>2. Resources

## <a name="resources-app-ctx"></a>2.8 Application Contexts와 Resource Paths