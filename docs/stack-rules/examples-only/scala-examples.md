# Scala Examples

## 1. Type Safety and Immutability {#type-safety}
### Example 1
```scala
// ❌ BAD - Using null
def findUser(id: String): User = {
  val user = userRepository.findById(id)
  if (user == null) {
    null
  } else {
    user
  }
}

// ❌ BAD - Using Option.get without check
def getUserName(id: String): String = {
  findUser(id).get.name // May throw NoSuchElementException
}

// ✅ GOOD - Proper Option usage
def findUser(id: String): Option[User] = {
  Option(userRepository.findById(id))
}

// ✅ GOOD - Safe Option consumption with getOrElse
def getUserName(id: String): String = {
  findUser(id).map(_.name).getOrElse("Unknown")
}

// ✅ GOOD - Pattern matching
def processUser(id: String): String = {
  findUser(id) match {
    case Some(user) => s"Processing ${user.name}"
    case None => "User not found"
  }
}

// ✅ GOOD - Using fold
def getUserEmail(id: String): String = {
  findUser(id).fold("no-email@example.com")(_.email)
}

// ✅ GOOD - Chaining Option operations
def getUserDepartment(id: String): Option[String] = {
  for {
    user <- findUser(id)
    profile <- user.profile
    department <- profile.department
  } yield department
}
```
### Example 2
```scala
// ❌ BAD - Using var
var count = 0
def increment(): Unit = {
  count += 1
}

// ❌ BAD - Mutable collection
import scala.collection.mutable.ListBuffer
val users = ListBuffer[User]()

// ✅ GOOD - Using val with immutable collection
val users: List[User] = List.empty

// ✅ GOOD - Functional update
def addUser(users: List[User], newUser: User): List[User] = {
  users :+ newUser
}

// ✅ GOOD - Case class with immutability
case class User(
  id: String,
  name: String,
  email: String,
  status: UserStatus
)

// ✅ GOOD - Updating with copy
val user = User("1", "John", "john@example.com", Active)
val updatedUser = user.copy(status = Inactive)

// ✅ GOOD - Immutable state transformation
def processOrders(orders: List[Order]): List[Order] = {
  orders
    .filter(_.status == Pending)
    .map(_.copy(status = Processing))
}
```

## 2. Case Classes and Sealed Traits {#case-classes}
### Example 1
```scala
// ✅ GOOD - Simple case class
case class UserResponse(
  id: String,
  name: String,
  email: String,
  createdAt: Instant
)

// ✅ GOOD - Case class with validation in companion object
case class UserRequest(name: String, email: String)

object UserRequest {
  def create(name: String, email: String): Either[String, UserRequest] = {
    if (name.isBlank) Left("Name cannot be blank")
    else if (!email.contains("@")) Left("Invalid email format")
    else Right(UserRequest(name, email))
  }
}

// ✅ GOOD - Case class with methods
case class Point(x: Double, y: Double) {
  def distanceFromOrigin: Double = Math.sqrt(x * x + y * y)
  def +(other: Point): Point = Point(x + other.x, y + other.y)
}

// ✅ GOOD - Pattern matching with case classes
def processResult(result: Result): String = result match {
  case Success(data) => s"Success: $data"
  case Error(message, _) => s"Error: $message"
  case Loading => "Loading..."
}
```
### Example 2
```scala
// ✅ GOOD - Sealed trait for result types
sealed trait Result[+T]
case class Success[T](data: T) extends Result[T]
case class Error(message: String, cause: Option[Throwable] = None) extends Result[Nothing]
case object Loading extends Result[Nothing]

// Exhaustive pattern matching (compiler ensures all cases covered)
def handleResult[T](result: Result[T]): Unit = result match {
  case Success(data) => logger.info(s"Success: $data")
  case Error(message, cause) => logger.error(s"Error: $message", cause.orNull)
  case Loading => logger.info("Loading...")
  // No default case needed - compiler ensures exhaustiveness
}

// ✅ GOOD - ADT for domain events
sealed trait OrderEvent
case class OrderCreated(orderId: String, customerId: String) extends OrderEvent
case class OrderPaid(orderId: String, amount: BigDecimal) extends OrderEvent
case class OrderShipped(orderId: String, trackingNumber: String) extends OrderEvent
case class OrderCancelled(orderId: String, reason: String) extends OrderEvent

// ✅ GOOD - Using ADT for payment methods
sealed trait PaymentMethod
case class CreditCard(number: String, cvv: String) extends PaymentMethod
case class PayPal(email: String) extends PaymentMethod
case object BankTransfer extends PaymentMethod

def processPayment(method: PaymentMethod, amount: BigDecimal): Unit = method match {
  case CreditCard(number, cvv) => processCreditCard(number, cvv, amount)
  case PayPal(email) => processPayPal(email, amount)
  case BankTransfer => processBankTransfer(amount)
}

// ✅ GOOD - Nested ADTs
sealed trait ApiResponse[+T]
object ApiResponse {
  case class Ok[T](data: T, metadata: Map[String, String] = Map.empty) extends ApiResponse[T]
  
  sealed trait ApiError extends ApiResponse[Nothing]
  case class BadRequest(message: String) extends ApiError
  case class Unauthorized(message: String) extends ApiError
  case class NotFound(resource: String) extends ApiError
  case class ServerError(message: String, cause: Option[Throwable]) extends ApiError
}
```

