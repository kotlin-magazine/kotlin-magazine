# Koonlander

<details>

<summary>Layouting and Meta Notes</summary>

This "article" should become a typealong: A thing, that people type directly from the magazine, like in the good old 80ies. 
Therefore it needs roughly 200 lines of code displayed, together with an intro and some informational bubbles (why not 
intellij, how to unix it and how windows terminal works) I'd recommend taking the following chapters and putting them in a 
visually pleasing orientation on a two pager.

The inspriation came from the [usborne.com/](https://usborne.com/gb/books/computer-and-coding-books) books, that are by now
available for free. Especially pages 12 and 13 are where I lifted the algorithm from.

The resulting two pager should be looking something like this:

<img width="200" alt="Moonlander exerpt from the coding magzine" src="https://github.com/mreichelt/kotlin-today-magazine/assets/1162562/28191f9b-ce40-4f9e-8cf1-ac18395358cb">

or

<img width="200" alt="Evil Alien excerpt from the coding magazine" src="https://github.com/mreichelt/kotlin-today-magazine/assets/1162562/c639acce-1fd9-48b2-889d-a85816c8e580">

(both lifted from the page ubsorne)

</details>

# Intro

The operating system of your space ship has just crashed! As the result of using outdated languages and something called
a use after free exception occured and whiped the complete software system out, you need to rebuild the operating system
of the `KSS Pegasus` (Kotlin Software Ship).

Luckily you found the following old drawing of the source code, so you should be able to reconstuct the system, and 
finally land the lander savely.

During your exploration of the code drawing, you'll need to also look at the hints on the side to see how they apply
to your flying sauser.

# HINT 1: Code Editor

Since the software system you are rebuilding will use some graphical powers of ANSI terminals, you need to run the result
in an ANSI compatible terminal. [Sadly IntelliJ does not interpret the needed bits, yet](https://github.com/JetBrains/intellij-community/blob/master/platform/platform-util-io/src/com/intellij/execution/process/AnsiEscapeDecoder.java#L73). 
Recomendation is to either write it there and execute it on the command line, or write it completely in the commandline 
using tools like vim, emacs, nano, pico.

# Code

<details>
  
<summary>Intension</summary>

Intensionally pasted here as pictures, fostering the [ikea effect](https://en.wikipedia.org/wiki/IKEA_effect), and make it harder to just copy and paste the code from the pdf.
</details>

~~~kotlin
@file:JvmName("Koonlander")

import kotlin.math.roundToInt
import State.*

const val ESC = "\u001b"
const val CLS = "$ESC[2J"
const val BLINK = "$ESC[5m"

enum class FG(val c: Int) {
    RED(31),
    GREEN(32),
    YELLOW(33),
    BLUE(34),
    MAGENTA(35),
    CYAN(36),
    WHITE(37),
    BRIGHT_RED(91),
    BRIGHT_GREEN(92),
    BRIGHT_YELLOW(93),
    BRIGHT_BLUE(94),
    BRIGHT_MAGENTA(95),
    BRIGHT_CYAN(96),
    BRIGHT_WHITE(97),
}

fun randomFG(): FG {
    val selected = FG.entries.random()
    return FG.entries.first { e ->
        e.c == selected.c
    }
}

enum class BG(val c: Int) {
    RED(41),
    GREEN(42),
    YELLOW(43),
    BLUE(44),
    MAGENTA(45),
    CYAN(46),
    WHITE(47),
    BRIGHT_RED(101),
    BRIGHT_GREEN(102),
    BRIGHT_YELLOW(103),
    BRIGHT_BLUE(104),
    BRIGHT_MAGENTA(105),
    BRIGHT_CYAN(106),
    BRIGHT_WHITE(107),
}

sealed class State {
    object LANDING: State()
    object BREAKING_MIN: State()
    object BREAKING: State()
    object BREAKING_MAX: State()
    data class CRASHED(val moves:Int): State()
    data class LANDED_WITH_CASUALTIES(val moves:Int): State()
    data class LANDED_SAFELY(val moves:Int): State()
}

fun State.toShip(x: Int, y: Int): String = move(x, y) + when (this) {
    is LANDING -> 128760.utf8
    is BREAKING_MIN -> 128760.utf8 + move(x, y + 1) + "::"
    is BREAKING -> 128760.utf8 + move(x, y + 1) + "^^"
    is BREAKING_MAX -> 128760.utf8 + move(x - 1, y + 1) + color(f = FG.RED, s = "/|\\")
    is CRASHED -> 128165.utf8
    is LANDED_WITH_CASUALTIES -> 128760.utf8 + 128173.utf8
    is LANDED_SAFELY ->
        move(x - 1, y) + 128170.utf8 + 128760.utf8 + 128077.utf8
}

val Int.utf8: String
    get() = Character.toString(this)

fun color(f: FG? = null, b: BG? = null, s: String) =
    "$ESC[${f?.c ?: ""}${if (b != null) ";${b.c}" else ""}m$s$ESC[m"

fun move(x: Number, y: Number) =
    "$ESC[${y.toInt()};${x.toInt()}H"

data class Star(
    val x: Int, val y: Int,
    val c: FG,
    val s: String
)

fun rect(
    x: Int, y: Int,
    w: Int, h: Int,
    border: String, fill: String = ""
) {
    if (fill.isNotEmpty()) {
        for (j in 1 until h - 1) {
            for (i in 1 until w - 1) {
                print(move(x + i, y + j) + fill)
            }
        }
    }

    for (i in 0 until w) {
        print(move(x + i, y + 0) + border)
        print(move(x + i, y + h - 1) + border)
    }

    for (j in 0 until h) {
        print(move(x + 0, y + j) + border)
        print(move(x + w - 1, y + j) + border)
    }
}

fun interact(
    move: Int,
    height: Int,
    velocity: Int,
    fuel: Int,
    update: (height: Int, state: State) -> Unit
): State {
    print(move(23, 7) + "You have ${color(f = FG.GREEN, s = "${fuel}l")} fuel left.")
    print(move(23, 8) + "The ship is ${color(f = FG.YELLOW, s = "${height}m")} above landing and")
    print(move(23, 9) + "it accelerates with a speed of ${color(f = FG.MAGENTA, s = "${velocity}m²")}.")

    var input: Int?
    do {
        print(
            move(23, 12) +
                    color(FG.BRIGHT_YELLOW, BG.BLUE, "How long do you want to fire the engines?") +
                    " "
        )

        input = readln().toIntOrNull()
        if (input == null) {
            print(
                move(23, 13) +
                        color(f = FG.BRIGHT_RED, s = "Not a valid number. Try again.")
            )
        }
    } while (input == null)

    val duration = if (input < 0) {
        0
    } else if (input > 30) {
        30
    } else if (input > fuel) {
        fuel
    } else {
        input
    }

    val v = velocity - duration + 5
    val f = fuel - duration
    val h = height - (v + velocity) / 2

    return if ((v + velocity) / 2 >= height) {
        val v1 = velocity + (5 - duration) * height / velocity
        if (v1 > 5) {
            CRASHED(move + 1)
        } else if (v > 1) {
            LANDED_WITH_CASUALTIES(move + 1)
        } else {
            LANDED_SAFELY(move + 1)
        }
    } else {
        val state = when (duration) {
            in 3..15 -> BREAKING_MIN
            in 15..25 -> BREAKING
            in 25..30 -> BREAKING_MAX
            else -> LANDING
        }

        update(h, state)
        interact(move + 1, h, v, f, update)
    }
}

fun display(stars: List<Star>, height: Int, state: State) {
    print(CLS)

    stars.forEach { s ->
        print(move(s.x, s.y) + color(f = s.c, s = s.s))
    }

    rect(
        1, 14, 20, 3,
        border = color(b = BG.BRIGHT_WHITE, s = " "),
        fill = color(b = BG.WHITE, s = " ")
    )
    rect(
        1, 1, 20, 16,
        border = "+"
    )

    val y = (height / 50.0).roundToInt()
    print(color(s = state.toShip(10, 13 - y)))

    print(move(23, 1) + color(f = FG.RED, s = "Koonlander: A Space Landing Odyssey"))

    print(move(23, 3) + "Your engines just failed. Break your fall by accelerating")
    print(move(23, 4) + "your ship to not crash. Keep an eye on your fuel and")
    print(move(23, 5) + "your speed. The ground is closer then you might think.")
}

fun main() {
    val stars = (0..24).map {
        Star(
            (2..18).random(),
            (2..12).random(),
            randomFG(),
            listOf(10024,/*11088,*/8902).random().utf8,
        )
    }

    display(stars = stars, height = 500, LANDING)

    val state = interact(
        move = 0, height = 500,
        velocity = 50, fuel = 1200
    ) { height, state ->
        display(stars, height, state)
    }

    display(stars, 0, state)
    
    var moves = 0
    when (state) {
        is CRASHED -> {
            rect(
                23, 7, 19, 3,
                color(f = FG.RED, b = BG.BRIGHT_RED, s = "$BLINK!")
            )
            print(
                move(24, 8) +
                        BLINK +
                        color(f = FG.RED, b = BG.BRIGHT_RED, s = " ! You crashed ! ")
            )
            moves = state.moves
        }

        is LANDED_WITH_CASUALTIES -> {
            rect(
                23, 7, 19, 3,
                color(f = FG.YELLOW, b = BG.BRIGHT_YELLOW, s = "+")
            )
            print(
                move(24, 8) +
                        BLINK +
                        color(f = FG.RED, b = BG.BRIGHT_YELLOW, s = "  Injured but ok ")
            )
            moves = state.moves
        }

        is LANDED_SAFELY -> {
            rect(
                23, 7, 19, 3,
                color(f = FG.GREEN, b = BG.BRIGHT_GREEN, s = "~")
            )
            print(
                move(24, 8) +
                        BLINK +
                        color(f = FG.GREEN, b = BG.BRIGHT_GREEN, s = "      LANDED     ")
            )
            moves = state.moves
        }

        else -> {}
    }

    if (state !is CRASHED) {
        println(move(23, 16) + color(f=FG.YELLOW, s="$BLINK You used $moves steps. Can you use less? ") + 128165.utf8)
    } else {
        println(move(23, 16) + color(f=FG.BRIGHT_RED, s="Better luck next time!"))
    }
}


~~~

# HINT 2: Installing Kotlin

Best to also reinstall the `kotlin compiler`. On Mac systems, you'd need to install it by running `brew install kotlin`, after `brew` is installed. On Android, you'd need to install
and fireup `termux` and is installed it using `pkg install kotlin`. On linux, you'd use your favorite package manager / snap / flatpak and issue this line in a terminal: `sudo dnf install kotlin`, 
replacing `dnf` with your manager of choice. On windows, you download a zip from the github releases page: https://github.com/JetBrains/kotlin/releases.

# HINT 3: Building and Running

Folowing Hint 2, you can simply open a terminal to where you saved your recovered source code and call the kotlin compiler named `kotlinc` with the kotlin file you created. I.e. `kotlinc Koonlander.kt`.
If you typed everything correctly, there should be no output from the compiler, and you can continue with the running part. If the compiler complained about something, take a look at your source code
and the source code presented here, maybe something is different?

Running is similarly simple: Assuming you are still in the same folder, you type `kotlin Koonlander` and let the os run for the first time. Can you save the space ship from crashing?

# Disclaimer

Originally written by `Daniel Isaaman` and `Jenny Tyler` in `Computer Spacegames`-magazine from `Usborne Publishing Ltd.` in 1982, downloaded from [usborns public archive](https://usborne.com/gb/books/computer-and-coding-books).
Adapted to Kotlin and minor story changes by `Mario Bodemann`. ❤️
