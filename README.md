# colorize-scala

Scala microlibrary for ANSI colored strings.

## Installation

```scala
// for normal usage
"org.polyvariant" %% "colorize" % "0.3.0"
// for Scala.js / Scala Native
"org.polyvariant" %%% "colorize" % "0.3.0"
```

## Usage

```scala
import org.polyvariant.colorize._
```

You can colorize any string by calling `.colorize`, `.overlay`, or one of the default helpers (we define one for each color in scala's `AnsiColor` trait):

```scala
"hello".red
// res0: string.ColorizedString = Overlay(
//   underlying = Wrap(s = "hello"),
//   color = Ansi(prefix = "\u001b[31m")
// )
```

If you colorize a string twice, the displayed color will be the one you used first.
This also applies if the string is part of a larger colored string:

```scala
colorize"hello ${"world".blue}.red"
// res1: string.ColorizedString = Concat(
//   lhs = Wrap(s = "hello "),
//   rhs = Concat(
//     lhs = Wrap(s = ""),
//     rhs = Concat(
//       lhs = Overlay(
//         underlying = Wrap(s = "world"),
//         color = Ansi(prefix = "\u001b[34m")
//       ),
//       rhs = Wrap(s = ".red")
//     )
//   )
// )
```

This will render as "hello" in red and "world" in blue.

As you've seen, There's a `colorize` string interpolator which allows you to easily nest colorized strings in each other.
You can also combine colorized strings with `++`:

```scala
"hello ".red ++ "world".blue
// res2: string.ColorizedString = Concat(
//   lhs = Overlay(
//     underlying = Wrap(s = "hello "),
//     color = Ansi(prefix = "\u001b[31m")
//   ),
//   rhs = Overlay(
//     underlying = Wrap(s = "world"),
//     color = Ansi(prefix = "\u001b[34m")
//   )
// )
```

## Rendering

To actually render a colorized string, call `.render`.

```scala
println("hello".red.render) // like this
```

You can customize rendering, currently this is limited to passing a custom color suffix.
Internally, `colorize` works by prepending the desired color sequence to your string and appending a suffix, and `render`'s default suffix is `Console.RESET`.

## RGB color support

If your terminal supports truecolor, you can display text colorized to an exact RGB value.
Instead of the default import, use:

```scala
import org.polyvariant.colorize.trueColor._

"hello".rgb(255, 0, 0).render
// res5: String = "\u001b[38;2;255;0;0mhello\u001b[0m"
```

To automatically detect RGB support, use:

```scala
import org.polyvariant.colorize.auto._

"hello".rgb(255, 0, 0).render
```

If truecolor isn't available, `rgb` will be ignored. You can use this to implement an ANSI fallback:

```scala
// if RGB is ignored, `.red` will still be applied
"hello".rgb(255, 0, 0).red.render
```


## Color removal

To remove all colors and other overlays from a colorized string, use `.dropOverlays`.

## Customization

To apply customizations, you can make your own `colorize` by extending `ConfiguredColorize`. For example:

```scala
import org.polyvariant.colorize.custom._

object myColorize extends ConfiguredColorize(RenderConfig.Default.copy(resetString = "<RESET>"))

import myColorize._

println("hello".overlay("<RED>").render)
// <RED>hello<RESET>
```
