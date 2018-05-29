# scala REST libraries comparison

scalatra
---------

<h4>1) setup/ bootstrap</h4>

- install sbt
- add scalatra dependency
- add scalatra-json dependency
- add json4s dependency

<h4>2) Server backend (to implement HTTP requests and HTTP responses)</h4>

[Jetty](http://scalatra.org/guides/2.5/deployment/standalone.html)

_Eclipse Jetty provides a Web server and javax.servlet container, plus support for HTTP/2, WebSocket etc_

<h4>3) REST API definition/ Routes</h4>

- provides http methods as function. request handler is bit weird as it returns `Any` instead of just `T`. meaning single resource can respond different types. also it should be `action: HttpRequest => [T]`

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
        protected implicit val jsonFormats: Formats = DefaultFormats
      
        protected implicit def executor: ExecutionContext = Implicits.global
      
        post("/api/chat") {
          new AsyncResult() {
            override val is: Future[ApiResponse] = Future {
            
              val correlationId = request.getHeader("correlationId")
              val body = request.body
              
              ApiResponse(correlationId, "hi, how can i help ya")
            }
          }
        }
      }
defined class RestServer

```

<h4>4) Request validation</h4>

Scalatra includes a very sophisticated set of validation commands.

These allow you to parse incoming data, instantiate command objects, and automatically apply validations to the objects. 

http://scalatra.org/guides/2.3/formats/commands.html

<h4>5) REST Error Handling</h4>

provides generic error handler for response 500 as a partial function

```scala

  final case class ApiError(errorCode: String, errorMessage: String)
  
  error {
    case e: Exception => ApiError("InternalApiError", e.getMessage)
  }
  
  ```
  
<h4>6) Request Deserialization/ Response Serialisation</h4>

Uses json4s which default encoders and decoders

http://scalatra.org/guides/2.3/formats/json.html

<h4>7) Exposing API definition</h4>

No a clean way to expose API definition to consumers as headers/ cookies/ req payload are not part of API definition

<h4>8) API http-client</h4>

<h4>9) Performance</h4>

[jetty perf](https://www.techempower.com/benchmarks/#section=data-r15&hw=ph&test=json&l=hra0e7) - 182,147 req/sec

<h4>10) Latency</h4>

jetty - 1.2 ms for above perf

<h4>11) example</h4>

https://github.com/duwamish-os/config-management-ui

play-framework
--------------

<h4>1) setup/ bootstrap</h4>

- install sbt
- add plugin PlayScala
- add play-json

<h4>2) Server backend (to implement HTTP requests and HTTP responses)</h4>

- [Akka-HTTP](https://www.playframework.com/documentation/2.6.x/Server)
- Netty

<h4>3) REST API definition/ Routes</h4>

Two places for endpoint definition. Play provides [routing DSL](https://www.playframework.com/documentation/2.6.x/ScalaSirdRouter) as partial function

```scala
trait Router {
  def routes: Router.Routes
  
  //
}

object Router {
    type Routes = PartialFunction[RequestHeader, Handler]
}
```

- has to read headers/cookies/ request payload etc explicitly inside a Handler function

example, 

Endopint routes

```scala
import api.GameApi
import play.api.libs.json.Json
import play.api.mvc.{Action, AnyContent, RequestHeader}
import play.api.routing._
import play.api.routing.sird._
import play.api.mvc._

class ApiRouter extends SimpleRouter {

  override def routes: PartialFunction[RequestHeader, Action[AnyContent]] = {
    case GET(p"/game/$correlationId") =>
      Action {
        Results.Ok(Json.toJson(GameApi.instance.scores(correlationId)))
      }
  }
}
```
conf/routes

```
->      /api                        route.ApiRouter
```

API

```scala
import play.api.libs.json.{Json, OWrites, Reads}

final case class ApiResponse(correlationID: String, scores: List[Map[String, String]])

object ApiResponse {
  implicit val reads: Reads[ApiResponse] = Json.reads[ApiResponse]
  implicit val writes: OWrites[ApiResponse] = Json.writes[ApiResponse]
}

class GameApi {

  def scores(correlationId: String): ApiResponse = {
    val response = ApiResponse(correlationId, List(
      Map("player" -> "upd",
        "score" -> "1000"),

      Map("player" -> "dude",
        "score" -> "999")))

    response
  }

}

