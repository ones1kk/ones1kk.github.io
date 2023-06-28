---
title: 템플릿 메소드 패턴(Template Method Pattern)
date: 2023-06-27 00:01:00 +09:00
categories: [ Software Architecture, Design Pattern ]
tags: [ GoF, Design-Pattern, Template-Method-Pattern ]
---

# Template Method Pattern

템플릿 메소드 패턴(Template Method Pattern)은 은 Gang of Four(GoF) 디자인 패턴 중
하나로 [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.yes24.com/Product/Goods/17525598)라는 책에서
소개 된 23가지 디자인 패턴 중 하나입니다.
템플릿 메소드 패턴은 **행동 패턴**으로서 특정 알고리즘을 사용하는 유사한 클래스들의 알고리즘의 구조를 정의하고, 하위 클래스에서는 구조를 변경하지 않고 특정 단계 메소드들을 구체화하여 해당 구조를 표현하기 위해서
사용됩니다.
이를 통해 클라이언트는 특정 알고리즘 구조(템플릿)를 변경하지 않고 구체 메소드를 확장하거나 재정의할 수 있는 유연성을 제공합니다. 또한 정의된 구조를 제외하고 하위의 세부 실행 내용을 다양하게 구현하여 사용할 수
있습니다.

# Structure

![template_pattern_uml_diagram](/assets/img/software-architecture/design-pattern/template-method-pattern/template_pattern_uml_diagram.png)
> [출처](https://www.tutorialspoint.com/design_pattern/template_pattern.htm)

- Abstract Class
  - 알고리즘의 구조를 정의하고 템플릿 메소드를 포함합니다.
  - 추상 메소드와 구체 메소드를 포함하며 구체 메소드들은 템플릿 메소드를 호출하여 알고리즘의 구조를 유지합니다.
  - 순서 제어용으로 사용되는 메소드로 기본적인 내용만 구현하거나 비워놓는 메서드인 ``hook method``를 가집니다.
- Concrete Class
  - 추상 클래스를 상속받아 템플릿 메소드를 구현하고, 필요한 구체 메소드들을 오버라이딩하여 알고리즘의 특정 단계를 구체화합니다.

# Template Method Pattern of Spring Framework

## JdbcTemplate

![jdbc-template](/assets/img/software-architecture/design-pattern/template-method-pattern/jdbc-template.png)

``org.springframework.jdbc.core.JdbcTemplate``은 자바에서 데이터베이스에 접속할 수 있도록 지원해주는 자바 API인 JDBC(Java Database Connectivity)
사용을 템플릿화한 클래스로 SQL 쿼리(Query) 실행이나 업데이트를 실행하고, ``ResultSet``을 순환, 또 발생한 JDBC 예외를 잡아서 ``org.springframework.dao`` 패키지에서
정의된
일반적인 예외로 변환합니다.
또 ``JdbcTemplate``은 템플릿 메소드 패턴과 템플릿 콜백 패턴을 적용해 구현된 클래스로 추상
클래스인 ``org.springframework.jdbc.support.JdbcAccessor``, ``org.springframework.jdbc.core.JdbcOperations``를 상속 받아
구현하였습니다.  
``JdbcAccessor``는 JDBC를 사용하여 데이터베이스에 접근하는 DAO(Database Access Object) Helper 클래스들을 위한 기본 클래스로 일반적으로 JDBC 접근 작업에 대한
데이터베이스 연결, 트랜잭션 관리, 예외 처리 등과 같은 공통된 동작을 처리하는 메소드들이 정의되어 있습니다. 
``JdbcOperations``는 JDBC 작업을 지정하는 인터페이스로 SQL 쿼리 실행, 업데이트, 프로시저 호출 등이 콜백 패턴으로 ``JdbcTemplate``에 구현되어 있습니다.  

``JdbcTemplate``의 경우 ``excute()`` 메소드를 통해 템플릿 메소드를 제공합니다.  

```java 
@Override
@Nullable
public <T> T execute(StatementCallback<T> action) throws DataAccessException {
  Assert.notNull(action, "Callback object must not be null");

  Connection con = DataSourceUtils.getConnection(obtainDataSource());
  Statement stmt = null;
  try {
    stmt = con.createStatement();
    applyStatementSettings(stmt);
    T result = action.doInStatement(stmt);
    handleWarnings(stmt);
    return result;
  }
  catch (SQLException ex) {
    // Release Connection early, to avoid potential connection pool deadlock
    // in the case when the exception translator hasn't been initialized yet.
    String sql = getSql(action);
    JdbcUtils.closeStatement(stmt);
    stmt = null;
    DataSourceUtils.releaseConnection(con, getDataSource());
    con = null;
    throw translateException("StatementCallback", sql, ex);
  }
  finally {
    JdbcUtils.closeStatement(stmt);
    DataSourceUtils.releaseConnection(con, getDataSource());
  }
}
```

JDBC 작업을 실행하기 위한 전반적인 구조를 제공하면서, 서브클래스 또는 콜백 객체가 프로세스의 특정 부분에 대한 구체적인 구현을 제공할 수 있게 합니다.
1. 데이터베이스 접속에 필요한 ``DataSource``를 ``JdbcAccessor``로부터 획득
2. 서브클래스 또는 콜백 객체를 인자로 SQL 쿼리 실행
3. 예외 처리
4. 리소스 정리

위와 같이 템플릿 메소드 패턴을 활용함으로써 공통적인 JDBC 작업 흐름을 캡슐화하고 코드 재사용성을 높여  작업의 저수준 세부 사항을 걱정하지 않고 필요한 SQL 문장과 결과 추출 로직에 집중할 수 있도록 합니다.

## FrameworkServlet

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다. 

