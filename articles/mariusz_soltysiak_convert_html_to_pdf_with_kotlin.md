# Convert HTML to PDF with Kotlin

## Why Playwright?

Recently in my team we faced the problem of how to convert HTML file to PDF without making any hacks or being limited to use old versions of CSS. We also wanted the solution to have a good performance, to not make the customers wait for seconds to receive back the generated PDF. We also wanted the project to be open source, so we can support it when necessary.

Finally we evaluated a lot of solutions, that unfortunately didn't satisfy our needs. Among them we evaluated popular projects: Gotenberg, Flying Saucer, PDFBox. The closest to satisfy our needs was Gotenberg, but we felt we still can achieve a better performance.

After second round of looking for possible solutions, I found [Playwright](https://playwright.dev/). Although its main purpose is not about generating PDF, it is one of the functionality it provides. Playwright originally works with Node.js, but there's a Java version called Playwright For Java, which works good also with Kotlin.

## What about scalability?

After making some tests Playwright happened to be very performant. However, we needed the solution to be also scalable, so it can easily handle up to 100 conversions per second. PDF generation is quite CPU and memory consuming process, so we couldn't just run 100 instances and keep it running, even during low traffic hours. I came up with an idea: What if we could do a pool of playwrights, like a pool of DB connections?_

That was the right choice, and finally we decided to use [Apache Commons Pool](https://commons.apache.org/proper/commons-pool/) for that purpose.

## Time to code!

Now, when we picked the tools we can finally write some code! For the purpose of this article I'll use Spring Boot 3.2.x, but the example with a low effort should work in any other framework. I won't dig into the project configuration here, please check the example repository to find Gradle imports and tasks.

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

We need to focus on the point 4. and create the PooledObjectFactory instance, which will tell the pool how to work with our Playwright instances:

```kotlin
class BrowserContextPooledObjectFactory : PooledObjectFactory<BrowserContext>, AutoCloseable {
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

	// this method is used to create a new instance of the object, which is added to the pool
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
    override fun validateObject(p: PooledObject<BrowserContext>) = p.getObject() != null

	// executed to close the whole pool of objects, usually during the app shutdown
    override fun close() =  
        playwrightMap.forEach { (browserContext, playwright) ->  
            cleanupBrowserContext(browserContext)  
            playwright.close()  
        }
}
```

The class above is the core of our solution. This simple implementation is enough to be now used with the pool configuration. Let's create a bean with the pool now:

```kotlin
@Configuration  
@ConditionalOnClass(Playwright::class, PooledObjectFactory::class)  
class PlaywrightConfig{  
    @Bean  
    fun browserContextPool() =  
        BrowserContextPool(  
            BrowserContextPooledObjectFactory(),  
            GenericObjectPoolConfig<BrowserContext>().also {  
                it.jmxEnabled = false  
                it.minIdle = 5  
                it.maxIdle = 10  
                it.maxTotal = 15  
                it.softMinEvictableIdleDuration = Duration.ofMinutes(3)  
                it.timeBetweenEvictionRuns = Duration.ofSeconds(30)  
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

This configuration defines a pool with 5 objects that will be created during the startup, and then this will be the minimum amount of instances available to borrow. If necessary we increase the amount up to 15 instances, but we allow max 10 instances to be in the idle state. They will be removed after 3 minutes without being borrowed again.

## Play(_wright_) time!

To play with the solution you can use this very simple example:

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
        return ByteArrayInputStream(page.pdf()).also { browserContextPool.returnObject(browserContext) }  
    }  
}
```

What it does, it takes an instance of BrowserContext and puts a HTML code to the first page of the browser window. Then it prints to PDF and returns a ByteArray which needs to be converted to ByteArrayInputStream and such a thing with proper headers can be returned from the controller metod.

## What's next?
Having the simple implementation and the config above is enough if you don't run a very popular service that needs to generate hundreds of PDFs per second. If you care about the concurrent performance, most likely you'll need both horizontal and vertical scaling. On M1 Pro processor I could the most effectively run 20 browser instances and that caused processing 20 concurrent requests in 550-650ms, which is a good result. In my team we assumed the same number per Kubernetes pod and we autoscale based on the CPU usage.

## Useful links:
* The example implementation you can find on my GitHub: https://github.com/marrek13/kotlin-html-to-pdf-demo
* Playwright For Java: https://playwright.dev/java/docs/intro
* Apache Commons Pool: https://commons.apache.org/proper/commons-pool/
