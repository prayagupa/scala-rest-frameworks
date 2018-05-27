# scala-rest-frameworks

scalatra
---------

<h4>1) setup/ bootstrap</h4>

- install sbt
- add scalatra dependency
- add scalatra-json dependency
- add json4s dependency

<h4>2) Server backend (to implement HTTP requests and HTTP responses)</h4>

[Jetty](http://scalatra.org/guides/2.5/deployment/standalone.html)


<h4>3) REST API definition/ Routes</h4>

- provides http methods as function 

```scala
def get(transformers: RouteTransformer*)(action: => Any): Route = addRoute(Get, transformers, action)
```

- has to read headers/cookies/ request payload etc explicitly inside a action function

example, 

```scala
import org.scalatra.{AsyncResult, FutureSupport, ScalatraServlet}
import scala.concurrent.{ExecutionContext, Future}
import org.json4s.{DefaultFormats, Formats}
import org.scalatra.json.JacksonJsonSupport
import org.scalatra.{AsyncResult, FutureSupport, ScalatraServlet}
import scala.concurrent.ExecutionContext.Implicits

scala> final case class ApiResponse(correlationId: String, message: String)
defined class ApiResponse

scala> class RestServer extends ScalatraServlet with JacksonJsonSupport with FutureSupport {
     |   protected implicit val jsonFormats: Formats = DefaultFormats
     | 
     |   protected implicit def executor: ExecutionContext = Implicits.global
     | 
     |   post("/api/chat") {
     |     new AsyncResult() {
     |       override val is: Future[ApiResponse] = Future {
     |         val correlationId = request.getHeader("correlationId")
     |         val body = request.body
     |         ApiResponse(correlationId, "hi, how can i help ya")
     |       }
     |     }
     |   }
     | }
defined class RestServer

```

<h4>4) Request validation</h4>

Scalatra includes a very sophisticated set of validation commands.

These allow you to parse incoming data, instantiate command objects, and automatically apply validations to the objects. 

http://scalatra.org/guides/2.3/formats/commands.html

<h4>5) REST Error Handling</h4>

provides generic error handler

```scala

  final case class ApiError(errorCode: String, errorMessage: String)
  
  error {
    case e: Exception => ApiError("InternalApiError", e.getMessage)
  }
  
  ```
  
<h4>6) Request Deserialization/ Response Serialisation</h4>

Uses json4s which demands deserializers and serializers for every type

http://scalatra.org/guides/2.3/formats/json.html

<h4>7) Exposing API definition</h4>

No a clean way to expose API definition to consumers as headers/ cookies/ req payload are not part of API definition

<h4>8) API http-client</h4>

<h4>9) Performance</h4>

<h4>10) Latency</h4>
