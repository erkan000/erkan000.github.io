## Servlets

A servlet is a Java programming language class used to extend the capabilities of servers that host applications accessed by means of a request-response programming model. Although servlets can respond to any type of request, they are commonly used to extend the applications hosted by web servers. For such applications, Java Servlet technology defines HTTP-specific servlet classes.

The javax.servlet and javax.servlet.http packages provide interfaces and classes for writing servlets. All servlets must implement the Servlet interface, which defines lifecycle methods. When implementing a generic service, you can use or extend the GenericServlet class provided with the Java Servlet API. The HttpServlet class provides methods, such as doGet and doPost, for handling HTTP-specific services.

### Servlet Lifecycle

The lifecycle of a servlet is controlled by the container in which the servlet has been deployed. When a request is mapped to a servlet, the container performs the following steps.

1. If an instance of the servlet does not exist, the web container:
   a. Loads the servlet class
   b. Creates an instance of the servlet class
   c. Initializes the servlet instance by calling the `init` method
2. The container invokes the service method, passing request and response objects. 

If it needs to remove the servlet, the container finalizes the servlet by calling the servlet's `destroy` method

You can monitor and react to events in a servlet's lifecycle by defining listener objects whose methods get invoked when lifecycle events occur. These types of events can monitor with following classes;

| Object      | Event                                                        | Listener Interface and Event Class                           |
| :---------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Web context | Initialization and destruction                               | `javax.servlet.ServletContextListener` and `ServletContextEvent` |
| Web context | Attribute added, removed, or replaced                        | `javax.servlet.ServletContextAttributeListener` and `ServletContextAttributeEvent` |
| Session     | Creation, invalidation, activation, passivation, and timeout | `javax.servlet.http.HttpSessionListener`, `javax.servlet.http.HttpSessionActivationListener`, and `HttpSessionEvent` |
| Session     | Attribute added, removed, or replaced                        | `javax.servlet.http.HttpSessionAttributeListener` and `HttpSessionBindingEvent` |
| Request     | A servlet request has started being processed by web components | `javax.servlet.ServletRequestListener` and `ServletRequestEvent` |
| Request     | Attribute added, removed, or replaced                        | `javax.servlet.ServletRequestAttributeListener` and `ServletRequestAttributeEvent` |

** Use the `@WebListener` annotation to define a listener to get events for various operations on the particular web application context.

### Sharing Information

Collaborating web components share information by means of objects that are maintained as attributes of four scope objects. You access these attributes by using the `getAttribute` and `setAttribute` methods of the class representing the scope. Below lists the scope objects.

| Scope Object | Class                                     | Accessible From                                              |
| :----------- | :---------------------------------------- | :----------------------------------------------------------- |
| Web context  | `javax.servlet.ServletContext`            | Web components within a web context.                         |
| Session      | `javax.servlet.http.HttpSession`          | Web components handling a request that belongs to the session. |
| Request      | Subtype of `javax.servlet.ServletRequest` | Web components handling the request.                         |
| Page         | `javax.servlet.jsp.JspContext`            | The JSP page that creates the object.                        |



#### Controlling Concurrent Access to Shared Resources

In a servlet never create or reassing a member variable(incuding static variales). Member variables are not thread safe. A web container will typically create a thread to handle each request. To ensure that a servlet instance handles only one request at a time, a servlet can implement the `SingleThreadModel` interface. If a servlet implements this interface, no two threads will execute concurrently in the servlet's service method. A web container can implement this guarantee by synchronizing access to a single instance of the servlet or by maintaining a pool of web component instances and dispatching each new request to a free instance. This interface does not prevent synchronization problems that result from web components' accessing shared resources, such as static class variables or external objects.

### Create a Servlet

```java
@WebServlet("/test")
public class TestServlet extends HttpServlet {
    
//  all of same 
	@WebServlet("/test")
    @WebServlet(value = "/test")
    @WebServlet(urlPatterns = {"/test"})
```



### Pass variables to servlet

```java
@WebInitParam(name = "initParam", value = "2")
```



### Service Methods

The service provided by a servlet is implemented in the `service` method of a `GenericServlet`, in the `do`*Method* methods (where *Method* can take the value `Get`, `Delete`, `Options`, `Post`, `Put`, or `Trace`) of an `HttpServlet` object, or in any other protocol-specific methods defined by a class that implements the `Servlet` interface. The term **service method** is used for any method in a servlet class that provides a service to a client.

GenericServlet.service method called in every HTTP request. Tthis method calls doGet or doPost or .. with appriciate incoming HTTP request. If you override this doXxx methods doesnt works because you must implement these methods. 

### Getting Information from Requests

A request contains data passed between a client and the servlet. All requests implement the `ServletRequest` interface. This interface defines methods for accessing the following information: 

