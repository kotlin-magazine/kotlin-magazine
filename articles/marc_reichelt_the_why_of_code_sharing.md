# The _Why_ of Code Sharing

As multiple technologies emerged for code sharing, the _Why_ has changed over time. It's good to take a look back in time - to understand what were the primary reasons for developers to share code, what mistakes have been done in the past, and what we can learn from that for today.

## Simplification

The primary reason for sharing code has always been to _reduce complexity_. Heck, why would we even want to write programs multiple times in different languages? And keep adding even more bugs while we're at it? That's insane! Thankfully, there were technologies available that ran on multiple platforms. Java™ with its slogan "Write Once, Run Anywhere" made it possible to share programs for the desktop. Even with C/C++, much of the code was compiling on multiple platforms. And with Flash and JavaScript, there were technologies available that allowed developers to share programs for the web. Then the mobile revolution happened.

## Saving Cost

Suddenly, developers could not share code with all interesting target platforms anymore. Apple disallowed dynamic runtimes like the JVM on iOS, so Java was not an option. And the web was locked down, too. For example, iOS users and developers had to wait for over 15 years to get push notifications in the browser. Of course, we developers are creative folks - new possibilities for code sharing appeared. PhoneGap/Cordova and other solutions made it possible to package web pages written with HTML, CSS and JavaScript into native apps, essentially getting access to the desired native platform APIs. Decision makers at companies faced the challenge of creating products for 2 platforms: Android and iOS. Developing software is costly, so some of the managers argued: let's _save costs_ by using PhoneGap, so "we'll just write it once"!

Ironically, the decision to write web apps in native packages in order to save costs often backfired. Now developers would need to support 3 tech stacks instead of 2 - driving up complexity. In addition, they would need to learn some native skills anyway in order to fight fires, and debugging bugs was hard - all driving up costs. Plus, customers would often reject these kinds of apps. And rewriting apps costs even more!

## Quality

What went wrong? It appears a very important ingredient was missing. Users were much more satisfied with native apps because they excelled in _quality_: they felt snappy, were consistent with the system UI, and shined with beautiful animations - all by staying close to the system and using the native UI toolkits. The _user experience_ was so much better than any wrapped web app. Frankly, I have not met a wrapped web app that I enjoyed using.
Newer code sharing approaches started tapping into this potential: by using the system UI components instead of rendering web pages like React Native, or by drawing pixels that looked as close to the system UI as possible. For some time, Flutter was the only framework on the market that made this possible.

## Speed

While some approaches like React Native still had some performance issues when crossing the bridge between JavaScript and native, many modern code sharing approaches focus on _speed_ from the beginning. Even though it's not a primary reason for sharing code it deserves its own mention, because it improves the perceived app _quality_.
But it's not just the apps that get faster: modern approaches improve _time-to-market_, and they got fast tooling, too: No matter if it is Flutter's hot reload or Compose previews: faster iteration cycles allow us to ship faster - and it's more fun to code, too!

## Target More Platforms

With big players like Apple and Google limiting what can be published in their stores, we remember that there are platforms with fewer rules. Platforms where we can publish what we like, with no strings attached, where we don't need to give away 30% of all digital sales we make, and - in the case of the web - where we can update our apps pretty much instantly. Developers and decision makers are discovering that being locked into one platform and relying on the gratitude of a platform owner can be bad for them. Today, we rediscover the desktop and the web. Code sharing technologies allow us to _target even more platforms_. With Kotlin Multiplatform and Compose Multiplatform, we can write code and run it on multiple platforms - be it mobile, desktop, the web or on servers.

## Tapping into Existing Potential

Development rarely happens in a vacuum. We are constrained by the skills we have, the codebases that already exist, budgets and more. Sure, it's easy to pick any code sharing technology when starting a greenfield project - but even then we would like to use technologies that we are familiar with. And most of the time, we already have existing codebases, and a complete rewrite is a luxury we can not afford. What if we could _tap into existing potential_? Often, our Android apps are written in Kotlin anyway! That's a big potential to tap into. And with libraries like Room and Lifecycle becoming Multiplatform-ready, that potential grows even larger as the costs for migrating get lower.

But there is more: what if we could _keep great features_ that already exist? It rarely makes sense to rewrite a beautiful UI that has been carefully crafted by iOS and web developers. By having a great interop, with Kotlin Multiplatform we can pick the best tool and the most fitting developer for a job - also for the future!

## Conclusion

So _why_ do we share code? Looking back, we developers always wanted one codebase in the first place in order to _reduce complexity_. But due to the rise of new platforms we had to build apps multiple times. So we started to use code sharing to _save costs_, only to find out that we had to keep up the _quality_ and _speed_ as well. We share code because it allows us to _get to the market faster_. As we discover that the new platforms are limiting us, we are finding more freedom by _targeting more platforms_. And with Kotlin Multiplatform, we can _tap into existing potential_ of our existing Kotlin codebase and rely on our amazing colleagues to do what they do best.

Ultimately, code sharing isn't just about convenience or trend-following. It's about using our resources wisely, empowering innovation, and staying ahead in an ever-evolving tech landscape. In essence, in today's world, not embracing code sharing might just be the real cost we can't afford.