## 3. For-Comprehensions and Monadic Composition {#for-comprehensions}
### Example 1
```scala
// ✅ GOOD - For-comprehension with Option
def getUserDepartment(userId: String): Option[String] = {
  for {
    user <- findUser(userId)
    profile <- user.profile
    department <- profile.department
  } yield department
}

// ✅ GOOD - For-comprehension with Future
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

def getUserWithOrders(userId: String): Future[UserWithOrders] = {
  for {
    user <- userService.getUser(userId)
    orders <- orderService.getOrdersByUser(userId)
    profile <- profileService.getProfile(userId)
  } yield UserWithOrders(user, orders, profile)
}

// ✅ GOOD - For-comprehension with Either
def processPayment(orderId: String): Either[String, Payment] = {
  for {
    order <- orderRepository.findById(orderId).toRight("Order not found")
    _ <- validateOrder(order)
    payment <- paymentService.process(order.paymentDetails)
  } yield payment
}

// ✅ GOOD - For-comprehension with guards
def getActiveAdminUsers: List[User] = {
  for {
    user <- allUsers
    if user.isActive
    if user.role == Admin
  } yield user
}

// ✅ GOOD - Parallel execution with Future
def getUserDataParallel(userId: String): Future[(User, List[Order], Profile)] = {
  val userFuture = userService.getUser(userId)
  val ordersFuture = orderService.getOrdersByUser(userId)
  val profileFuture = profileService.getProfile(userId)
  
  for {
    user <- userFuture
    orders <- ordersFuture
    profile <- profileFuture
  } yield (user, orders, profile)
}
```

## 4. Error Handling with Either and Try {#error-handling}
### Example 1
```scala
// ❌ BAD - Using exceptions for control flow
def parseAge(input: String): Int = {
  try {
    input.toInt
  } catch {
    case _: NumberFormatException => throw new ValidationException("Invalid age")
  }
}

// ❌ BAD - String error messages
def validateUser(user: User): Either[String, User] = {
  if (user.name.isEmpty) Left("Name is empty")
  else Right(user)
}

// ✅ GOOD - Custom error types
sealed trait ValidationError
case class EmptyField(fieldName: String) extends ValidationError
case class InvalidFormat(fieldName: String, expected: String) extends ValidationError
case class OutOfRange(fieldName: String, min: Int, max: Int) extends ValidationError

def validateAge(input: String): Either[ValidationError, Int] = {
  Try(input.toInt).toEither match {
    case Right(age) if age >= 0 && age <= 150 => Right(age)
    case Right(age) => Left(OutOfRange("age", 0, 150))
    case Left(_) => Left(InvalidFormat("age", "integer"))
  }
}

// ✅ GOOD - Chaining Either operations
def createUser(request: UserRequest): Either[ValidationError, User] = {
  for {
    validName <- validateName(request.name)
    validEmail <- validateEmail(request.email)
    validAge <- validateAge(request.age)
  } yield User(validName, validEmail, validAge)
}

// ✅ GOOD - Pattern matching with Either
def handleResult(result: Either[ValidationError, User]): String = result match {
  case Right(user) => s"User created: ${user.name}"
  case Left(EmptyField(field)) => s"Field '$field' cannot be empty"
  case Left(InvalidFormat(field, expected)) => s"Field '$field' has invalid format, expected: $expected"
  case Left(OutOfRange(field, min, max)) => s"Field '$field' must be between $min and $max"
}

// ✅ GOOD - Converting between Try and Either
import scala.util.{Try, Success, Failure}

def parseJson(json: String): Either[String, JsonObject] = {
  Try(parse(json)).toEither.left.map(_.getMessage)
}
```
### Example 2
```scala
import scala.util.{Try, Success, Failure}

// ✅ GOOD - Using Try for exception-prone operations
def readFile(path: String): Try[String] = {
  Try {
    scala.io.Source.fromFile(path).mkString
  }
}

// ✅ GOOD - Pattern matching with Try
def processFile(path: String): Unit = {
  readFile(path) match {
    case Success(content) => println(s"File content: $content")
    case Failure(exception) => println(s"Error reading file: ${exception.getMessage}")
  }
}

// ✅ GOOD - Using recover for fallback
def readConfigWithDefault(path: String): String = {
  readFile(path).recover {
    case _: FileNotFoundException => "default-config"
  }.get
}

// ✅ GOOD - Converting Try to Either
def parseNumber(input: String): Either[String, Int] = {
  Try(input.toInt).toEither.left.map(_.getMessage)
}
```

