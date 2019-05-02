---
layout: default
title: Local Context oluşturmak
date: 12.02.2019
---


# Local Context oluşturmak

Test edeceğimiz uygulama eğer Context'e bakıp bir nesneye erişiyorsa bunu test etmek için bir Context Container'a, yani uygulama sunucusu veya enbedded container'a vb.. ihtiyacımız vardır. Testlerimizi bunlar aracılığı ile yapmak ise bize zaman kaybettirir.(Sunucunun ayağa kalkması için geçecek zaman) Bu sebepten Context'imizi kendimiz oluşturup çok daha hızlı testler yapacağız. Öncelikle testimizin bizim kullanacağımız context'i kullanmasını belirtmek için;

```java
NamingManager.setInitialContextFactoryBuilder(new CustomContextFactory());
```

CustomContextFactory sınıfımız ise InitialContextFactory ve InitialContextFactoryBuilder interface'leri implemente eden özelleştirilmiş Context sınıfımızdır. Sınıfımız şu şekilde;

```java
public class CustomContextFactory implements InitialContextFactory, InitialContextFactoryBuilder {

    public static Context ctx;

    static{
        try {
            ctx = new CustomContext();
        } catch (NamingException e) {
            e.printStackTrace();
        }
    }
    public InitialContextFactory createInitialContextFactory(Hashtable<?, ?> environment) throws NamingException {
        return new CustomContextFactory();
    }

    public Context getInitialContext(Hashtable<?, ?> environment) throws NamingException {
        return ctx;
    }

}
```

Yukarıdaki factory sınıfımız static kod bloğunda CustomContext sınıfı instance'ı oluşturuyor. CustomContext sınıfımız ise InitialContext sınıfımızdan extend olmaktadır.

```java
public class CustomContext extends InitialContext{

    public CustomContext() throws NamingException {
        super();
    }

    @Override
    public Object lookup(String name) throws NamingException {
        if(name.equals("java:comp/env/jdbc/test")){

            return getDS();
        }else if(name.equals("java:comp/env/service/TestService")){

            TestProxy a = new TestProxy();

            return a;
        }
        return null;
    }
}
```

Bu sınıfta test ettiğim uygulamada çağrılan bazı lookup ları manuel olarak buraya ekledim ve InitialContext'i implemente etmiş oldum. Testlerimde context'de bulunan herhangi bir nesneye ise şu şekilde ulaşabilidim.

```java
    Context ctx = new InitialContext();

    TakipEJB takipEJB = (TakipEJB)ctx.lookup("TakipEJB");
```

14.02.2019
