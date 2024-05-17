# Convert HTML to PDF with Kotlin/JVM

## Why Playwright?

Recently, my team encountered the challenge of converting HTML files to PDF without resorting to hacks or outdated CSS versions. Our goal was to find a high-performance, open-source solution that would not keep customers waiting for their generated PDFs. After evaluating several options, we found that none of the popular projects met our requirements.

After two rounds of searching for possible solutions, we found that Gotenberg, Flying Saucer, and PDFBox did not fully satisfy our needs.

However, we discovered [Playwright](https://playwright.dev/), which, although not primarily designed for generating PDFs, provides this functionality. Playwright is originally designed for use with Node.js, but there is also a Java version called Playwright for Java, which works well with Kotlin.

## What about scalability?

After conducting tests, we found that Playwright is highly performant. However, we also required a scalable solution that could handle up to 100 conversions per second. Generating PDFs is a CPU and memory-intensive process, so running 100 instances continuously, even during low traffic hours, was not feasible.   

To address this, I proposed creating a pool of Playwright instances, similar to a pool of database connections. We decided to use Apache Commons Pool for that purpose, which was the right choice.

## Time to code!

Now that we have selected our tools, we can begin writing code. For this article, I will be using Spring Boot 3.2.x, but the example should work with any other framework. Project configuration details will not be covered here, but Gradle imports and tasks can be found in the example repository.

The pool works like following:
1. A client requests an object from the pool using the `borrowObject()` method.
2. The pool checks if there are any idle objects available.
3. If there are idle objects available, the pool returns an idle object to the client.
4. If there are no idle objects available, the pool creates a new object using the `factory` object and returns it to the client.
5. The client uses the object.
6. The client returns the object to the pool using the `returnObject(T obj)` method.
7. The pool checks if the object is still valid using the `evictionPolicy` object.
8. If the object is valid, the pool adds the object to the set of idle objects.
9. If the object is not valid, the pool destroys the object using the `factory` object and removes it from the pool.
10. The client can request another object from the pool using the `borrowObject()` method.
11. The client can close the pool using the `close()` method.
12. The pool releases all resources associated with it.

To implement point 4, we must create a PooledObjectFactory instance that will instruct the pool on how to handle our Playwright instances.

```kotlin
class BrowserContextPooledObjectFactory :
  PooledObjectFactory<BrowserContext>, AutoCloseable {
  // this is a pool storage
  private val playwrightMap = ConcurrentHashMap<BrowserContext, Playwright>()

  // this method is executed after "borrowObject" method call on the pool
  override fun activateObject(p: PooledObject<BrowserContext>) {
    p.getObject()?.clearCookies()
  }

  // this method is executed when the pool evicts idle object
  override fun destroyObject(p: PooledObject<BrowserContext?>) {
    p.getObject()?.run {
      playwrightMap.remove(this)?.close()
    }
  }

  private fun cleanupBrowserContext(browserContext: BrowserContext) {
    browserContext.clearCookies()
    browserContext.clearPermissions()
    val pages = browserContext.pages()
    if (pages.isNotEmpty()) {
      for (page in pages) {
        if (page.isClosed) {
          continue
        }
      }
    }
  }

  // this method is used to create a new instance of the object,
  //  which is added to the pool
  override fun makeObject(): PooledObject<BrowserContext> {
    val playwright = Playwright.create()
    val browserContext = playwright.chromium().launch().newContext()
    playwrightMap[browserContext] = playwright
    return DefaultPooledObject(browserContext)
  }

  // this method is called after "returnObject" method call on the pool
  override fun passivateObject(p: PooledObject<BrowserContext>) {
    p.getObject()?.run {
      clearCookies()
      pages().forEach { page ->
        page?.takeIf { !it.isClosed }?.close()
      }
    }
  }

  // checks if the object can be safely borrowed
  override fun validateObject(p: PooledObject<BrowserContext>) =
    p.getObject() != null

  // executed to close the whole pool of objects,
  //  usually during the app shutdown
  override fun close() =
    playwrightMap.forEach { (browserContext, playwright) ->
      cleanupBrowserContext(browserContext)
      playwright.close()
    }
}
```

The above class is the foundation of our solution. This straightforward implementation is sufficient for use with the pool configuration. Let's now create a bean with the pool:

```kotlin
@Configuration
@ConditionalOnClass(Playwright::class, PooledObjectFactory::class)
class PlaywrightConfig{
  @Bean
  fun browserContextPool() =
    BrowserContextPool(
      BrowserContextPooledObjectFactory(),
      GenericObjectPoolConfig<BrowserContext>().apply {
        jmxEnabled = false
        minIdle = 5
        maxIdle = 10
        maxTotal = 15
        softMinEvictableIdleDuration = Duration.ofMinutes(3)
        timeBetweenEvictionRuns = Duration.ofSeconds(30)
      },
    ).also {
      it.addObjects(it.minIdle)
    }

  class BrowserContextPool(
    factory: BrowserContextPooledObjectFactory,
    config: GenericObjectPoolConfig<BrowserContext>,
  ) : GenericObjectPool<BrowserContext>(factory, config)
}
```

This configuration defines a pool of 5 objects that will be created during startup. These 5 objects will be the minimum amount of instances available to borrow. If necessary, the amount can be increased up to 15 instances, but only a maximum of 10 instances can be in the idle state. Any idle instances will be removed after 3 minutes of not being borrowed again.

## Play(_wright_) time!

To test the solution, use this simple example:

```kotlin
@RestController
class TestController(
  private val browserContextPool: BrowserContextPool,
) {
  @GetMapping("/test")
  suspend fun test(): ResponseEntity<Resource> = ResponseEntity
    .ok()
    .contentType(MediaType.APPLICATION_PDF)
    .header("Content-Disposition", "attachment; filename=test.pdf")
    .body(InputStreamResource(getPdf()))

  private suspend fun getPdf(): InputStream {
    val browserContext = browserContextPool.borrowObject()
    val page = browserContext.newPage()
    page.setContent("<html><body><h1>Hello, World!</h1></body></html>")
    return ByteArrayInputStream(page.pdf())
      .also { browserContextPool.returnObject(browserContext) }
  }
}
```

The function takes a BrowserContext instance, adds HTML code to the first page of the browser window, prints it to PDF, and returns a ByteArray. This ByteArray should be converted to a ByteArrayInputStream and returned from the controller method with proper headers.

## What's next?

The provided implementation and configuration are sufficient for services that do not require generating a high volume of PDFs. However, for optimal concurrent performance, both horizontal and vertical scaling may be necessary. During testing on an M1 Pro processor, 20 browser instances were found to effectively process 20 concurrent requests in 550-650ms, which is a favorable result. My team assumes the same number of instances per Kubernetes pod and autoscales based on CPU usage.

## Useful links:
* The example implementation you can find on my GitHub: https://github.com/marrek13/kotlin-html-to-pdf-demo
* Playwright For Java: https://playwright.dev/java/docs/intro
* Apache Commons Pool: https://commons.apache.org/proper/commons-pool/