## 5. Collections and Higher-Order Functions {#collections}
### Example 1
```scala
// ❌ BAD - Imperative style with loops
var total = 0
for (order <- orders) {
  if (order.status == "completed") {
    total += order.amount
  }
}

// ❌ BAD - Mutable collection
import scala.collection.mutable.ArrayBuffer
val results = ArrayBuffer[Int]()
for (i <- 1 to 10) {
  results += i * 2
}

// ✅ GOOD - Functional style
val total = orders
  .filter(_.status == "completed")
  .map(_.amount)
  .sum

// ✅ GOOD - Map operation
val doubled = (1 to 10).map(_ * 2).toList

// ✅ GOOD - Complex transformation
val usersByDepartment = users
  .filter(_.isActive)
  .groupBy(_.department)
  .view
  .mapValues(_.map(_.name))
  .toMap

// ✅ GOOD - Fold for aggregation
case class Stats(total: Int, count: Int)

val stats = orders.foldLeft(Stats(0, 0)) { (acc, order) =>
  Stats(acc.total + order.amount, acc.count + 1)
}

// ✅ GOOD - Using view for lazy evaluation
val largeList = (1 to 1000000).toList

// Without view - creates intermediate collections
val result1 = largeList.map(_ * 2).filter(_ > 100).take(10)

// With view - lazy evaluation, no intermediate collections
val result2 = largeList.view.map(_ * 2).filter(_ > 100).take(10).toList

// ✅ GOOD - FlatMap for nested operations
val userIds = List("1", "2", "3")
val allOrders: List[Order] = userIds.flatMap(id => getOrdersByUser(id))
```

## 6. Implicit Classes and Extension Methods {#implicits}
### Example 1
```scala
// ✅ GOOD - Implicit class for string extensions
implicit class StringOps(val s: String) extends AnyVal {
  def isValidEmail: Boolean = s.contains("@") && s.contains(".")
  
  def toTitleCase: String = s.split(" ").map(_.capitalize).mkString(" ")
  
  def truncate(maxLength: Int): String = {
    if (s.length <= maxLength) s
    else s.take(maxLength - 3) + "..."
  }
}

// Usage
val email = "user@example.com"
if (email.isValidEmail) {
  println("Valid email")
}

// ✅ GOOD - Implicit class for collection extensions
implicit class ListOps[T](val list: List[T]) extends AnyVal {
  def second: Option[T] = list.drop(1).headOption
  
  def lastOption: Option[T] = list.reverse.headOption
}

// Usage
val numbers = List(1, 2, 3, 4, 5)
println(numbers.second) // Some(2)

// ✅ GOOD - Organizing implicits in objects
object DateTimeImplicits {
  implicit class InstantOps(val instant: Instant) extends AnyVal {
    def isOlderThan(duration: Duration): Boolean = {
      instant.plus(duration).isBefore(Instant.now())
    }
    
    def formatISO: String = instant.toString
  }
}

// Usage
import DateTimeImplicits._
val timestamp = Instant.now()
if (timestamp.isOlderThan(Duration.ofDays(7))) {
  println("More than a week old")
}
```

