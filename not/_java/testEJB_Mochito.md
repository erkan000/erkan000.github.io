---
layout: default
title: Unit testing EJB's with embedded @Resources
date: 03.02.2019
---



At my work, i often need to test newly created EJB methods. My EJB classes also have @Resources datasource and it gets datasource from Application Server Context. For doing unit test of this situation, i need to have Mock datasource and inject EJB's. First of all mochito pom is;

```sh
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>2.23.4</version>
        <scope>test</scope>
    </dependency>
```

Test class definition and datasource mock are;

```java
@RunWith(MockitoJUnitRunner.class)
public class Mocker {

    @Mock
    private DataSource medulaDataSource;

    @InjectMocks
    private MyEJB ejb;

    @Before
    public void setUp() throws Exception {
        assertNotNull(xDataSource);
        when(xDataSource.getConnection()).thenReturn(connect());
    }
```

At above code, we mock the datasource and ejb, InjectMocks is magic;) After that when our ejb class call "xDataSource.getConnection()" we handle it and replace it with our real connection to the database. Connect methot calls "DriverManager.getConnection" method and returns a datasource.

Happy unit testing!
