quinault-http
-------------

Here's what I expect from a REST library.

1) define endpoints as function of `HttpRequest => Future[Either[HttpResponseError, HttpResponse]]`
2) provide functional way of headers/ cookies/ payload validation (like finch)
3) provide auto decoder and encoder for the endpoints (circe provides that)
4) provide http-client autogen so that clients can simply add the jar and can use the API. obviously provides versioning

API details
------------

<h4>1) setup/ bootstrap</h4>

- install sbt
- add quinault-core
- add quinault-json

<h4>2) Server backend (to implement HTTP requests and HTTP responses)</h4>

- [Netty](https://netty.io/) or akka-http

<h4>3) REST API definition/ Routes</h4>

provides clean routing API as a function `HttpRequest => Future[Either[HttpResponseError[E], HttpResponse[R]]]`

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
      
          val purchaseEndpoint: Future[Either[PurchaseResponseError, PurchaseResponse]] = post("api" :: "purchases" :: header[String]("correlationId") :: jsonBody[PurchaseRequest]) {
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

Uses circe which provides compile time derivation of deserializers and serializers


<h4>7) Exposing API definition</h4>

Headers/Cookies are part of part of API definition. 

It helps the API definition to be exposed cleanly to the consumers as a jar dependency.

<h4>8) API http-client</h4>

Provides featherbed as async http-client - quileute

<h4>9) Performance</h4>


<h4>10) Latency</h4>


<h4>11) example</h4>

