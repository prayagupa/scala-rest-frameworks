quinault-http
-------------

Summary 
---------

Here's what I expect from a REST library.

1) define endpoints as function of `HttpRequest => Future[Either[HttpResponseError, HttpResponse]]`
2) provide functional way to define headers/ cookies/ payload validation as part of route (like in finch)
3) provide auto decoder and encoder for the endpoints (circe provides that)
4) provide http-client autogen so that clients can simply add the jar and can use the API. obviously provides versioning. (inspired from jaxrs-client https://docs.oracle.com/javaee/7/tutorial/jaxrs-client001.htm#BABBIHEJ)

API details
------------

<h4>1) setup/ bootstrap</h4>

- install sbt
- add quinault-core
- add quinault-json

<h4>2) Server backend (to implement HTTP requests and HTTP responses)</h4>

- [Netty](https://netty.io/) or akka-http

<h4>3) REST API definition/ Routes</h4>

Provides clean routing API as a function `HttpRequest => Future[Either[HttpResponseError[E], HttpResponse[R]]]`

The problem with play framework for example is the endpoint allows whatever type as response.

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

Same problem with scalatra.

```scala
def get(transformers: RouteTransformer*)(action: => Any): Route = addRoute(Get, transformers, action)
```

So, I expect `quinault-http` to have typed route definition to be something roughly as,

```scala
private[quinault] trait Endpoint {

  def get[A](e: HttpDefinition[R]): Endpoint[Either[HttpResponseError[E], HttpResponse[R]]]

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

scala> final case class PurchaseResponseError(correlationId: String, errorCode: String, errorResponse: String)
defined class PurchaseResponseError
```

```scala
import com.quinault.http.{Request, Response}
import com.quinault.{Http, Service}
import io.circe.generic.auto._
import io.quinault._
import io.quinault.circe._
import io.quinault.syntax._

import scala.concurrent.ExecutionContext.Implicits.global
import scala.util.{Failure, Success}

scala> object PurchasesServer {
      
          def purchaseItems: Future[Either[PurchaseResponseError, PurchaseResponse]] = post("api" :: "purchases" :: header[String]("correlationId") :: jsonBody[PurchaseRequest]) {
            (correlationId: String, purchaseRequest: PurchaseRequest) => {
      
              concurrent.Future { PurchaseResponse(correlationId) }
      
            }
      
          }
            
      }
defined object PurchasesServer
```

<h4>4) Request validation</h4>

Provides well designed request validation API


```scala
scala> val purchases = post("api" :: "purchases" :: header[Int]("purchaseId").should("be greater than 5"){_ > 5})
purchases: io.quinault.syntax.Endpoint[Int] = POST /api :: purchases :: header(purchaseId)
```

<h4>5) REST Error Handling</h4>

Provides error handler (takes partial function) for each API endpoint

TODO

<h4>6) Request Deserialization/ Response Serialisation</h4>

provide compile time derivation of deserializers and serializers. can use circe


<h4>7) Exposing API definition</h4>

Headers/Cookies/payload should be typed and be part of API definition. 

`quinault-http` should help the API definition to be exposed cleanly to the consumers can simply use the jar and use the REST API

<h4>8) API http-client</h4>

quileute will auto-gen the http-client so that the API consumers can use it as jar dependency.


```scala

val purchases : Future[Either[PurchaseResponseError, PurchaseResponse]] = PurchasesServer.purchaseItems(
            purchaseId = 100, 
            payload = """{ "items": ["item1", "item2"] }""")
```

<h4>9) Performance</h4>


<h4>10) Latency</h4>


<h4>11) example</h4>

