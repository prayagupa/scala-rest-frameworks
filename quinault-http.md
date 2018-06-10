quinault-http
-------------

Summary 
---------

Here's what I expect from a REST library.

1) define endpoints as function of `HttpRequest => Future[Either[HttpResponseError, HttpResponse]]`
2) provide typesafe and functional way to define headers/ cookies/ payload validation as part of route (like in finch)
3) provide auto decoder and encoder for the endpoints (circe provides that)
4) provide typesafe http-client autogen'd so that clients can simply add the jar and can use the API. obviously provides nice versioning. (inspired from jaxrs-client https://docs.oracle.com/javaee/7/tutorial/jaxrs-client001.htm#BABBIHEJ)

API details
------------

<h4>1) setup/ bootstrap</h4>

- install sbt
- add quinault-core
- add quinault-json

<h4>2) Server backend (to implement HTTP requests and HTTP responses)</h4>

- [Netty](https://netty.io/) or akka-http or blaze

<h4>3) REST API definition/ Routes</h4>

Provides clean routing API as a function `HttpRequest => Future[Either[HttpResponseError[E], HttpResponse[R]]]`

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

**client schema upgrade rules**

1) First rule is client jars should NEVER EVER introduce breaking changes, introduce changes in a safe manner.
2.1) if a field type is changing (eg, could be header/cookie/json fields in request/response schema), deprecate old field, introduce a new v2 field with new type and publish jar. API should still support both.

v1
```scala
final case class ChatResponse(message: String, options: Seq[String])
```

v2
```scala
final case class ChatResponse(message: String, @deprecated("please use ChatOption instead, will be dropped in v3", "v1") options: Seq[String], chatOptions: Seq[ChatOption])

final case class ChatOption(option: String)
```

2.2) if a field name is changing do the same, deprecate old field, introduce a new v2 field.
3) once consumers are done upgrading to new fields, drop the old field.

v3
```scala
final case class ChatResponse(message: String, chatOptions: Seq[ChatOption])

final case class ChatOption(option: String)
```

4) enumeration are evils because new values can not just be added without client being able to decode. So,
  Option 1) publish a client jar only with new fields. Once consumers are done implementing new schema, API will implment the changes and populate data on new fields.
  Option 2) publish a client jar with new field for enum along with implmentation on API side, deprecate older field.


```
val purchases : Future[Either[PurchaseResponseError, PurchaseResponse]] = PurchasesServer.purchaseItems(
            purchaseId = 100, 
            payload = """{ "items": ["item1", "item2"] }""")
```

<h4>9) Performance</h4>


<h4>10) Latency</h4>


<h4>11) example</h4>