## 7. Future and Asynchronous Programming {#futures}
### Example 1
```scala
import scala.concurrent.{Future, ExecutionContext, Await}
import scala.concurrent.duration._
import scala.util.{Success, Failure}

// ✅ GOOD - Define ExecutionContext
implicit val ec: ExecutionContext = ExecutionContext.global

// ✅ GOOD - Basic Future usage
def getUserAsync(id: String): Future[User] = Future {
  // Async operation
  userRepository.findById(id)
}

// ✅ GOOD - Chaining Futures
def getUserWithOrders(userId: String): Future[UserWithOrders] = {
  for {
    user <- getUserAsync(userId)
    orders <- getOrdersAsync(userId)
  } yield UserWithOrders(user, orders)
}

// ✅ GOOD - Error handling with recover
def getUserWithDefault(id: String): Future[User] = {
  getUserAsync(id).recover {
    case _: UserNotFoundException => User.guest
    case ex: Exception => 
      logger.error(s"Failed to get user $id", ex)
      User.guest
  }
}

// ✅ GOOD - Parallel execution
def getAllUserData(userIds: List[String]): Future[List[User]] = {
  val futures = userIds.map(getUserAsync)
  Future.sequence(futures)
}

// ✅ GOOD - Timeout handling
import akka.pattern.after
import akka.actor.ActorSystem

implicit val system: ActorSystem = ActorSystem()

def getUserWithTimeout(id: String, timeout: FiniteDuration = 5.seconds): Future[User] = {
  val userFuture = getUserAsync(id)
  val timeoutFuture = after(timeout, system.scheduler) {
    Future.failed(new TimeoutException(s"Request timeout after $timeout"))
  }
  
  Future.firstCompletedOf(Seq(userFuture, timeoutFuture))
}

// ✅ GOOD - Callback handling
getUserAsync("123").onComplete {
  case Success(user) => logger.info(s"User found: ${user.name}")
  case Failure(exception) => logger.error("Failed to get user", exception)
}

// ❌ BAD - Blocking on Future (avoid in production code)
val user = Await.result(getUserAsync("123"), 5.seconds)

// ✅ GOOD - Non-blocking composition
def processUser(id: String): Future[String] = {
  getUserAsync(id).map { user =>
    s"Processing ${user.name}"
  }
}
```

## 8. Type Classes and Implicits {#type-classes}
### Example 1
```scala
// ✅ GOOD - Type class definition
trait JsonSerializer[T] {
  def toJson(value: T): String
}

// ✅ GOOD - Type class instances in companion object
object JsonSerializer {
  implicit val stringSerializer: JsonSerializer[String] = new JsonSerializer[String] {
    def toJson(value: String): String = s""""$value""""
  }
  
  implicit val intSerializer: JsonSerializer[Int] = new JsonSerializer[Int] {
    def toJson(value: Int): String = value.toString
  }
  
  implicit def listSerializer[T](implicit elementSerializer: JsonSerializer[T]): JsonSerializer[List[T]] = {
    new JsonSerializer[List[T]] {
      def toJson(value: List[T]): String = {
        value.map(elementSerializer.toJson).mkString("[", ",", "]")
      }
    }
  }
}

// ✅ GOOD - Using type class with implicit parameter
def serialize[T](value: T)(implicit serializer: JsonSerializer[T]): String = {
  serializer.toJson(value)
}

// ✅ GOOD - Using context bound (syntactic sugar)
def serializeList[T: JsonSerializer](values: List[T]): String = {
  val serializer = implicitly[JsonSerializer[T]]
  values.map(serializer.toJson).mkString("[", ",", "]")
}

// Usage
val jsonString = serialize("Hello") // "Hello"
val jsonInt = serialize(42) // 42
val jsonList = serialize(List(1, 2, 3)) // [1,2,3]
```

## 9. Testing & Tooling {#testing-tooling}
### Example 1
```scala
class UserServiceSpec extends AsyncFlatSpec with Matchers {
  "UserService" should "return the user" in {
    val repo = mock[UserRepository]
    when(repo.find("123")).thenReturn(Future.successful(Some(User("123", "Jane"))))

    val service = new UserService(repo)
    service.get("123").map { result =>
      result.map(_.name) shouldBe Some("Jane")
    }
  }
}
```

## 10. Effect Systems & Resource Safety {#effect-systems}
### Example 1
```scala
def kafkaResource: Resource[IO, KafkaProducer[IO, String, String]] =
  KafkaProducer.resource(producerSettings)

kafkaResource.use { producer =>
  producer.produce(topic, key, value)
}
```

## 11. Streaming & Back-pressure {#streaming}
### Example 1
```scala
def eventsStream(source: Source[Event]): Stream[IO, Processed] =
  source
    .through(validate)
    .evalMap(processEvent)
    .handleErrorWith { err =>
      Stream.eval(logger.error(err)("stream failure")) >> Stream.empty
    }
```

## Tools for Code Quality
### Example 1
```bash
# Scalafmt for code formatting
scalafmt

# Scalafix for linting and refactoring
scalafix

# Wartremover for additional warnings
# Add to build.sbt:
addCompilerPlugin("org.wartremover" %% "wartremover" % "2.4.16")
```
### Example 2
```scala
// Compiler options
scalacOptions ++= Seq(
  "-deprecation",
  "-feature",
  "-unchecked",
  "-Xlint",
  "-Ywarn-dead-code",
  "-Ywarn-numeric-widen",
  "-Ywarn-value-discard"
)

// Scala 3 specific
scalacOptions ++= Seq(
  "-explain",
  "-Xfatal-warnings"
)
```