- Parameters, which are typically used to convey information between clients and servlets
- Object-valued attributes, which are typically used to pass information between the web container and a servlet or between collaborating servlets
- Information about the protocol used to communicate the request and about the client and server involved in the request
- Information relevant to localization 

You can also retrieve an input stream from the request and manually parse the data. To read character data, use the `BufferedReader` object returned by the request's `getReader` method. To read binary data, use the `ServletInputStream` returned by `getInputStream`. 

HTTP servlets are passed an HTTP request object, `HttpServletRequest`, which contains the request URL, HTTP headers, query string, and so on. 



```java
HttpSession session = request.getSession();
```

can hold information about a given user, between requests. So, if you set an object into the session object during one request, it will be available for you to read during any subsequent requests within the same session time scope. 



```java
ServletContext context = request.getSession().getServletContext();
```

contains meta information about the web application. For instance, you can access context parameters set in the web.xml file, you can forward the request to other servlet, you can store application wide parameters in the ServletContext



### Constructing Responses

A response contains data passed between a server and the client. All responses implement the `ServletResponse` interface. This interface defines methods that allow you to do the following. 

- Retrieve an output stream to use to send data to the client. To send character data, use the `PrintWriter` returned by the response's `getWriter` method. To send binary data in a Multipurpose Internet Mail Extensions (MIME) body response, use the `ServletOutputStream` returned by `getOutputStream`. To mix binary and text data, as in a multipart response, use a `ServletOutputStream` and manage the character sections manually.
- Indicate the content type (for example, `text/html`) being returned by the response with the `setContentType(String)` method. This method must be called before the response is committed. A registry of content type names is kept by the Internet Assigned Numbers Authority (IANA) at `http://www.iana.org/assignments/media-types/`.
- Indicate whether to buffer output with the `setBufferSize(int)` method. By default, any content written to the output stream is immediately sent to the client. Buffering allows content to be written before anything is sent back to the client, thus providing the servlet with more time to set appropriate status codes and headers or forward to another web resource. The method must be called before any content is written or before the response is committed.
- Set localization information, such as locale and character encoding. 

HTTP response objects, `javax.servlet.http.HttpServletResponse`, have fields representing HTTP headers, such as the following. 

- Status codes, which are used to indicate the reason a request is not satisfied or that a request has been redirected.
- Cookies, which are used to store application-specific information at the client. Sometimes, cookies are used to maintain an identifier for tracking a user's session 

## Filters

A **filter** is an object that can transform the header and content (or both) of a request or response. The filtering API is defined by the `Filter`, `FilterChain`, and `FilterConfig` interfaces in the `javax.servlet` package. You define a filter by implementing the `Filter` interface.

```java
@WebFilter(urlPatterns = {"/Login"})
public class SampleFilter implements Filter{
```



The most important method in the `Filter` interface is `doFilter`, which is passed request, response, and filter chain objects. This method can perform the following actions; 

- Examine the request headers.
- Customize the request object if the filter wishes to modify request headers or data.
- Customize the response object if the filter wishes to modify response headers or data.
- Invoke the next entity in the filter chain. If the current filter is the last filter in the chain that ends with the target web component or static resource, the next entity is the resource at the end of the chain; otherwise, it is the next filter that was configured in the WAR. The filter invokes the next entity by calling the `doFilter` method on the chain object, passing in the request and response it was called with or the wrapped versions it may have created. Alternatively, the filter can choose to block the request by not making the call to invoke the next entity. In the latter case, the filter is responsible for filling out the response.
- Examine response headers after invoking the next filter in the chain.
- Throw an exception to indicate an error in processing.

## Invoking Other Web Resources

To invoke a resource available on the server that is running a web component, you must first obtain a `RequestDispatcher` object by using the `getRequestDispatcher("URL")` method. You can get a `RequestDispatcher` object from either a request or the web context; however, the two methods have slightly different behavior. The method takes the path to the requested resource as an argument. A request can take a relative path (that is, one that does not begin with a `/`), but the web context requires an absolute path. If the resource is not available or if the server has not implemented a `RequestDispatcher` object for that type of resource, `getRequestDispatcher` will return null. Your servlet should be prepared to deal with this condition.

### 1. Including Other Resources in the Response

```java
include(request, response);
```

### 2. Transferring Control to Another Web Component

To transfer control to another web component, you invoke the `forward` method of a `RequestDispatcher`. When a request is forwarded, the request URL is set to the path of the forwarded page. The original URI and its constituent parts are saved as the following request attributes:

javax.servlet.forward.request_uri
javax.servlet.forward.context_path
javax.servlet.forward.servlet_path
javax.servlet.forward.path_info
javax.servlet.forward.query_string



This text copied from https://docs.oracle.com/javaee/7/tutorial/servlets001.htm