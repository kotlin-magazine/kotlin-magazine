# Kotlynne

## The Future of Kotlin, with the History of English

Chet Haase, May 3, 2024

One of the first joyous experiences that everyone has when starting to use Kotlin is with the for() expression. Having spent years in the same old drudgery of other, similar languages, we all write something like:

```kotlin
for (i = 0; i < n; ++i)
```


And then see the syntax errors highlighted and remember that isn’t quite right. We then remember there’s something about ellipses and write:

```kotlin
for (i = 0 ... n)
```

but that doesn’t work either. We add in more ellipses, take some away, and finally end up doing a web search to be reminded of the more eloquent, explanatory, and prose-like:

```kotlin
for (i in 0 .. n)
```

expression. Now that the syntax errors are gone, we build the code, ship the product, and discover to our joy that this Kotlin expression has enabled a beautiful new off-by-one error potential for us. So we quickly ship a new version of the product with the correct, and even more explanatory:

```kotlin
for (i in 0 until n)
```

Bugs can be unfortunate, as are apps crashing on devices in the field, angry customers, lost revenue, and potential hits to reputation, job, and career, but these risks pale in comparison to the wonder and joy of having re-learned, over and over, the beauty and eloquence of Kotlin’s more natural way of expressing this logic. It’s so easy and natural that we happily learn it again and again, to joyously re-train ourselves out of the rut that we’ve been in for all of these years prior to encountering Kotlin’s beauty, since every other language we’ve ever used does it in that other, single, classic fashion. Thank goodness for Kotlin, shaking things up and opening our eyes to other ways of doing mundane tasks!

It is in this wonderful tradition of the `for` loop’s eloquence that I hereby propose a new Kotlin dialect which expands—nay, expounds—upon this type of expression to other situations throughout the language, in an attempt to create more opportunities for wondrous coding expressions. It does so by dipping into ye Olde Englishe language in a way that code has avoided thus far. By why should we not look to the past to inform the future? As the philosopher said surprisingly long ago, “Those who forget the past are doomed to repeat it until there’s a segmentation fault.”

Here is an introduction to Kotlynne. The full list of new operators and expressions is beyond the scope of this article, but I will whet your appetite with just a few:

### whence

whence is a simple term which begs the question, “How did we get here?” This operator can replace more obtuse mechanisms for getting a stack track (`Thread.dumpStack()`? Please. How gauche!), resulting in a more elegant snippet such as:

```kotlin
println("Lo, upon a summer's day, didst I find myself $whence()")
```

### beseech

`beseech` is not only beautiful to read and to hear (especially in dramatic code readings, which I expect to become a thing once Kotlynne is in common usage throughout the land), but it is also very flexible and functional. Kotlin has multiple different mechanisms for requesting values, or state, or even pausing until such things can be returned to the requester. All of these can be replaced by a simple, and indeed polite, call to `beseech`:

```kotlin
val foo = service.beseech(TARRY)
println("yon value dost equal $foo")
```

for blocking calls, and:

```kotlin
val foo = service.beseech(ANON, (foo) -> { "hither is thing value $foo" })
```

for unblocking calls, which will eventually call the supplied lambda… anon.

### e’er

`const` is a useful (if inelegant) operator in Kotlin. But besides its raw harshness of language used (abbreviations? Really? Tarry we still in the dark days of Unix, limiting command length and readability to optimize storage space and typing speed?), `const` only works with primitive types. `e’er`, however, will not only work for all values; it will do so in a whimsical, poetic way:

```kotlin
e’er val foo = // any type of value whatsoe’er
```

### betwixt

While the beauty of Kotlin’s `for()` syntax began this journey, there is clearly more that can be done for that lonely loop. Why use small words when larger ones can suffice? For example:

```kotlin
for (i betwixt 0 until n)
```

It means the same as the simpler in expression, but it’s obviously far prettier.

### nought
How many bugs have you chased in your job—nay, in your entire career!—that ended up being related to 0? Zero-sized arrays that you mistakenly added items to, iterating down to 0 and then mistakenly beyond, or mistaking yet again whether a given collection’s first member is the 0th or first? All of this is due to 0 itself; it is an abomination in the number system. It is, indeed, the only “value” in the infinite set of numbers which is not actually a number (except for the loathsome case of imaginary numbers; do not get me started on that detestable mess). Zero is, by definition, nothing, where all other numbers are indeed something. So why do we treat zero like everything else, when it is clearly something else entirely?

It is time that we dealt with zero separately, to make it clear when we are dealing with it versus literally _any other number in the universe_. Kotlynne does this by changing the name of 0 to `nought`, making it clear when you are dealing with that specific value (or lack thereof), thus easily avoiding all zero-based problems of the past.

What bugs will come from the use of this new expression? None.

### All’s Well that Ends Swell

That’s all I have time for now. Please tune in next time to learn about some of the other exciting operators and expressions, including `wherefore`, `perchance`, and of course `gallimaufry` in this new exciting dialect:

Kotlynne: A proper language.™


But of course, no treatise on proper Olde Englishe language would be complete without a sonnet.

### An Ode to Code

_With code that reads like books upon the shelf,<br>
And logic that is pleasing to the eye,<br>
I hope that you are pleased as I, myself,<br>
And bid all other languages goodbye._

_For Lo, there is more beauty in this world,<br>
More things of wonder yearning to be seen,<br>
More bits of flags that wait to be unfurled,<br>
And splashed upon each massive 4k screen._

_The language! That’s the thing which we desire,<br>
More crucial than the products that we ship.<br>
‘Tis joy we seek when syntax we acquire,<br>
Until we have that language in our grip.<br>_
_&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;And then we feel life’s passion, even truer<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(Until there is a language that is newer)._
