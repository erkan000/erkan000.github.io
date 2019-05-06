---
layout: default
title: JAX-RS ile web servisler
date: 03.02.2019
---



JAX-RS         =         "RESTful Web Services" 

Jax-rs rest servisler yazmak için java tarafından oluşturulmuş spesifikasyondur.( [JSR-311](http://jcp.org/aboutJava/communityprocess/final/jsr311/index.html) )  Bu specleri implemente eden resteasy, jersey ve apache cf gibi popüler imlementasyonlar bulunmaktadır. Uygulama sunucunuzun dökümanlarında hangisini uyguladığı bulabilirsiniz.

Önce resti kullanacağımız belirtmek için projemize 2 yolla bunu belirtebiliriz.

#### Annotation kullanarak

Web projemizde bir sınıf oluşturarak aşağıdaki gibi belirtilebilir.

```java
    @ApplicationPath("rest")
    public class JAXRSConfiguration extends Application {

    }
```

### web.xml dosyası ile

Web pojemizin web.xml dosyasına;

```markup
    <servlet>
        <servlet-name>javax.ws.rs.core.Application</servlet-name>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>javax.ws.rs.core.Application</servlet-name>
        <url-pattern>/rest/*</url-pattern>
    </servlet-mapping>
```

Bu configurasyonlar ile context-path'in altında REST servislerin hangi path'den sunulacağını belirtmiş olduk. Ardından bir sınıf oluşturup servisimizi ve bütün özelliklerini belirtiyoruz.

```java
@Path("hizmetBildirim")
@Produces("application/json")
@Consumes("application/json")
public class ProductService {

    @POST
    public Response createProduct(Product prod) {

        if(prod.getPrice() > 100)
            return Response.status(Response.Status.OK).entity(new                 CustomException("OK")).build();
        else
            return Response.status(Response.Status.BAD_REQUEST).entity(new CustomException("NOK")).build();
    }

}
```

Burada görüldüğü gibi annotationlar ile herşey açık ve net! Dikkat ettiyseniz parametre olarak Product sınıfı alınmış, implementasyon bunu consume ve produce annotationlarına göre kendi otomatik olarak ilgili sınıfın türüne değiştirmektedir. 

Giriş sınıfında parametre almak için bazı metodlar vardır, bunlar kullanacağımız metoda göre değişir. Şu şekildedir;

- **PathParam:** Bunu kullanarak path üzerinden(url değişkeni) parametre alıyoruz. 
- **QueryParam:** Parametrelerinizi “&” ve “?” kullanarak url içerisinde belirtiyorsunuz. 
- **FormParam:**  HTML Form’larından gönderdiğimiz veriler

Giriş sınıfında validasyon yapmak için JAXB (Bean Validation) spesifikasyonundan yararlanabiliriz. Önce ilgili sınıfta alanların üzerine validasyonları belirtiyoruz. (@NotNull, @Size(min=2, max=8))) Ardından JAX-RS servis metodu önüne @Valid annotation'ınını koyuyoruz.

```java
public Response createProduct(@Valid Product prod) {
```

Bu şekilde validasyonu tamamlamış olduk. Tabii bu haliyle işimize yaramaz çünkü gelen mesajları özelleştirmek ve dilini değiştirmek istiyorum. Bu sebeple önce validasyon hatalarını yakalayabilmek için @Provider anotasyonu ile şu sınıfı oluştumamız gerekiyor;

```java
@Provider
public class ValidationExceptionMapper implements ExceptionMapper<ValidationException> {

    @Override
    public Response toResponse(ConstraintViolationException exception) {
        if (exception instanceof ValidationViolationException) {
            // violation.getPropertyPath() + " " + violation.getMessage();

            return Response.status(Response.Status.BAD_REQUEST).entity(new CustomException("NOK")).build();
        }
```

Hata mesajını yerelleştirmek için ise resource'lara `ValidationMessages.properties` dosyası eklenir. İçine istediğimiz gibi mesajları ekleyebiliriz. Değişkenler şu şekildedir;

```bash
javax.validation.constraints.AssertFalse.message     = must be false

javax.validation.constraints.AssertTrue.message      = must be true

javax.validation.constraints.DecimalMax.message      = must be less than ${inclusive == true ? 'or equal to ' : ''}{value}

javax.validation.constraints.DecimalMin.message      = must be greater than ${inclusive == true ? 'or equal to ' : ''}{value}

javax.validation.constraints.Digits.message          = numeric value out of bounds (<{integer} digits>.<{fraction} digits> expected)

javax.validation.constraints.Email.message           = must be a well-formed email address

javax.validation.constraints.Future.message          = must be a future date

javax.validation.constraints.FutureOrPresent.message = must be a date in the present or in the future

javax.validation.constraints.Max.message             = must be less than or equal to {value}

javax.validation.constraints.Min.message             = must be greater than or equal to {value}

javax.validation.constraints.Negative.message        = must be less than 0

javax.validation.constraints.NegativeOrZero.message  = must be less than or equal to 0

javax.validation.constraints.NotBlank.message        = must not be blank

javax.validation.constraints.NotEmpty.message        = must not be empty

javax.validation.constraints.NotNull.message         = must not be null

javax.validation.constraints.Null.message            = must be null

javax.validation.constraints.Past.message            = must be a past date

javax.validation.constraints.PastOrPresent.message   = must be a date in the past or in the present

javax.validation.constraints.Pattern.message         = must match "{regexp}"

javax.validation.constraints.Positive.message        = must be greater than 0

javax.validation.constraints.PositiveOrZero.message  = must be greater than or equal to 0

javax.validation.constraints.Size.message            = size must be between {min} and {max}
```

Dosya içinde görüleceği üzere bazı parametreleri de gönderebiliriz.

```bash
${value} ${validatedValue} gibi
```

Doğru bir REST servis yazmak için rest api design guide;

https://github.com/NationalBankBelgium/REST-API-Design-Guide/wiki





```java
    ConstraintViolationException ex = (ConstraintViolationException) exception;
    ConstraintViolation<?> hata = ex.getConstraintViolations().iterator().next();
    System.out.println(hata.getPropertyPath().toString());
    System.out.println(hata.getMessage().toString());
```