object GameApi {
  lazy val instance = new GameApi
}
```


<h4>4) Request validation</h4>

https://www.playframework.com/documentation/2.6.x/ScalaActionsComposition#Validating-requests


<h4>5) REST Error Handling</h4>

https://www.playframework.com/documentation/2.6.x/ScalaErrorHandling
  
<h4>6) Request Deserialization/ Response Serialisation</h4>

Uses play-json which demands deserializers and serializers for every type

<h4>7) Exposing API definition</h4>

No a clean way to expose API definition to consumers as headers/ cookies/ req payload are not part of API definition

<h4>8) API http-client</h4>

Provides play-ws async http clinet - https://www.playframework.com/documentation/2.6.x/ScalaWS which uses netty-client

<h4>9) Performance</h4>

93,002 req/sec

<h4>10) Latency</h4>

6.3 ms/req

<h4>11) example</h4>

https://github.com/duwamish-os/scala-play-war


finch
------

<h4>1) setup/ bootstrap</h4>

- install sbt
- add finch-core
- add finch-circe

<h4>2) Server backend (to implement HTTP requests and HTTP responses)</h4>

- [Netty](https://netty.io/)

_Netty is a NIO client server framework which enables quick and easy development of network applications such as protocol servers and clients. It greatly simplifies and streamlines network programming such as TCP and UDP socket server._

https://blog.twitter.com/engineering/en_us/a/2014/netty-at-twitter-with-finagle.html

<h4>3) REST API definition/ Routes</h4>

provides clean routing API as a function `Input => EndpointResult[T]` - https://finagle.github.io/finch/user-guide.html#understanding-endpoints

```scala
private[finch] trait EndpointMappers {

  def get[A](e: Endpoint[A]): EndpointMapper[A] = new EndpointMapper[A](Method.Get, e)

  //
}
```

example, 

Endopint routes

```scala
scala> final case class PurchaseRequest(items: List[String])
defined class PurchaseRequest

scala> final case class PurchaseResponse(correlationId: String)
defined class PurchaseResponse
```

```scala
import com.twitter.finagle.http.{Request, Response}
import com.twitter.finagle.{Http, Service}
import com.twitter.util.{Await, Future}
import io.circe.generic.auto._
import io.finch._
import io.finch.circe._
import io.finch.syntax._

import scala.concurrent.ExecutionContext.Implicits.global
import scala.util.{Failure, Success}
import io.finch.syntax.scalaFutures._

scala> object PurchasesServer {
      
          val chatEndpoint: Endpoint[PurchaseResponse] = post("api" :: "purchases" :: header[String]("correlationId") :: jsonBody[PurchaseRequest]) {
            (correlationId: String, purchaseRequest: PurchaseRequest) => {
      
              concurrent.Future {
                PurchaseResponse(correlationId)
              }.map(Ok)
      
            }
      
          }
            
      }
defined object PurchasesServer
```

<h4>4) Request validation</h4>

Provides well designed request validation API

https://finagle.github.io/finch/user-guide.html#validation

```scala
scala> val purchases = post("api" :: "purchases" :: header[Int]("purchaseId").should("be greater than length 5"){_ > 5})
purchases: io.finch.syntax.EndpointMapper[Int] = POST /api :: purchases :: header(purchaseId)
```

<h4>5) REST Error Handling</h4>

Provides error handler (takes partial function) for each API endpoint
  
https://finagle.github.io/finch/user-guide.html#errors

<h4>6) Request Deserialization/ Response Serialisation</h4>

Uses circe which provides compile time derivation of deserializers and serializers

Also provides support for other poular functional json libraries - https://finagle.github.io/finch/user-guide.html#json

<h4>7) Exposing API definition</h4>

Headers/Cookies are part of part of API definition. It helps the API definition to be exposed cleanly to the consumers as a jar dependency with small change

<h4>8) API http-client</h4>

Provides featherbed as async http-client - https://finagle.github.io/featherbed/doc/06-building-rest-clients.html 

<h4>9) Performance</h4>

[396,796 req/sec](https://www.techempower.com/benchmarks/#section=data-r15&hw=ph&test=json&l=hr9u67)

<h4>10) Latency</h4>

6.3 ms

<h4>11) example</h4>

https://github.com/duwamish-os/chat-server
