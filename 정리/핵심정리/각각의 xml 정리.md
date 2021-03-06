각각의 xml 정리
=======================
# 1. web.xml
**web.xml**
```
웹과 관련된 설정 (리스너, 어플리케이션 파라미터, 서블릿 설정, 필터 설정 등..)을 담고 있고 
tomcat은 web.xml 이지만 웹 컨테이너 마다 조금씩 이름은 다르다. 
``` 
기본적으로 서블릿 컨테이너와 연관된 xml 이므로 
1. 서블릿 객체 등록 ```<servlet>``` 
2. SpringMVC 적용시 DispatcherServlet 등록 
3  서블릿 또는 객체에 대한 url 매핑 
4. Spring 컨테이너 등록 가능 ```<context-param>```(근데 우리는 안 했었다)  
5. ```<listener>```로 우선 생성할 컨테이너 등록 가능

***
# 2. pom.xml
**pom.xml**
```
빌드/배포와 관련된 모든 설정을 담고 있고 
maven 이라는 유틸리티에서 메타 정보로 사용하는 설정파일 입니다.
``` 
스프링부트에서 gradle 사용하듯이 maven 으로 라이브러리 버전관리     
    
***
# 3. applicationContext.xml
우선 ```applicationContext.xml```은 ```web.xml``` 에서 ```<context-param>```을 이용해 생성하는 xml 이다.     
   
기존 web.xml 은 웹 어플리케이션에서 일반적으로 사용되는 큰 틀의 xml을 의미한다면   
applicationContext.xml 그중 스프링에 관련된 부분만 실행 시킬 수 있도록 따로 빼 놓은 xml이라 생각하면 된다.    
   
그리고 필수 사항은 아니어서 이름도 아무렇게나 지어도 되고 여러개 만들어도 된다.  

1. 의존성 주입을 위한 ```<bean>``` 객체 등록
2. AOP를 위한 어드바이스 객체 지정 및 포인트컷 지정
3. JDBC 관련 엘리먼트 사용 
4. 프로퍼티 읽어오기  

***
# 4. 동작 방식
## (IoC)
web.xml로 지정된 Servlet은 서블릿 컨테이너로 인해 객체가 생성되고 그 안에 메소드가 실행된다.   
그렇다면 서블릿 클래스 안에 객체를 넣어서 사용하고 싶을 수 있는데  
서블릿이 100개 있다고 가정시 각각의 서블릿에 A를 사용하고자 하여 new A()를 100번 해야되는 번거로움과  
무엇보다 유지보수시에 클래스를 수정하거나 바꿀시에 이를 100배 더 해야된다.    
  
스프링 프레임워크에서는 이를 방지하고자 컨테이너에서 객체를 생성해서 주입해주는 DI/IoC 개념이 생겼으며  
의존성 주입해주는 방법으로는     
1. ```<bean>```등록후 -> 생성자 인젝션/Setter인젝션
2. ```@Component``` 등록후 -> @Autowired(자료형)/@Resouce(id 이름)/@Qualifier(2개 일때 이름으로)   
   
가 있다. 하지만 주로 xml은 큰 틀에서 동작하니 어노테이션 사용을 권장한다.    

## (AOP)  
DB 트랜잭션 관련 처리나 로그를 남기고 싶을 때 등등 
공통으로 사용하는 코드에 대해서 관리하기 쉽게 만들어주는 방법으로    
  
예를 들면 ServiceImpl 에서 LogAdvice 객체를 생성하고 각 메소드마다 메소드를 사용한다고 가정시  
LogAdvice 객체를 의존성 주입을 하여 객체간의 변동을 편하게 할 수가 있더라도  
각 기능 메소드마다 어드바이스 메소드를 실행하기에 기능 메소드가 100개면 또 100개를 입력하고 수정해야한다.    
이를 방지하기 위해 AOP를 지원해준다.  

1. pom.xml 에서 AOP 라이브러리 설치
2. applicationContext.xml 에서 AOP 네임 스페이스 추가  
3. applicationContext.xml ```<bean>```으로 어드바이스 객체 생성 
4. AOP를 사용할 범위를 지정하는 포인트 컷 설정
5. 어드바이스 객체 지정 및 포인트컷에 사용될 공동 기능 메소드를 지정   
  
그리고 메소드를 지정할때 언제 동작할 것인지 설정 가능   

**용어정리**
* 조인포인트 : 클라이언트가 호출하는 모든 비즈니스 클래스  
* 포인트컷 : 조인포인트에서 어드바이스 메소드를 사용할 특정 클래스  
* 어드바이스 : 공통 기능을 가지고 있는 클래스 (캡슐화)  
* 어드바이스 메소드 : 어드바이스내에 있는 공통기능 메소드  
* 위빙 : ```<aop:before..... method="로그1()">```과 같이 메소드가 삽입되는 과정
* 애스팩트 : 포인트컷과 어드바이스가 결합된 상태를 의미  

추가로 어드바이스 메소드를 사용하는 핵심 메소드도 알 수 있는데   
어드바이스 메소드에 ```JoinPoinr jp```로 매개변수를 만들어 주고  
```
Signature getSignature() : 클라이언트가 호출한 메소드의 시그니처 정보가 저장된 Signature 객체 리턴  
```
```
String getName() : 클라이언트가 호출한 메소드 이름 리턴
String toLongString() : 클라이언트가 호출한 메소드의 리턴타입, 이름, 매개변수를 패키지 경로까지 포함하여 리턴 
String toShortString() : 클라이언트가 호출한 메소드 시그니처를 축약한 문자열로 리턴  
```
```
String methodName =  jp.getSignature().getName();
```
를 호출하여 사용할 수 있다.  
이렇게 사용하는 이유는 명확하게 하여 어떠한 부분에서 실행되는지 로그의 추적을 돕도록 해준다.  
  
