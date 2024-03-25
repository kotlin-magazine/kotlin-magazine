Identifiers aren’t Services
===========================

By [Jesse Wilson][jesse_wilson], contributor to lots of open-source Kotlin stuff including
[OkHttp][okhttp] and [Zipline][zipline].


The programs I write frequently involve strings that identify things: email
addresses, file paths, URLs, time zones... even credit card numbers and driver's
license numbers.

For many years I followed the patterns of the Java standard library when
creating my own identifiers.

I’ve grown to dislike these patterns!

java.net.URL
------------

I’ll start by complaining about Java’s URL class, [again][url_blog_post]:

```
val url = URL("https://publicobject.com/helloworld.txt")
val content = url.openStream().use { it.readBytes() }
println(content.decodeToString())
```

This does a lot in 3 lines of Kotlin! We identify a URL, fetch its contents, and
print them to the console. But despite its compactness, this code is bad.

There's an HTTP client hiding in the URL class. When I call `openStream()`, that
client is prepared and put to work. I don’t like being cut out of that setup! I
can't dependency-inject my own configured instance for production or a fake in a
test.

`java.net.URL` is serving two competing purposes: as an identifier and as a
service.

Identifiers
-----------

Identifiers are values that we do value-like things with:

 - Accept as input from a person
 - Validate for structure
 - Write to a database, file, or remote process
 - Assert equality in a test case

Nothing is lost when we send an identifier from one program to another.

Services
--------

While it's easy to pass a URL string from one computer to another, we can’t pass
the `InputStream` that reads the response body.

Symmetrically, we can send a timestamp from one computer to another, but we
can’t send the `Clock` that produced it. That's because it's a software
abstraction over a specific quartz crystal that’s bound to the physical world!

java.io.File
------------

Java's original file class both _identifies_ a location on the file system and
also _operates_ on that location.

I might write an Android app that sends its collection of cached images to the
server as a `List<File>`. My server could call `File.delete()` to free up space
on that Android device, but that's not what would happen!

Subclassing `File` is another thing you could do, but shouldn’t:

```
class ImmortalFile(delegate: File) : File(delegate.path) {
  override fun delete() = false
}
```

If you want to write testable code that operates on files, consider
[Okio][okio]!

java.nio.Path
-------------

The new (2011) file system APIs are on the right track, but you have to be
careful to use it in a testable way. Each `Path` has both a path string (the
identifier) and a file system (the service).

This code is implicitly coupled to the default file system:

```
class HelloReader() {
  fun readHello(): String {
    val helloPath = Paths.get("hello.txt")
    return Files.readString(helloPath)
  }
}
```

By changing every call to `Paths.get(...)` with `FileSystem.getPath(...)`, I can
make this testable (such as with [Jimfs](https://github.com/google/jimfs)):

```kotlin
class HelloReader(
  val fileSystem: FileSystem,
) {
  fun readHello(): String {
    val helloPath = fileSystem.getPath("hello.txt")
    return Files.readString(helloPath)
  }
}
```

java.net.InetSocketAddress
--------------------------

I get myself into trouble whenever I use Java's Internet address API:

```
// Wrong! This eagerly looks up an IP address for publicobject.com.
val connectAddress = InetSocketAddress("publicobject.com", 443)
```

The method to use is `createUnresolved()`:

```
val connectAddress = InetSocketAddress.createUnresolved("publicobject.com", 443)
```

Even though I used the same host and port to create these two instances, they
don't `.equals()` each other. Unless I’m offline, in which case they do.


kotlinx.datetime.TimeZone
-------------------------

I need to build a report that summarizes the emails that our service sends each
day. The input is a set of `SentEmail` records:

```
data class SentEmail(
  val customerId: Id,
  val timeZone: TimeZone,
  val locale: Locale,
  val emailAddressId: Id,
  val templateId: TemplateId,
  val enqueuedAt: Instant,
  val deliveredAt: Instant,
)
```

The `TimeZone` class looks like an innocuous way to track a string like
"America/New_York" in a type-safe way.

Unfortunately, Kotlin’s time zone class throws when it's given a time zone that
isn't in its host [JVM's time zone database][jvm_tzdb]. My report will crash if
any customer uses `Europe/Kyiv` (renamed from `Europe/Kiev` in 2022).

```
java.time.zone.ZoneRulesException: Unknown time-zone ID: Europe/Kyiv
  at java.time.zone.ZoneRulesProvider.getProvider(ZoneRulesProvider.java)
  at java.time.zone.ZoneRulesProvider.getRules(ZoneRulesProvider.java)
  at java.time.ZoneRegion.ofId(ZoneRegion.java)
  at java.time.ZoneId.of(ZoneId.java)
  at kotlinx.datetime.TimeZone$Companion.of(TimeZoneJvm.kt)
```

I can fix this crash by updating my JVM to one with more up-to-date time zone
data. Even when I only need an identifier, `TimeZone` always loads the offset
rules.

Advice
------

When using an identifier type like `File`, `InetSocketAddress`, or `TimeZone`,
pay careful attention to what side effects your identifier is triggering.

When writing your own identifier code, please use a data class for the value
part and a separate interface for its related services.


[jesse_wilson]: https://publicobject.com/
[jvm_tzdb]: https://www.oracle.com/java/technologies/tzdata-versions.html
[okhttp]: https://square.github.io/okhttp/
[okio]: https://square.github.io/okio/
[url_blog_post]: https://developer.squareup.com/blog/okhttps-new-url-class/
[zipline]: https://github.com/cashapp/zipline/
