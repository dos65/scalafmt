---
id: configuration
title: Configuration
---

Configuration for scalafmt is defined in a plain text file `.scalafmt.conf`
using [HOCON](https://github.com/lightbend/config) syntax.

Here is an example `.scalafmt.conf`:

```scala config
align.preset = more    // For pretty alignment.
maxColumn = 100 // For my wide 30" display.
```

## Most popular

### `maxColumn`

```scala mdoc:defaults
maxColumn
```

- Keep in mind that 80 characters fit perfectly on a split laptop screen with
  regular resolution.
- GitHub mobile view only shows 80 characters and sometimes you might review
  code on your phone.
- Consider refactoring your code before of choosing a value above 100.

### `assumeStandardLibraryStripMargin`

This parameter simply says the `.stripMargin` method was not redefined by the
user to assign special meaning to indentation preceding the `|` character.
Hence, that indentation can be modified.

```scala mdoc:defaults
assumeStandardLibraryStripMargin
align.stripMargin
```

If `true`, lines starting with the margin character `|` (or another if specified
in the `.stripMargin(...)` call) will be indented differently.

If `align.stripMargin` is true, they will align with the opening triple-quote
`"""` in interpolated and raw string literals. Otherwise, they will be indented
relative to the _start_ of the opening line.

```scala mdoc:scalafmt
assumeStandardLibraryStripMargin = true
align.stripMargin = true
---
val example1 =
  s"""Examples:
  |  * one
  |  * two
  |  * $three
  |""".stripMargin
```

```scala mdoc:scalafmt
assumeStandardLibraryStripMargin = true
align.stripMargin = false
---
val example1 =
  s"""|Examples:
      |  * one
      |  * two
      |  * $three
      |""".stripMargin
```

The pipe character can immediately follow the opening `"""`

```scala mdoc:scalafmt
assumeStandardLibraryStripMargin = true
align.stripMargin = true
---
val example2 =
  s"""|Examples:
  |  * one
  |  * two
  |  * $three
  |""".stripMargin
```

## Scala 3

Scalafmt supports Scala 3 features that can be enabled by changing
the dialect of the Scalafmt parser.

```
runner.dialect = scala3
---
open class MyOpenClass()
val myQuote = '{ expr }
val mySplice = ${ expr }
enum class Weekday {
  case Monday, Tuesday, Wednesday, Thursday, Friday
}
```

Please also see [rewrite rules](#scala3-rewrites) for Scala 3.

Additionally, when using `-Xsource:3` option for Scala 2 you can change the
dialect to `Scala213Source3`, which will allow to format the new syntax ported
from Scala 3.

## Presets

Some sections provide preset values to set multiple parameters at once. These
are always accessed via the `preset` key of the appropriate section, including
top-level.

### Top-level presets

- `preset=default`: this preset is implicit and sets all values to their
  defaults.
- `preset=IntelliJ`: this preset is defined as

```
    preset = default
    indent.defnSite = 2
    optIn.configStyleArguments = false
```

- `preset=defaultWithAlign`: this preset is defined as

```
    preset = default
    align.preset = more
```

- `preset=Scala.js`: this preset is defined as

```
    preset = default
    binPack.preset = true
    align.ifWhileOpenParen = false
    indent.callSite = 4
    docstrings.style = Asterisk
    importSelectors = binPack
    newlines {
      neverInResultType = true
      neverBeforeJsNative = true
      sometimesBeforeColonInMethodReturnType = false
    }
    runner.optimizer {
      forceConfigStyleOnOffset = 500
      forceConfigStyleMinArgCount = 5
    }
```

### Appending to preset collections

If, instead of redefining the default or preset value of a list or a map parameter,
you'd like to append to it, use the following syntax:

```
// set
a.b.c.list = [ ... ]
a.b.c.dict = { ... }

// append; "+" must be the only key
a.b.c.list."+" = [ ... ]
a.b.c.dict."+" = { ... }
```

## Indentation

### `indent.main`

> Since v3.0.0.

This parameter controls the primary code indentation. Various context-specific
overrides are defined below, within this section.

```scala mdoc:defaults
indent.main
```

### `indent.significant`

> Since v3.0.0.

This parameter controls the amount of significant indentation used when
[optional braces](https://dotty.epfl.ch/docs/reference/other-new-features/indentation.html)
rules apply.

By default, equals to `indent.main`.

```scala mdoc:scalafmt
runner.dialect = scala3
indent.main = 2
indent.significant = 3
---
object a {
  if (foo)
    bar
  else
    baz

  if foo then
    bar
    bar
  else
    baz
    baz
}
```

### `indent.callSite`

```scala mdoc:defaults
indent.callSite
```

Example:

```scala mdoc:scalafmt
indent.callSite = 2
---
function(
argument1, // indented by 2
""
)
```

### `indent.defnSite`

```scala mdoc:defaults
indent.defnSite
```

Same as `indent.callSite` except for definition site. Example:

```scala mdoc:scalafmt
indent.defnSite = 4
---
def function(
argument1: Type1 // indented by 4
): ReturnType
```

### `indent.ctorSite`

> Since v2.5.0.

Applies to constructors. Defaults to `indent.defnSite`.

```scala mdoc:scalafmt
indent.ctorSite = 4
indent.defnSite = 2
---
class A(
 field1: Type1 // indented by 4
) {
 def function2(
  argument1: Type1 // indented by 2
 ): ReturnType = None
}
```

### `indent.caseSite`

> Since v3.0.0.

Applies indentation to case values before arrow.

```scala mdoc:defaults
indent.caseSite
```

```scala mdoc:scalafmt
maxColumn = 20
indent.caseSite = 5
---
x match {
  case _: Aaaaaa |
      _: Bbbbbb |
      _: Cccccc =>
}
```

### `indent.extendSite`

```scala mdoc:defaults
indent.extendSite
```

This parameter defines indentation used for the `extends A with B` or `derives A, B` sequences
in a template (class, trait, object, enum, etc.).

```scala mdoc:scalafmt
indent.extendSite = 4
maxColumn = 20
---
trait Foo extends A {
  def foo: Boolean = true
}
```

### `indent.withSiteRelativeToExtends`

> Since v2.5.0.

This parameter defines _additional_ indentation used for the `with` elements of an
`extends A with B` sequence in a template.

```scala mdoc:defaults
indent.withSiteRelativeToExtends
```

```scala mdoc:scalafmt
indent.extendSite = 4
indent.withSiteRelativeToExtends = 2
maxColumn = 30
---
trait Foo extends A with B with C with D with E {
  def foo: Boolean = true
}
```

### `indent.commaSiteRelativeToExtends`

> Since v3.0.0

This parameter defines _additional_ indentation used for the post-comma elements
of an `extends A, B` or `derives A, B` sequences of a template.

Added to support Scala 3, which allows to specify multiple parents with a comma.

```scala mdoc:defaults
indent.commaSiteRelativeToExtends
```

```scala mdoc:scalafmt
runner.dialect = scala3
indent.extendSite = 4
indent.commaSiteRelativeToExtends = 4
maxColumn = 20
---
trait Foo extends A, B, C, D, E {
  def foo: Boolean = true
}
```

### `indentOperator`

Normally, the first eligible break _inside_ a chain of infix operators is
indented by 2.

This group of parameters allows overriding which infix operators are eligible
and controls when indentation is _omitted_.

#### `indentOperator.topLevelOnly`

If true, only top-level infix operators are eligible to be exempted from the
default indentation rule.

```scala mdoc:defaults
indentOperator.topLevelOnly
```

```scala mdoc:scalafmt
indentOperator.topLevelOnly = true
---
function(
  a &&
    b
)
function {
  a &&
    b
}
```

```scala mdoc:scalafmt
indentOperator.topLevelOnly = false
---
function(
  a &&
    b
)
function {
  a &&
    b
}
```

#### `indentOperator.exclude`

Defines a regular expression for excluded infix operators. If an eligible
operator matches, it will not be indented.

```scala mdoc:defaults
indentOperator.exclude
```

#### `indentOperator.include`

Defines a regular expression for included infix operators. If an eligible
operator matches and is not excluded explicitly by
[indentOperator.exclude](#indentoperatorexclude), it be will indented.

```scala mdoc:defaults
indentOperator.include
```

#### `indentOperator.preset`

- `default`
  - use defaults for all fields
- `spray` (also `akka`)
  - set `include = "^.*=$"`, `exclude = "^$"`

## Alignment

Default: **some**

Align has several nested fields, which you can customize. However, it comes with
four possible presets: none, some, more, & most.

### `align.preset`

#### `align.preset=none`

```scala mdoc:scalafmt
align.preset = none
---
x match { // false for case arrows
  case 2  => 22 // also comments!
  case 22 => 222 // don't align me!
}
```

> **Pro tip**: Enable this setting to minimize git diffs/conflicts from
> renamings and other refactorings, without having to ignore whitespace changes
> in diffs or use `--ignore-all-space` to avoid conflicts when git merging or
> rebasing.

> Starting with the introduction of `align.stripMargin` parameter in v2.5.0, one
> must explicitly enable it to get earlier behaviour of `align.preset=none`. See
> [assumeStandardLibraryStripMargin](#assumestandardlibrarystripmargin).

#### `align.preset=some`

```scala mdoc:scalafmt
align.preset = some
---
x match { // true for case arrows
  case 2 => 22
  case 22 => 222
}

val x = 2 // false for assignment
val xx = 22

case object B extends A // false for `extends`
case object BB extends A
```

#### `align.preset=more`

```scala mdoc:scalafmt
align.preset = more
---
val x = 2 // true for assignment
val xx = 22

case object B extends A // true for `extends`
case object BB extends A

q -> 22 // true for various infix operators
qq -> 3   // and also comments!

for {
  x <- List(1) // true for alignment enumerator
  yyy <- List(2)
} yield x ** yyy

x match { // true for multiple tokens across multiple lines
  case 1 => 1 -> 2 // first
  case 11 => 11 -> 22 // second

  // A blank line separates alignment blocks.
  case `ignoreMe` => 111 -> 222
}

// Align assignments of similar type.
def name = column[String]("name")
def status = column[Int]("status")
val x = 1
val xx = 22

// Align sbt module IDs.
libraryDependencies ++= Seq(
  "org.scala-lang" % "scala-compiler" % scalaVersion.value,
  "com.lihaoyi" %% "sourcecode" % "0.1.1"
)
```

#### `align.preset=most`

```scala mdoc:scalafmt
align.preset = most
---
for {
  // align <- with =
  x <- List()
  yyy = 2
  // aligns body by arrow
  zzz <- new Integer {
    def value = 3
  }
} yield x
```

> **Note**. Only for the truest vertical aligners. Feel free to open PR enabling
> more crazy vertical alignment here. Expect changes.

### `align.tokens`

Default: **[caseArrow]**

An align token contains a `code` (the string literal of an operator of token) and a list of
`owners`; each owner entry in turn contains an optional `regex` (the kind of the closest tree
node that owns that token), and a list of `parents` (to match the tree containing the owner of
the token).

> To find the `owner` part for a custom tree, look for its type prefix using
> [ScalaFiddle Playgroud](https://scalameta.org/docs/trees/scalafiddle.html) or
> [AST Explorer](https://scalameta.org/docs/trees/astexplorer.html).

```scala mdoc:scalafmt
align.tokens = [{
  code = "=>"
  owners = [{
    regex = "Case"
  }]
}]
---
x match {
  case 1 => 1 -> 2
  case 11 => 11 -> 22
}
```

```scala mdoc:scalafmt
align.tokens = [
  {
    code = "%"
    owners = [{
      regex = "Term.ApplyInfix"
    }]
  }, {
    code = "%%"
    owners = [{
      regex = "Term.ApplyInfix"
    }]
  }
]
---
val x = List(
"org.scala-lang" %% "scala-compiler" % scalaVersion.value,
"com.lihaoyi" %% "sourcecode" % "0.1.1"
)
```

```scala mdoc:scalafmt
align.tokens."+" = [{
  code = ":"
  owners = [{
    regex = "Term\\.Param"
    parents = [ "Ctor\\.Primary" ]
  }]
}]
---
case class Foo(
  firstParam: Int,
  secondParam: String,
  thirdParam: Boolean
) {
  def Foo(
    firstParam: Int,
    secondParam: String,
    thirdParam: Boolean
  ) = ???
}
```

```scala mdoc:scalafmt
align.tokens."+" = [{
  code = ":"
  owners = [{
    parents = [ "Defn\\." ]
  }]
}]
---
case class Foo(
  firstParam: Int,
  secondParam: String,
  thirdParam: Boolean
) {
  def Foo(
    firstParam: Int,
    secondParam: String,
    thirdParam: Boolean
  ) = ???
}
```

### `align.arrowEnumeratorGenerator`

```scala mdoc:defaults
align.arrowEnumeratorGenerator
```

```scala mdoc:scalafmt
align.arrowEnumeratorGenerator = false
---
for {
  x <- new Integer {
     def value = 2
     }
} yield x
```

```scala mdoc:scalafmt
align.arrowEnumeratorGenerator = true
---
for {
  x <- new Integer {
     def value = 2
     }
} yield x
```

### `align.openParenCallSite`

```scala mdoc:defaults
align.openParenCallSite
```

> Default changed from `true` to `false` in v1.6.

```scala mdoc:scalafmt
align.openParenCallSite = true
---
foo(arg1, arg2)

function(arg1, // align by (
    arg2,
 arg3)

function(
  argument1,
  argument2)
```

```scala mdoc:scalafmt
align.openParenCallSite = false
---
foo(arg1, arg2)

function(arg1, // no align by (
    arg2,
 arg3)

function(
  argument1,
  argument2)
```

### `align.openParenDefnSite`

```scala mdoc:defaults
align.openParenDefnSite
```

> Default changed from `true` to `false` in v1.6.

```scala mdoc:scalafmt
align.openParenDefnSite = true
---
class IntString(int: Int, string: String)

class IntStringLong(int: Int,
    string: String,
  long: Long)
```

```scala mdoc:scalafmt
align.openParenDefnSite = false
---
class IntString(int: Int, string: String)

class IntStringLong(
      int: Int,
    string: String,
  long: Long
)
```

### `align.stripMargin`

See [assumeStandardLibraryStripMargin](#assumestandardlibrarystripmargin).

```scala mdoc:defaults
align.stripMargin
```

This functionality is enabled in all presets except `align.preset=none` where it
was disabled since the parameter's introduction in v2.5.0.

### `align.multiline`

If this flag is set, when alignment is applied, multiline statements will not be
excluded from search of tokens to align.

> Since v2.5.0.

```scala mdoc:defaults
align.multiline
```

```scala mdoc:scalafmt
align.preset = more
align.multiline = true
---
for {
  a <- aaa
  bbb <- bb
  cccccc <- c {
    3
  }
  dd <- ddddd
} yield ()
```

```scala mdoc:scalafmt
align.preset = more
align.multiline = false
---
for {
  a <- aaa
  bbb <- bb
  cccccc <- c {
    3
  }
  dd <- ddddd
} yield ()
```

### `align.allowOverflow`

If this flag is set, as long as unaligned lines did not overflow, we will not check
whether alignment causes any lines to overflow the [`maxColumn`](#maxcolumn) setting.

> Since v3.0.0.

```scala mdoc:defaults
align.allowOverflow
```

It's also enabled by default in `align.preset = most`.

## Newlines

The `newlines.*` options are used to configure when and where `scalafmt` should
insert newlines.

> You might be interested in the [Vertical Multiline](#vertical-multiline)
> section.

### `newlines.source`

> Since v2.5.0.

This parameter controls the general approach to line breaks, and whether to take
into account existing newlines in the source. The default value (if the
parameter is not specified) is the _classic_, original way. Below are the
alternatives.

> These alternatives are EXPERIMENTAL and might change in the future without
> regard to any `edition` settings, until fully released (and this message
> deleted).

#### `newlines.source=keep`

This approach attempts to preserve line breaks in the input whenever possible.

#### `newlines.source=fold,unfold`

These two approaches _completely ignore_ existing line breaks, except around
comments and blank lines (i.e., multiple consecutive newlines).

> Might require increasing runner limits (`runner.optimizer.maxVisitsPerToken`,
> possibly even `runner.maxStateVisits`), to avoid _SearchStateExploded_
> exceptions.

`fold` attempts to remove line breaks whenever possible resulting in a more
horizontal, or vertically compact look.

`unfold`, on the other hand, is intended for those who prefer a more vertical,
or horizontally compact look.

Both settings attempt to play nice with other parameters, but some combinations
are prohibited and will result in an error.

### `newlines.topLevelStatementBlankLines`

> Since v3.0.0.

This parameter controls when to add blank lines before and/or after a top-level
statement (a member of a package or template; nesting is allowed but not within
a block). It does _not_ apply to end markers, imports and non-indented packages.

> This parameter might reduce the number of blank lines but will not eliminate
> them completely.

Each entry on this list consists of the following fields:

- `regex`
  - a regular expression to match the type of the statement
  - if unspecified, will match all valid statements
  - see [align.tokens](#aligntokens) for instructions on how to find the type
- `maxNest`
  - basically, limits indentation level (not actual indentation) of a statement
  - unindented statements (under source-level unindented package) have
    nest level of 0, those under them are 1 etc.
  - if unspecified, will match any nesting level
- `minBreaks` (default: 1)
  - sets the minimum number of line breaks between the first and last line of
    a statement (i.e., one less than the number of lines the statement spans).
  - for instance, `minBreaks=0` will apply to all statements, whereas 1 will
    require at least one line break (that is, a multi-line statement).
- `blanks`
  - consists of two integer fields, `before` and `after`, specifying how many
    blank lines need to be added in the appropriate place
  - if omitted while the entry matches, serves to exclude another entry
  - can be specified as a single integer, to set both:

```
// these two are equivalent
newlines.topLevelStatementBlankLines = [{ blanks { before = 1, after = 1 } }]
newlines.topLevelStatementBlankLines = [{ blanks = 1 }]
```

If you'd like to override or exclude some cases, add them explicitly:

```
newlines.topLevelStatementBlankLines = [
  { maxNest = 0, blanks = 2 } // uses 2 blanks for all unindented statements
  { regex = "^Import" } // excludes import groups; blanks are not specified
]

```

> If multiple entries match a statement, an entry with the lowest `minBreaks`
> will be selected. Since we'll be adding lines, this will increase the span
> of the statement and might potentially lead to another entry, with a higher
> `minBreaks`, to match as well, which is undesirable.

```scala mdoc:defaults
newlines.topLevelStatementBlankLines
```

```scala mdoc:scalafmt
newlines.topLevelStatementBlankLines = []
---
import org.scalafmt
package core { // no newline added here
  class C1 {}
  object O { // nor here
    val x1 = 1
    val x2 = 2
    def A = "A"
    def B = "B"
  } // nor here
  class C2 {}
}
```

```scala mdoc:scalafmt
newlines.topLevelStatementBlankLines = [
  {
    blanks { before = 1 }
  }
]
---
import org.scalafmt
package core {
  class C1 {}
  object O {
    val x1 = 1
    val x2 = 2
    def A = "A"
    def B = "B"
  }
  class C2 {}
}
```

```scala mdoc:scalafmt
newlines.topLevelStatementBlankLines = [
  {
    blanks { after = 1 }
  }
]
---
import org.scalafmt
package core {
  class C1 {}
  object O {
    val x1 = 1
    val x2 = 2
    def A = "A"
    def B = "B"
  }
  class C2 {}
}
```

```scala mdoc:scalafmt
newlines.topLevelStatementBlankLines = [
  {
    blanks = 1
  }
]
---
import org.scalafmt
package core {
  class C1 {}
  object O {
    val x1 = 1
    val x2 = 2
    def A = "A"
    def B = "B"
  }
  class C2 {}
}
```

```scala mdoc:scalafmt
newlines.topLevelStatementBlankLines = [
  {
    minBreaks = 0
    blanks = 1
  }
]
---
package core {
  object O {
    val x1 = 1
    val x2 = 2
    def A =
      "A"
    def B = {
      "B"
    }
  }
}
```

```scala mdoc:scalafmt
newlines.topLevelStatementBlankLines = [
  {
    minBreaks = 2
    blanks = 1
  }
]
---
import org.scalafmt
package core {
  object O {
    val x1 = 1
    val x2 = 2
    def A =
      "A"
    def B = {
      "B"
    }
  }
}
```

### `newlines.topLevelStatements`

> Deprecated in v3.0.0.

This parameter and its companion `newlines.topLevelStatementsMinBreaks` have
been deprecated and will be removed in a subsequent release.

An equivalent setting is

```
newlines.topLevelStatementBlankLines = [
  {
    regex = "^Pkg|^Defn\\.|^Decl\\."
    blanks = 1
    minBreaks = <value of topLevelStatementsMinBreaks>
  }
]
```

### Newlines around package or template body

> Since v3.0.0.

This group of parameters controls whether to enforce a blank line before the first
or after the last statement of a package or template (i.e., body of a class, object,
trait, enum).

> These parameters will not cause any blank lines to be removed.

#### `newlines.topLevelBodyIfMinStatements`

`topLevelBodyIfMinStatements` can be `before` and/or `after`, while
`topLevelBodyMinStatements` limits when the rule is applied.

```scala mdoc:defaults
newlines.topLevelBodyIfMinStatements
newlines.topLevelBodyMinStatements
```

```scala mdoc:scalafmt
newlines.topLevelBodyIfMinStatements = []
---
import org.scalafmt
package core {
  class C1 {
    def one = 1
  }
  object O1 {
    val one = 1
    def two = 2
  }
  class C2 {}
}
```

```scala mdoc:scalafmt
newlines.topLevelBodyIfMinStatements = [before]
---
import org.scalafmt
package core {
  class C1 {
    def one = 1
  }
  object O1 {
    val one = 1
    def two = 2
  }
  class C2 {}
}
```

```scala mdoc:scalafmt
newlines.topLevelBodyIfMinStatements = [after]
---
package core {
  class C1 {
    def one = 1
  }
  object O1 {
    val one = 1
    def two = 2
  }
  class C2 {}
}
```

```scala mdoc:scalafmt
newlines.topLevelBodyIfMinStatements = [before,after]
---
import org.scalafmt
package core {
  class C1 {
    def one = 1
  }
  object O1 {
    val one = 1
    def two = 2
  }
  class C2 {}
}
```

#### `newlines.beforeTemplateBodyIfBreakInParentCtors`

This parameter will force a blank line before the first statement of a template body if the
token _before_ `extends` and the `{` (or, in scala3, `:`) token are not on the same line.

```scala mdoc:defaults
newlines.beforeTemplateBodyIfBreakInParentCtors
```

```scala mdoc:scalafmt
newlines.source = keep
newlines.beforeTemplateBodyIfBreakInParentCtors = true
---
package core {
  class C1 extends S { // no breaks between "C1" and "{"
    def one = 1
  }
  class C1(
    param: Int
  ) extends S { // no breaks between ")" and "{"
    def one = 1
  }
  class C1 extends S { // no breaks between "C1" and "=>"
    self =>
    def one = 1
  }
  class C1
    extends S { // break between "C1" and "{"
    def one = 1
  }
}
```

```scala mdoc:scalafmt
newlines.source = keep
newlines.beforeTemplateBodyIfBreakInParentCtors = false
---
package core {
  class C1 extends S { // no breaks between "C1" and "{"
    def one = 1
  }
  class C1(
    param: Int
  ) extends S { // no breaks between ")" and "{"
    def one = 1
  }
  class C1 extends S { // no breaks between "C1" and "=>"
    self =>
    def one = 1
  }
  class C1
    extends S { // break between "C1" and "{"
    def one = 1
  }
}
```

### `newlines.beforeMultiline`

> Since v2.7.0

This parameter controls whether to force a new line before a multi-line body of
`case/if/def/val` and how to format it if the space is allowed. (For
additional control with assignment expressions, please also see
[`newlines.forceBeforeMultilineAssign`](#newlinesforcebeforemultilineassign) below.)

It accepts the same values as [`newlines.source`](#newlinessource) (and defaults
to that parameter's setting).

NB: for breaks before parameters of a multi-line lambda, use `multiline` with
[`newlines.beforeCurlyLambdaParams`](#newlinesbeforecurlylambdaparams).

```scala mdoc:scalafmt
newlines.beforeMultiline = unfold
---
a match {
  // had space after "=>"
  case a => if (step != 0)
      d.name should be("dir" + step)
  // had newline after "=>"
  case a =>
    if (step != 0)
      d.name should be("dir" + step)
}
```

```scala mdoc:scalafmt
newlines.beforeMultiline = fold
---
a match {
  // had space after "=>"
  case a => if (step != 0)
      d.name should be("dir" + step)
  // had newline after "=>"
  case a =>
    if (step != 0)
      d.name should be("dir" + step)
}
```

```scala mdoc:scalafmt
newlines.beforeMultiline = keep
---
a match {
  // had space after "=>"
  case a => if (step != 0)
      d.name should be("dir" + step)
  // had newline after "=>"
  case a =>
    if (step != 0)
      d.name should be("dir" + step)
}
```

```scala mdoc:scalafmt
# newlines.beforeMultiline = classic
---
a match {
  // had space after "=>"
  case a => if (step != 0)
      d.name should be("dir" + step)
  // had newline after "=>"
  case a =>
    if (step != 0)
      d.name should be("dir" + step)
}
```

### `newlines.forceBeforeMultilineAssign`

> Since v3.0.0

This section controls whether to force a break before a multi-line body of an
assignment expression unless it can be formatted on a single line (or is enclosed
in braces). By default, the rule is disabled. It takes precedence
over `newlines.beforeMultiline` settings.

It can take the following values:

- `never`: the rule is disabled
- `any`: applies to any assignment expression (`def`, assignment to
  a `var`, default value of a method parameter, etc.)
- `def`: applies only to definitions which can potentially be parameterized
  (`def`, `macro`, `given` alias, etc.)
- `anyMember`: applies to members of a `class/trait/object`
- `topMember`: applies to members of a `class/trait/object` which itself
  can only be nested within a sequence of `class/trait/object` definitions

It replaces deprecated `newlines` parameters `beforeMultilineDef=unfold` and
`alwaysBeforeMultilineDef=true` which, if this parameter is not set,
map to `def`.

```scala mdoc:scalafmt
maxColumn = 17
newlines.forceBeforeMultilineAssign = def
---
class A {
  // break, allows params (even if it doesn't define any)
  def foo = func(foo, bar)
  // no break, doesn't allow params
  val foo = func(foo, bar)
  def foo = {
    def a = func(foo, bar)
    val a = func(foo, bar)
  }
}
```

```scala mdoc:scalafmt
maxColumn = 19
newlines.forceBeforeMultilineAssign = topMember
---
class A {
  class B {
    // break, a top member
    def foo = func(foo, bar)
    // break, a top member
    val foo = func(foo, bar)
  }
  def foo = {
    // no break, not a member
    def a = func(foo, bar)
    // no break, not a member
    val a = func(foo, bar)
    new A with B {
      // no break, not a top member
      def foo = func(foo, bar)
      // no break, not a top member
      val foo = func(foo, bar)
    }
  }
}
```

```scala mdoc:scalafmt
maxColumn = 17
newlines.forceBeforeMultilineAssign = never
---
// all disabled, no breaks
class A {
  def foo = func(foo, bar)
  val foo = func(foo, bar)
  def foo = {
    def a = func(foo, bar)
    val a = func(foo, bar)
  }
}
```

### `newlines.beforeTypeBounds`

> Since v3.0.0

This parameter controls formatting of bounds of type parameters: upper `<:`,
lower `>:`, view `<%`, and context `:` bounds. It accepts the
same values as [`newlines.source`](#newlinessource).

- `classic`: simply allows no line breaks (default; can't be specified)
- `keep`: preserves a no-break if the next bound fits on the line
- `fold`: uses a no-break if the next bound fits on the line
- `unfold`: puts all bounds on the same line, or breaks before each

### `newlines.alwaysBeforeElseAfterCurlyIf`

```scala mdoc:defaults
newlines.alwaysBeforeElseAfterCurlyIf
```

```scala mdoc:scalafmt
newlines.alwaysBeforeElseAfterCurlyIf = true
---
if (someCond) {
  foo()
} else {
  bar()
}
```

```scala mdoc:scalafmt
newlines.alwaysBeforeElseAfterCurlyIf = false
---
if (someCond) {
  foo()
}
else {
  bar()
}
```

### `newlines.beforeCurlyLambdaParams`

This parameter controls whether a newline is forced between the opening curly
brace and the parameters of a lambda or partial function. Added in 2.7.0,
replacing boolean `alwaysBeforeCurlyBraceLambdaParams`.

```scala mdoc:defaults
newlines.beforeCurlyLambdaParams
```

```scala mdoc:scalafmt
newlines.beforeCurlyLambdaParams = never
---
// should keep one-line
x.map { x => s"${x._1} -> ${x._2}" }
x.map { case (c, i) => s"$c -> $i" }
// should break on arrow since case doesn't fit on a line
x.zipWithIndex.map { case (c, i) => s"$c -> $i" }
x.zipWithIndex.map { case (c, i) => s"$c -> $i (long comment)" }
```

```scala mdoc:scalafmt
newlines.beforeCurlyLambdaParams = always
---
// should break on brace, though fits on the same line
x.map { x => s"${x._1} -> ${x._2}" }
x.map { case (c, i) => s"$c -> $i" }
x.zipWithIndex.map { case (c, i) => s"$c -> $i (long comment)" }
// should break on brace and arrow as lambda doesn't fit on a line
x.zipWithIndex.map { x => s"${x._1} -> ${x._2} (long comment)" }
```

```scala mdoc:scalafmt
newlines.beforeCurlyLambdaParams = multiline
---
// should keep one-line
x.map { x => s"${x._1} -> ${x._2}" }
x.map { case (c, i) => s"$c -> $i" }
// should break on brace as lambda doesn't fit on the same line
x.zipWithIndex.map { x => s"${x._1} -> ${x._2}" }
x.zipWithIndex.map { case (c, i) => s"$c -> $i" }
// should break on brace and arrow as lambda doesn't fit on a line
x.zipWithIndex.map { x => s"${x._1} -> ${x._2} (long comment)" }
x.zipWithIndex.map { case (c, i) => s"$c -> $i (long comment)" }
```

```scala mdoc:scalafmt
newlines.beforeCurlyLambdaParams = multilineWithCaseOnly
---
// should keep one-line
x.map { x => s"${x._1} -> ${x._2}" }
x.map { case (c, i) => s"$c -> $i" }
// should break after arrow as lambda doesn't fit on the same line
x.zipWithIndex.map { x => s"${x._1} -> ${x._2}" }
x.zipWithIndex.map { x => s"${x._1} -> ${x._2} (long comment)" }
// should break on brace as lambda doesn't fit on the same line
x.zipWithIndex.map { case (c, i) => s"$c -> $i" }
// should break on brace and arrow as lambda doesn't fit on a line
x.zipWithIndex.map { case (c, i) => s"$c -> $i (long comment)" }
```

### `newlines.afterCurlyLambdaParams`

This parameter controls handling of newlines after the arrow following the
parameters of a curly brace lambda or partial function, and whether a space can
be used for one-line formatting of the entire function body (if allowed but the
body doesn't fit, a break is always forced).

This parameter was renamed in 2.7.0 from `afterCurlyLambda`, for clarity and
consistency with `beforeCurlyLambdaParams` defined above.

```scala mdoc:defaults
newlines.afterCurlyLambdaParams
```

```scala mdoc:scalafmt
newlines.afterCurlyLambdaParams = squash
---
// remove all blank lines if any
// one-line formatting is allowed
something.map { x =>



  f(x)
}

something.map { x => f(x) }
```

```scala mdoc:scalafmt
newlines.afterCurlyLambdaParams = never
---
// remove all blank lines if any
// one-line formatting depends on newlines.source:
// yes for fold; no for unfold; otherwise, only if there was no break
something.map { x =>



  f(x)
}

something.map { x => f(x) }
```

```scala mdoc:scalafmt
newlines.afterCurlyLambdaParams = keep
---
// if blank lines are present, keep only one
// one-line formatting depends on newlines.source:
// yes for fold; no for unfold; otherwise, only if there was no break
something.map { x =>



  f(x)
}

something.map { x => f(x) }
```

```scala mdoc:scalafmt
newlines.afterCurlyLambdaParams = always
---
// ensure a single blank line
// one-line formatting is not allowed
something.map { x =>



  f(x)
}

something.map { x => f(x) }
```

### `newlines.implicitParamListModifierXXX`

These parameters control newlines around `implicit` parameter list modifier.

> Since v2.5.0.

#### Newlines around `using` parameter and argument list modifier

`using` soft keyword was introduced in Scala 3 and is supported in `scalafmt`.
Besides parameter lists, `using` can also be used with argument lists hence
provided rules will then also be applied to them.

> Since v3.0.0.

The parameters defined below can also be accessed via their alternative names where
`implicit` is replaced with `using` (for instance, `newlines.usingParamListModifierPrefer`).
Whichever naming you use, formatting will be applied to both `implicit` and `using`.

#### Prefer After (default)

> Prefers newline after `implicit`. Newline will be added unless the entire
> implicit parameter list fits on a line, or config style is false. Newline can
> also be added _before_ if the keyword itself would overflow the line.

```scala mdoc:scalafmt
maxColumn = 60
newlines.implicitParamListModifierPrefer = after
---
def format(code: String, age: Int)(implicit ev: Parser, c: Context): String
```

#### Prefer Before

> Prefers newline before `implicit`. Newline will not be added if the entire
> implicit parameter list fits on a line.

```scala mdoc:scalafmt
maxColumn = 60
newlines.implicitParamListModifierPrefer = before
---
def format(code: String, age: Int)(implicit ev: Parser, c: Context): String
```

#### Force Before

> If set, forces newline before `implicit`. Otherwise, newline can still be
> added if the keyword would overflow the line.

```scala mdoc:scalafmt
maxColumn = 60
newlines.implicitParamListModifierForce = [before]
---
def format(code: String, age: Int)(implicit ev: Parser, c: Context): String
```

#### Force After

> If set, forces newline after `implicit`. Otherwise, newline can still be added
> unless `before` is true, or the entire implicit parameter list fits on a line,
> or config style is false.

```scala mdoc:scalafmt
maxColumn = 60
newlines.implicitParamListModifierForce = [after]
---
def format(code: String, age: Int)(implicit ev: Parser, c: Context): String
```

#### Force both before and after

```scala mdoc:scalafmt
maxColumn = 60
newlines.implicitParamListModifierForce = [before,after]
---
def format(code: String, age: Int)(implicit ev: Parser, c: Context): String
```

#### Implicit with `optIn.configStyleArguments`

While config-style normally requires a newline after the opening parenthesis,
postponing that break until after the `implicit` keyword is allowed if other
parameters require keeping this keyword attached to the opening brace.

Therefore, any of the parameters described in this section will take precedence
even when `optIn.configStyleArguments = true` is used.

### `newlines.afterInfix`

> Since v2.5.0.

This parameter (and its companions) controls formatting around infix
expressions.

> The default value depends on `newlines.source` (see below).

#### `newlines.afterInfix=keep`

This approach preserves line breaks in the input. This is the original
behaviour, and default for `newlines.source=classic,keep`.

#### `newlines.afterInfix=many,some`

These approaches _completely ignore_ existing newlines around infix, always use
a space before an infix operator and occasionally break after it. `some` is
default for `newlines.source=fold`, and `many` for `newlines.source=unfold`.

> Might require increasing runner limits (`runner.optimizer.maxVisitsPerToken`,
> possibly even `runner.maxStateVisits`), to avoid _SearchStateExploded_
> exceptions.

`some` will introduce fewer line breaks than `many`. Both will attempt to break
after
[higher-precedence operators](https://scala-lang.org/files/archive/spec/2.11/06-expressions.html#infix-operations),
and both will _always_ break before an expression enclosed in matching
parentheses.

#### `newlines.afterInfixMaxCountPerFile`

```scala mdoc:defaults
newlines.afterInfixMaxCountPerFile
```

If the total number of infix operations in the _entire file_ exceeds
`newlines.afterInfixMaxCountPerFile`, the formatter automatically switches to
`newlines.afterInfix=keep` for this file.

#### `newlines.afterInfixMaxCountPerExprForSome`

```scala mdoc:defaults
newlines.afterInfixMaxCountPerExprForSome
```

If `newlines.afterInfix` is set to `some` and the number of infix operations in
a _given expression sequence_ (top-level or enclosed in parens/braces) exceeds
`newlines.afterInfixMaxCountPerExprForSome`, the formatter switches to `many`
for that sequence only.

#### `newlines.afterInfixBreakOnNested`

```scala mdoc:defaults
newlines.afterInfixBreakOnNested
```

If enabled, will force line breaks around a nested parenthesized sub-expression
in a multi-line infix expression.

### `newlines.avoidForSimpleOverflow`

A list parameter (of comma-separated flags), with possible flags described
below. These flags relax formatting rules to allow occasional line overflow
(i.e., when line exceeds `maxColumn`) in simple cases instead of introducing a
newline.

```scala mdoc:defaults
newlines.avoidForSimpleOverflow
```

#### `newlines.avoidForSimpleOverflow=[tooLong]`

> Since v2.6.0.

This flag tries to avoid introducing a newline if the line would overflow even
with a newline.

```scala mdoc:scalafmt
maxColumn = 50
danglingParentheses.callSite = false
newlines.avoidForSimpleOverflow = [tooLong]
---
object Example {
  foo_bar_baz("the quick brown fox jumps over the lazy dog") {
    println("")
  }
  foo_bar_baz("the quick brown fox jumps over a dog") {
    println("")
  }
}
```

#### `newlines.avoidForSimpleOverflow=[punct]`

> Since v2.6.0.

This flag tries to avoid a newline if the line would overflow only because of
trailing punctuation (non-alphanum symbols of length 1).

With the flag set:

```scala mdoc:scalafmt
maxColumn = 80
newlines.avoidForSimpleOverflow = [punct]
---
class Engine[TD, EI, PD, Q, P, A](
    val dataSourceClassMap: Map[
      String,
      Class[_ <: BaseDataSource[TD, EI, Q, A]]]) {}
```

### `newlines.avoidInResultType`

If true, newlines in definition result type will only be used if formatting
without them is impossible. This parameter was added in 2.7.0, replacing
`neverInResultType`.

```scala mdoc:defaults
newlines.avoidInResultType
```

```scala mdoc:scalafmt
maxColumn = 40
newlines.avoidInResultType = true
newlines.neverBeforeJsNative = true
---
// no newlines in result type
def permissionState(a: A = js.native): js.Promise[PushPermissionState] = js.native
// no newlines in result type
val permissionState: js.Promise[PushPermissionState] = js.native
// can't format without newlines
implicit protected val td: TildeArrow {
  type Out = RouteTestResult } = TildeArrow.injectIntoRoute
```

## Newlines: `danglingParentheses`

While this parameter is not technically under the `newlines` section, it
logically belongs there.

### `danglingParentheses.defnSite`

```scala mdoc:defaults
danglingParentheses.defnSite
```

```scala mdoc:scalafmt
danglingParentheses.defnSite = true
danglingParentheses.callSite = false
maxColumn=25
---
object a {
  // defnSite
  def method(a: Int, b: String): Boolean

  // callSite
  method(argument1, argument2)
}
```

### `danglingParentheses.callSite`

```scala mdoc:defaults
danglingParentheses.callSite
```

```scala mdoc:scalafmt
danglingParentheses.defnSite = false
danglingParentheses.callSite = true
maxColumn=25
---
object a {
  // defnSite
  def method(a: Int, b: String): Boolean

  // callSite
  method(argument1, argument2)
}
```

### `danglingParentheses.ctrlSite`

> Since v2.5.0.

Forces dangling on open/close parens around control structures (`if`, `while`,
`for`) when line breaks must occur.

```scala mdoc:defaults
danglingParentheses.ctrlSite
```

```scala mdoc:scalafmt
danglingParentheses.ctrlSite = true
maxColumn=20
---
if (something) {
  // nothing
}
if (something_else) {
  // nothing
}
```

### `danglingParentheses.exclude`

> Since v2.5.0.

When the appropriate `danglingParentheses` flag (e.g., `defnSite`) has been set,
this parameter can be used to limit contexts where dangling is applied
(currently, `class`, `trait`, `enum`, `extension` and `def` are supported).

```scala mdoc:defaults
danglingParentheses.exclude
```

```scala mdoc:scalafmt
indent.defnSite = 2
danglingParentheses.defnSite = true
danglingParentheses.exclude = [def]
---
def other(a: String, b: String)(c: String, d: String) = a + b + c
other(a, b)(c, d)
```

## Newlines: Config-style formatting

This formatting applies to argument lists in class definitions and method calls.
It normally involves a newline after the opening parenthesis (or after the
`implicit` keyword) and a newline before the closing parenthesis.

As part of the formatting output, arguments are output one per line (but this is
not used in determining whether the source uses config-style formatting).

While this parameter is not technically under the `newlines` section, it
logically belongs there.

### `optIn.configStyleArguments`

If true, applies config-style formatting:

- if single-line formatting is impossible
- if the source uses config-style and `newlines.source = classic/keep`
- if other parameters force config-style (see below)

```scala mdoc:defaults
optIn.configStyleArguments
```

```scala mdoc:scalafmt
optIn.configStyleArguments = true
maxColumn=45
---
object a {
  // keeps single line
  def method1(a: Int, b: String): Boolean

  // forces config style
  def method2(a: Int, b: String, c: String): Boolean

  // preserves config style
  def method3(
    a: Int, b: String, c: String
  ): Boolean
}
```

### Forcing config style

Controls parameters which trigger forced config-style formatting. All conditions
must be satisfied in order for this rule to apply.

```scala mdoc:defaults
runner.optimizer.forceConfigStyleOnOffset
runner.optimizer.forceConfigStyleMinArgCount
```

- `runner.optimizer.forceConfigStyleOnOffset`: applies to method calls; if
  positive, specifies the minimum character distance between the matching
  parentheses, excluding any whitespace
- `runner.optimizer.forceConfigStyleMinArgCount` applies to method calls;
  specifies the minimum number of arguments

```scala mdoc:scalafmt
optIn.configStyleArguments = true
runner.optimizer.forceConfigStyleOnOffset = 5
runner.optimizer.forceConfigStyleMinArgCount = 2
maxColumn = 60
---
object a {
  // this is a definition, not a method call
  def method(a: String, b: String = null): Boolean

  // keeps single line; min offset not satisfied
  method(a, b)

  // keeps single line; min arg not satisfied
  method(SomeVeryVeryVeryVeryLongArgument)

  // forces config style
  method(foo, bar)
}
```

## Rewrite Rules

To enable a rewrite rule, add it to the config like this
`rewrite.rules = [SortImports]`.

### `AvoidInfix`

```scala mdoc:scalafmt
rewrite.rules = [AvoidInfix]
rewrite.neverInfix.excludeFilters."+" = [ "map" ]
---
a success b
a error (b, c)
a map { x =>
  x + 2
}
"o" % "a" % "v" c(D)
future map {
  case e: Err => 0
} recover (_.toString)
future recover {
  case e: Err => 0
} map (_.toString)
```

### `ExpandImportSelectors`

```scala mdoc:scalafmt
rewrite.rules = [ExpandImportSelectors]
---
import a.{
    b,
    c
  }, h.{
    k, l
  }
import d.e.{f, g}
import a.{
    foo => bar,
    zzzz => _,
    _
  }
```

### `RedundantBraces`

> Warning. This rewrite can cause non-idempotent formatting, see
> [#1055](https://github.com/scalameta/scalafmt/issues/1055).

```scala mdoc:scalafmt
rewrite.rules = [RedundantBraces]
---
def foo = {
  List(1, 2, 3).sum
}
```

```scala mdoc:scalafmt
rewrite.rules = [RedundantBraces]
rewrite.redundantBraces.stringInterpolation = true
---
q"Hello ${name}"
```

```scala mdoc:scalafmt
rewrite.rules = [RedundantBraces]
---
List(1, 2, 3).map { x => x + 1 }
```

Entire power of `RedundantBraces` can be accessed with
`newlines.afterCurlyLambdaParams=squash`. It will try to squash lambda body in
one line and then replace braces with parens:

```scala mdoc:scalafmt
rewrite.rules = [RedundantBraces]
newlines.afterCurlyLambdaParams=squash
---
List(1, 2, 3).map { x =>
  x + 1
}

List(1, 2, 3).map { x =>
  println("you can't squash me!")
  x + 1
}
```

#### Configuration options

```scala mdoc:defaults
rewrite.redundantBraces.generalExpressions
```

```scala mdoc:scalafmt
rewrite.rules = [RedundantBraces]
rewrite.redundantBraces.generalExpressions = true
---

while (x < 10) {
  x += 1
}

str match {
  case "a" => {
    println("ok")
  }
  case _ => {
    println("not ok")
  }
}
```

```scala mdoc:defaults
rewrite.redundantBraces.ifElseExpressions
```

```scala mdoc:scalafmt
rewrite.rules = [RedundantBraces]
rewrite.redundantBraces.ifElseExpressions = true
---

if (a > b) {
  doSomething()
} else {
  doAnything()
}
```

```scala mdoc:defaults
rewrite.redundantBraces.methodBodies
```

```scala mdoc:scalafmt
rewrite.rules = [RedundantBraces]
rewrite.redundantBraces.methodBodies = true
---
def f() = {
  1 + 1
}
```

```scala mdoc:defaults
rewrite.redundantBraces.includeUnitMethods
```

Affects only functions with **explicitly** specified `Unit` type

```scala mdoc:scalafmt
rewrite.rules = [RedundantBraces]
rewrite.redundantBraces.methodBodies = true
rewrite.redundantBraces.includeUnitMethods = false
---
def f() = {
  1 + 1
}

def x(): Unit = {
  println("example")
}
```

```scala mdoc:defaults
rewrite.redundantBraces.stringInterpolation
```

```scala mdoc:scalafmt
rewrite.rules = [RedundantBraces]
rewrite.redundantBraces.stringInterpolation = true
---
s"user id is ${id}"
```

`rewrite.redundantBraces.parensForOneLineApply` is `true` by default for
`edition` >= 2020-01. See also
[newlines.afterCurlyLambdaParams = squash](#newlinesaftercurlylambdaparams).

```scala mdoc:scalafmt
rewrite.rules = [RedundantBraces]
rewrite.redundantBraces.parensForOneLineApply = true
---
xs.map { x => x + 1 }
```

```scala mdoc:defaults
rewrite.redundantBraces.maxLines
```

```scala mdoc:scalafmt
rewrite.rules = [RedundantBraces]
rewrite.redundantBraces.maxLines = 3
---
def f() = {
  collection
    .map(x => x + 1)
    .filter(_ < 10)
    .map(_ * 2)
}

def f() = {
  collection
    .map(x => x + 1)
    .filter(_ < 10)
    .map(_ * 2)
    .headOption
}
```

### `RedundantParens`

```scala mdoc:scalafmt
rewrite.rules = [RedundantParens]
---
for {
  a <- b
  if (a.nonEmpty)
} yield a

val z = (insertData *> readDatabase(id))
```

### `SortModifiers`

Modifiers are sorted based on the given order. Affects modifiers of the
following definitions: trait, class, object, type, and val+var, both as fields
and class parameters.

```scala mdoc:scalafmt
rewrite.rules = [SortModifiers]
---
final lazy private implicit val x = 42
lazy final implicit private val y = 42
```

```scala mdoc:scalafmt
rewrite.rules = [SortModifiers]
---
class Test(
    implicit
    final private val i1: Int,
    private final val i2: String
)
```

```scala mdoc:scalafmt
rewrite.rules = [SortModifiers]
---
sealed protected[X] trait ADT
final private case object A1 extends ADT
private final case class A2(a: Int)
    extends ADT
```

If you choose the non-default sort order then you have to specify all eight
modifiers in the order you wish to see them. Hint: since some modifiers are
mutually exclusive, you might want to order them next to each other.

```scala mdoc:scalafmt
rewrite.rules = [SortModifiers]
rewrite.sortModifiers.order = [
  "implicit", "final", "sealed", "abstract",
  "override", "private", "protected", "lazy"
]
---
override implicit final val x = 2
```

### `PreferCurlyFors`

Replaces parentheses into curly braces in for comprehensions that contain
multiple enumerator generators.

```scala mdoc:scalafmt
rewrite.rules = [PreferCurlyFors]
---
for(a <- as; b <- bs if b > 2)
 yield (a, b)
```

### `SortImports`

The imports are sorted by the groups: symbols, lower-case, upper-case.

```scala mdoc:scalafmt
rewrite.rules = [SortImports]
---
import foo.{Zilch, bar, Random, sand}
```

### `AsciiSortImports`

The imports are sorted by their Ascii codes

```scala mdoc:scalafmt
rewrite.rules = [AsciiSortImports]
---
import foo.{~>, `symbol`, bar, Random}
```

### Trailing commas

> See [SIP](https://docs.scala-lang.org/sips/trailing-commas.html)

The rule handles how trailing commas are treated in case of a dangling closing
delimiter (parenthesis or bracket for definitions or invocations, brace for
import statements only).

Regardless of the setting, trailing commas are always removed if the closing
delimiter is not dangling (i.e., follows the final argument without a line
break).

> This logic is not triggered via the `rewrite.rules` parameter, but by setting
> `trailingCommas`.

```scala mdoc:defaults
trailingCommas
```

#### Trailing commas: `never`

Makes sure there are no trailing commas:

```scala mdoc:scalafmt
trailingCommas = never
---
import a.{
  b,
  c,
}
def method1(
  a: Int,
  b: Long,
) = {}
def method2(
  a: Int,
  b: Long*,
) = {}
def method3(
  a: Int,
) = {}
method1(
  a,
  b,
)
method2(
  a,
  b: _*,
)
method3(
  a,
)
```

#### Trailing commas: `keep`

Keeps any trailing commas:

```scala mdoc:scalafmt
trailingCommas = keep
---
import a.{
  b,
  c,
}
def method1(
  a: Int,
  b: Long,
) = {}
def method2(
  a: Int,
  b: Long*
) = {}
def method3(
  a: Int,
) = {}
method1(
  a,
  b
)
method2(
  a,
  b: _*,
)
method3(
  a,
)
```

#### Trailing commas: `always`

Makes sure there are trailing commas:

```scala mdoc:scalafmt
trailingCommas = always
---
import a.{
  b,
  c
}
def method1(
  a: Int,
  b: Long
) = {}
def method2(
  a: Int,
  b: Long*
) = {}
def method3(
  a: Int
) = {}
method1(
  a,
  b
)
method2(
  a,
  b: _*
)
method3(
  a
)
```

#### Trailing commas: `multiple`

> Since v2.5.0.

Makes sure there are trailing commas for multiple-argument expressions only,
except when the last argument is repeated:

```scala mdoc:scalafmt
trailingCommas = multiple
---
import a.{
  b,
  c
}
def method1(
  a: Int,
  b: Long
) = {}
def method2(
  a: Int,
  b: Long*,
) = {}
def method3(
  a: Int,
) = {}
method1(
  a,
  b
)
method2(
  a,
  b: _*,
)
method3(
  a,
)
```

## Scala3 rewrites

This section describes rules which are applied if the appropriate dialect (e.g.,
`runner.dialect = scala3`) is selected.

> This logic is not triggered via the `rewrite.rules` parameter, but by setting
> parameters under `rewrite.scala3` subsection.

### `rewrite.scala3.convertToNewSyntax`

If this flag is enabled, the following new syntax will be applied:

- [control syntax](https://dotty.epfl.ch/docs/reference/other-new-features/control-syntax.html)
- [vararg splices](https://dotty.epfl.ch/docs/reference/changed-features/vararg-splices.html)
- [imports](https://dotty.epfl.ch/docs/reference/changed-features/imports.html)

### `rewrite.scala3.removeOptionalBraces`

If this flag is enabled,
[optional braces](https://dotty.epfl.ch/docs/reference/other-new-features/indentation.html)
will be removed and significant indentation applied.

The flag takes the following values:

- `no`: disabled
- `yes`: applies to expressions using the new control syntax (or
  `rewrite.scala3.convertToNewSyntax` is set)
- `oldSyntaxToo`: applies also to expressions using deprecated syntax

### `rewrite.scala3.insertEndMarkerMinLines`

If this flag is set to a positive value, when an expression containing an
[optional braces](https://dotty.epfl.ch/docs/reference/other-new-features/indentation.html)
region spans at least as many lines and isn't followed by an end marker, one will be inserted.

> The lines computation ignores additional lines in multiline strings or comments.

> We will not insert end markers if the statement is not part of a template body or
> a multiline block. Doing so might turn a single-stat expression (which doesn't
> require significant indentation handling) into a multi-stat block.

## Vertical Multiline

Since: v1.6.0.

If enabled this formats methods such that parameters are on their own line
indented by [`indent.defnSite`](#indentdefnsite).
Separation between parameter groups are indented by two spaces less than
`indent.defnSite`. The return type is on its own line at the end.

> This formatting is only triggered if the method definition exceeds the
> maxColumn value in width or if the number of arguments to the method exceeds
> the [`verticalMultiline.arityThreshold`](#verticalmultilinearitythreshold).

### `verticalMultiline.arityThreshold`

```scala mdoc:defaults
verticalMultiline.arityThreshold
```

```scala mdoc:scalafmt
verticalMultiline.atDefnSite = true
verticalMultiline.arityThreshold = 2
---
case class Foo(x: String)
case class Bar(x: String, y: String)
object A {
  def foo(x: String, y: String)
  def hello(how: String)(are: String)(you: String) = how + are + you
}
```

### `verticalMultiline.newlineAfterOpenParen`

```scala mdoc:defaults
verticalMultiline.newlineAfterOpenParen
```

```scala mdoc:scalafmt
indent.defnSite = 2
verticalMultiline.atDefnSite = true
verticalMultiline.arityThreshold = 2
verticalMultiline.newlineAfterOpenParen = true
---
def other(a: String, b: String)(c: String, d: String) = a + b + c
```

### `verticalMultiline.excludeDanglingParens`

> This parameter has been deprecated, please use
> [danglingParentheses.exclude](#danglingparenthesesexclude). Keep in mind,
> though, that the new parameter is empty by default while the old one isn't, so
> to use empty exclude list, one must set the old
> `verticalMultiline.excludeDanglingParens=[]`.

```scala mdoc:defaults
verticalMultiline.excludeDanglingParens
```

```scala mdoc:scalafmt
indent.defnSite = 2
verticalMultiline.excludeDanglingParens = [def]
verticalMultiline.atDefnSite = true
verticalMultiline.arityThreshold = 2
verticalMultiline.newlineAfterOpenParen = true
---
def other(a: String, b: String)(c: String, d: String) = a + b + c
other(a, b)(c, d)
```

### Vertical multiline with `implicit` parameter lists

> Also see the general section on
> [implicit parameter lists](#newlinesimplicitparamlistmodifierxxx).

#### Before only

```scala mdoc:scalafmt
maxColumn = 60
verticalMultiline.atDefnSite = true
newlines.implicitParamListModifierForce = [before]
---
def format(code: String, age: Int)(implicit ev: Parser, c: Context): String
```

#### After only

```scala mdoc:scalafmt
maxColumn = 60
verticalMultiline.atDefnSite = true
newlines.implicitParamListModifierForce = [after]
---
def format(code: String, age: Int)(implicit ev: Parser, c: Context): String
```

#### Before and after

```scala mdoc:scalafmt
maxColumn = 60
verticalMultiline.atDefnSite = true
newlines.implicitParamListModifierForce = [before,after]
---
def format(code: String, age: Int)(implicit ev: Parser, c: Context): String
```

## Comment processing

### `comments.wrap`

> Since v2.6.0.

Allows wrapping comments exceeding `maxColumn`.

```scala mdoc:defaults
comments.wrap
```

#### `comments.wrap = standalone`

A standalone comment is one which is surrounded by line breaks.

```scala mdoc:scalafmt
maxColumn = 20
comments.wrap = standalone
---
/* long multiline comment */
// long singleline comment
val a = 1 // short
val b = 2 // long singleline comment
```

#### `comments.wrap = trailing`

A trailing comment is one which is followed by a line break.

```scala mdoc:scalafmt
maxColumn = 20
comments.wrap = trailing
---
/* long multiline comment */
// long singleline comment
val a = 1 // short
val b = 2 // long singleline comment
```

### `comments.wrapStandaloneSlcAsSlc`

> Since v2.6.0.

This parameter allows formatting a standalone single-line comment (i.e., `//`)
to be wrapped using the same type, not a multi-line comment (`/* ... */`).

```scala mdoc:defaults
comments.wrapStandaloneSlcAsSlc
```

```scala mdoc:scalafmt
maxColumn = 20
comments.wrap = trailing
comments.wrapStandaloneSlcAsSlc = true
---
// long singleline comment
val b = 2 // long singleline comment
```

### `docstrings.style`

> Since v2.6.0.

```scala mdoc:defaults
docstrings.style
```

#### `docstrings.style = keep`

Prohibits formatting of docstrings. All other `docstrings` parameters are
ignored.

> Since v3.0.0.

```scala mdoc:scalafmt
docstrings.style = keep
---
/**   do not touch
 * this style
  * keep the text as-is
*/
```

#### `docstrings.style = Asterisk`

This variant used to be called `JavaDoc`.

```scala mdoc:scalafmt
docstrings.style = Asterisk
---
/** Skip first line, format intermediate lines with an asterisk
  * below the first asterisk of the first line (aka JavaDoc)
  */
```

#### `docstrings.style = SpaceAsterisk`

This variant used to be called `ScalaDoc`.

```scala mdoc:scalafmt
docstrings.style = SpaceAsterisk
---
/** Format intermediate lines with a space and an asterisk,
 * both below the two asterisks of the first line
 */
```

#### `docstrings.style = AsteriskSpace`

```scala mdoc:scalafmt
docstrings.style = AsteriskSpace
---
/** Format intermediate lines with an asterisk and a space,
  * both below the two asterisks of the first line
  */
```

### `docstrings.oneline`

> Since v2.6.0. Ignored for `docstrings.style = keep`.

```scala mdoc:defaults
docstrings.oneline
```

#### `docstrings.oneline = fold`

```scala mdoc:scalafmt
docstrings.style = Asterisk
docstrings.oneline = fold
---
/** Scaladoc oneline */
/**
  * Scaladoc multiline
  */
val a = 1
```

#### `docstrings.oneline = unfold`

```scala mdoc:scalafmt
docstrings.style = Asterisk
docstrings.oneline = unfold
---
/** Scaladoc oneline */
/**
  * Scaladoc multiline
  */
val a = 1
```

#### `docstrings.oneline = keep`

```scala mdoc:scalafmt
docstrings.style = Asterisk
docstrings.oneline = keep
---
/** Scaladoc oneline */
/**
  * Scaladoc multiline
  */
val a = 1
```

### `docstrings.wrap`

Will parse scaladoc comments and reformat them.

This functionality is generally limited to
[standard scaladoc elements](https://docs.scala-lang.org/overviews/scaladoc/for-library-authors.html)
and might lead to undesirable results in corner cases; for instance, the
scaladoc parser doesn't have proper support of embedded HTML.

However,
[tables are supported](https://www.scala-lang.org/blog/2018/10/04/scaladoc-tables.html).

> Since v2.6.0. Ignored for `docstrings.style = keep`.

```scala mdoc:defaults
docstrings.wrap
```

```scala mdoc:scalafmt
docstrings.wrap = yes
maxColumn = 30
---
/**
 * @param d the Double to square, meaning multiply by itself
 * @return the result of squaring d
 *
 * Thus
 * - if [[d]] represents a negative value:
 *  a. the result will be positive
 *  a. the value will be {{{d * d}}}
 *  a. it will be the same as for `-d`
 * - however, if [[d]] is positive
 *  - the value will still be {{{d * d}}}
 *    - i.e., the same as {{{(-d) * (-d)}}}
 *
 * In other words:
 * {{{
 *    res = d * d
 *        = (-d) * (-d) }}}
 */
def pow2(d: Double): Double
```

### `docstrings.blankFirstLine`

Controls whether to force the first line to be blank in a multiline docstring.
Keep in mind that some combinations of parameters are prohibited (e.g.,
`blankFirstLine=keep` contradicts with `style=Asterisk`).

> Since v2.7.5. Ignored for `docstrings.style = keep` or
> `docstrings.wrap = no`.

```scala mdoc:defaults
docstrings.blankFirstLine
```

```scala mdoc:scalafmt
# do not force a blank first line
docstrings.blankFirstLine = no
docstrings.style = SpaceAsterisk
maxColumn = 30
---
/** Scaladoc oneline */
/** Scaladoc multiline1
  */
/**
  * Scaladoc multiline2
  */
val a = 1
```

```scala mdoc:scalafmt
# force a blank first line
docstrings.blankFirstLine = yes
docstrings.style = SpaceAsterisk
maxColumn = 30
---
/** Scaladoc oneline */
/** Scaladoc multiline1
  */
/**
  * Scaladoc multiline2
  */
val a = 1
```

```scala mdoc:scalafmt
# preserve a blank first line
docstrings.blankFirstLine = keep
docstrings.style = SpaceAsterisk
maxColumn = 30
---
/** Scaladoc oneline */
/** Scaladoc multiline1
  */
/**
  * Scaladoc multiline2
  */
val a = 1
```

## Disabling or customizing formatting

### For code block

There is a possibility to override scalafmt config for a specific code with
`// scalafmt: {}` comment:

```scala mdoc:scalafmt
---
// scalafmt: { align.preset = most, danglingParentheses.preset = false }
libraryDependencies ++= Seq(
  "org.scalameta" %% "scalameta" % scalametaV,
  "org.scalacheck" %% "scalacheck" % scalacheckV)

// scalafmt: { align.preset = some, danglingParentheses.preset = true } (back to defaults)
libraryDependencies ++= Seq(
  "org.scalameta" %% "scalameta" % scalametaV,
  "org.scalacheck" %% "scalacheck" % scalacheckV)
```

### // format: off

Disable formatting for specific regions of code by wrapping them in
`// format: off` blocks:

```scala mdoc:scalafmt
---
// format: off
val identity = Array(1, 0, 0,
                     0, 1, 0,
                     0, 0, 1)
// format: on
```

### Project

Configure which source files should be formatted in this project.

#### `project.git`

If this boolean flag is set, only format files tracked by git.

#### `project.include/exclude`

> Since v3.0.0.

```scala mdoc:defaults
project.includePaths
project.excludePaths
```

Allows specifying
[PathMatcher](https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileSystem.html#getPathMatcher-java.lang.String-)
selection patterns to identify further which files are to be formatted (explicit `glob:` or
`regex:` prefixes are required; keep in mind that `PathMatcher` patterns must match the entire
path).

For instance,

```conf
project {
  includePaths = [
    "glob:**.scala",
    "regex:.*\\.sc"
  ]
  excludePaths = [
    "glob:**/src/test/scala/**.scala"
  ]
}
```

### `fileOverride`

> Since v2.5.0.

Allows specifying an additional subset of parameters for each file matching a
[PathMatcher](https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileSystem.html#getPathMatcher-java.lang.String-)
pattern. For instance,

```
align.preset = none
fileOverride {
  "glob:**/*.sbt" {
    align.preset = most
  }
  "glob:**/src/test/scala/**/*.scala" {
    maxColumn = 120
    binPack.unsafeCallSite = true
  }
}
```

uses `align.preset=none` for all files except `.sbt` for which
`align.preset=most` will apply. It will also use different parameters for test
suites.

> This parameter does not modify which files are formatted.

## Spaces

### `spaces.beforeContextBoundColon`

```scala mdoc:defaults
spaces.beforeContextBoundColon
```

```scala mdoc:scalafmt
spaces.beforeContextBoundColon=Never
---
def method[A: Bound]: B
def method[A : Bound]: B
def method[A: Bound: Bound2]: B
```

```scala mdoc:scalafmt
spaces.beforeContextBoundColon=Always
---
def method[A: Bound]: B
def method[A : Bound]: B
def method[A: Bound: Bound2]: B
```

```scala mdoc:scalafmt
spaces.beforeContextBoundColon=IfMultipleBounds
---
def method[A: Bound]: B
def method[A : Bound]: B
def method[A: Bound: Bound2]: B
```

### `spaces.inImportCurlyBraces`

```scala mdoc:defaults
spaces.inImportCurlyBraces
```

```scala mdoc:scalafmt
spaces.inImportCurlyBraces=true
---
import a.b.{c, d}
```

### `spaces.inInterpolatedStringCurlyBraces`

> Since v3.0.0.

```scala mdoc:defaults
spaces.inInterpolatedStringCurlyBraces
```

```scala mdoc:scalafmt
spaces.inInterpolatedStringCurlyBraces = true
---
s"Hello ${the} world!"
s"Hello ${ th.e} world!"
s"Hello ${the() } world!"
```

```scala mdoc:scalafmt
spaces.inInterpolatedStringCurlyBraces = false
---
s"Hello ${ oneHundred }% world!"
s"Hello ${ th.e} world!"
s"Hello ${the() } world!"
```

### `spaces.inParentheses`

```scala mdoc:defaults
spaces.inParentheses
```

```scala mdoc:scalafmt
spaces.inParentheses=true
---
foo(a, b)
```

### `spaces.neverAroundInfixTypes`

```scala mdoc:defaults
spaces.neverAroundInfixTypes
```

```scala mdoc:scalafmt
spaces.neverAroundInfixTypes=["##"]
---
def f: Foo##Repr
def g: Foo\/Repr
// usage same operator not as type
def e = a##b
```

### `spaces.afterKeywordBeforeParen`

```scala mdoc:defaults
spaces.afterKeywordBeforeParen
```

```scala mdoc:scalafmt
spaces.afterKeywordBeforeParen = false
---
if (a) println("HELLO!")
while (a) println("HELLO!")
```

### `spaces.inByNameTypes`

```scala mdoc:defaults
spaces.inByNameTypes
```

```scala mdoc:scalafmt
spaces.inByNameTypes = false
---
def foo(a: => A): A
```

### `spaces.afterSymbolicDefs`

```scala mdoc:defaults
spaces.afterSymbolicDefs
```

```scala mdoc:scalafmt
spaces.afterSymbolicDefs=true
---
def +++(a: A): F[A]
```

## Literals

> Since v2.5.0.

Scalafmt allows flexible configuration of
[Integer](https://www.scala-lang.org/files/archive/spec/2.13/01-lexical-syntax.html#integer-literals)
and
[Floating Point](https://www.scala-lang.org/files/archive/spec/2.13/01-lexical-syntax.html#integer-literals)
literals formatting.

Default formatting:

```scala mdoc:scalafmt
---
123l
0XFff
0x1Abl
10E-1
10e-1D
```

Each `literals.*` setting has three available options: `Upper`, `Lower`,
`Unchanged`.

### `literals.long`

```scala mdoc:defaults
literals.long
```

Responsible for the case of `Long` literals suffix `L`

```scala mdoc:scalafmt
literals.long=Upper
---
123l
```

### `literals.float`

```scala mdoc:defaults
literals.float
```

Responsible for the case of `Float` literals suffix `F`

```scala mdoc:scalafmt
literals.float=Lower
---
42.0F
```

### `literals.double`

```scala mdoc:defaults
literals.double
```

Responsible for the case of `Double` literals suffix `D`

```scala mdoc:scalafmt
literals.double=Lower
---
42.0d
```

### `literals.hexPrefix`

```scala mdoc:defaults
literals.hexPrefix
```

Responsible for the case of hex integer literals prefix `0x`

```scala mdoc:scalafmt
literals.hexPrefix=Lower
---
0X123
```

### `literals.hexDigits`

```scala mdoc:defaults
literals.hexDigits
```

Responsible for the case of hex integer literals digits

```scala mdoc:scalafmt
literals.hexDigits=Lower
literals.long=Upper
---
0xaAaA
0xaAaAl
```

### `literals.scientific`

```scala mdoc:defaults
literals.scientific
```

Responsible for the case of `Double` literals exponent part

```scala mdoc:scalafmt
literals.scientific=Upper
literals.float=Lower
---
10e-1
10e-1f
```

## XML

Controls formatting of Scala embedded within XML.

### `xmlLiterals.assumeFormatted`

> Since v2.6.0.

If set, formats embedded Scala relative to containing XML, making the assumption
that XML itself is properly formatted. Otherwise, formatting is relative to the
outer Scala code which contains the XML literals.

```scala mdoc:defaults
xmlLiterals.assumeFormatted
```

```scala mdoc:scalafmt
maxColumn = 40
xmlLiterals.assumeFormatted = true
---
object Example2 {
  def apply() = {
      <foo>
        <bar>{ (1 + 2 + 3).toString("some long format") }</bar>
      </foo>
  }
}
```

```scala mdoc:scalafmt
maxColumn = 40
xmlLiterals.assumeFormatted = false
---
object Example2 {
  def apply() = {
      <foo>
        <bar>{ (1 + 2 + 3).toString("some long format") }</bar>
      </foo>
  }
}
```

## Binpacking

### `binPack.literalArgumentLists`

```scala mdoc:defaults
binPack.literalArgumentLists
```

```scala mdoc:scalafmt
binPack.literalArgumentLists = true
---
val secret: List[Bit] = List(0, 0, 1, 1, 1, 0, 1, 0, 1, 1, 1, 0, 0, 1, 1, 0, 1,
  0, 1, 1, 1, 0, 0, 1, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1)
```

```scala mdoc:scalafmt
binPack.literalArgumentLists = false
---
val secret: List[Bit] = List(0, 0, 1, 1, 1, 1, 1, 0, 0, 1, 1, 0, 1)
```

```scala mdoc:scalafmt
binPack.literalArgumentLists = true
binPack.literalsSingleLine = true
---
val secret: List[Bit] = List(0, 0, 1, 1, 1, 0, 1, 0, 1, 1, 1, 0, 0, 1, 1, 0, 1,
  0, 1, 1, 1, 0, 0, 1, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1)
```

See also:

- `binPack.literalsIncludeSimpleExpr` (to allow a few select methods followed by
  a few nested single-argument apply calls, with literals as arguments); added
  in v2.5.0
- all other `binPack.literalXXX` parameters (see list at the bottom) are
  self-explanatory.

### `binPack.parentConstructors`

Parent constructors are `C` and `D` in `class A extends B with C and D`. Changed
from a boolean to a wider set of options in v2.6.0.

```scala mdoc:defaults
binPack.parentConstructors
```

The behaviour of `binPack.parentConstructors = source` depends on the value of
[`newlines.source`](#newlinessource); `keep` maps to `keep` and attempts to preserve the
space if there's no line break in the source, `fold` maps to `Oneline`, rest to `Never`.

```scala mdoc:scalafmt
binPack.parentConstructors = Always
maxColumn = 30
---
object A {
  trait Foo
  extends Bar
  with Baz
}
```

```scala mdoc:scalafmt
binPack.parentConstructors = Never
maxColumn = 30
---
object A {
  trait Foo extends Bar with Baz
}
```

```scala mdoc:scalafmt
binPack.parentConstructors = Oneline
maxColumn = 30
---
object A {
  class Foo(a: Int)
  extends Bar
  with Baz

  class Foo(
    a: Int
  )
  extends Bar
  with Baz
}
```

```scala mdoc:scalafmt
binPack.parentConstructors = OnelineIfPrimaryOneline
maxColumn = 30
---
object A {
  class Foo(a: Int, b: Int)
  extends Bar
  with Baz

  class Foo(
    a: Int,
    b: Int
  ) extends Bar with Baz
}
```

```scala mdoc:scalafmt
binPack.parentConstructors = keep
---
object A {
  class Foo(a: Int, b: Int) extends Bar(
    a,
    b
  ) with Baz
  with Qux
}
```

## Classic select chains

The parameters below control formatting of select chains when
`newlines.source = classic`, and specifically which select expressions are
included in a chain.

Generally, a chain can either be formatted on one line up to the last select, or
will have a break on the first select.

### `includeCurlyBraceInSelectChains`

Controls if select followed by curly braces can _start_ a chain.

```scala mdoc:defaults
includeCurlyBraceInSelectChains
```

```scala mdoc:scalafmt
includeCurlyBraceInSelectChains = true
---
List(1).map { x =>
    x + 2
  }
  .filter(_ > 2)
```

```scala mdoc:scalafmt
includeCurlyBraceInSelectChains = false
---
List(1)
  .map { x =>
    x + 2
  }
  .filter(_ > 2)
```

### `includeNoParensInSelectChains`

Controls if select _not_ followed by an apply can _start_ a chain.

```scala mdoc:defaults
includeNoParensInSelectChains
```

```scala mdoc:scalafmt
includeNoParensInSelectChains = true
---
List(1).toIterator.buffered
  .map(_ + 2)
  .filter(_ > 2)
```

```scala mdoc:scalafmt
includeNoParensInSelectChains = false
---
List(1).toIterator.buffered.map(_ + 2).filter(_ > 2)
```

### `optIn.breakChainOnFirstMethodDot`

Keeps the break on the first select of the chain if the source contained one.
Has no effect if there was no newline in the source.

```scala mdoc:defaults
optIn.breakChainOnFirstMethodDot
```

```scala mdoc:scalafmt
optIn.breakChainOnFirstMethodDot = false
---
// collapse into a single line
foo
  .map(_ + 1)
  .filter(_ > 2)
```

```scala mdoc:scalafmt
optIn.breakChainOnFirstMethodDot = true
---
// preserve break on first dot and break on subsequent dots
foo
  .map(_ + 1).filter(_ > 2)
```

### `optIn.breaksInsideChains`

Controls whether to preserve a newline before each subsequent select when the
very first one used a line break; that is, this parameter doesn't prohibit
single-line formatting even if there are source breaks down the chain.

If false, each subsequent select within the chain will behave exactly like the
first, that is, either the entire chain will be formatted on one line, or will
contain a break on every select.

If true, preserves existence or lack of breaks on subsequent selects if the
first select was formatted with a newline.

```scala mdoc:defaults
optIn.breaksInsideChains
```

```scala mdoc:scalafmt
optIn.breaksInsideChains = true
maxColumn = 35
---
foo.bar(_ + 1)
  .baz(_ > 2).qux
foo.bar(_ + 1).baz(_ > 2).qux
foo.bar(_ + 1).baz(_ > 2).qux(_ * 12)
foo.bar(_ + 1).baz(_ > 2).qux { _ * 12 }
foo.bar(_ + 1)
  .baz(_ > 2).qux(_ * 12)
```

```scala mdoc:scalafmt
optIn.breaksInsideChains = false
maxColumn = 35
---
foo.bar(_ + 1)
  .baz(_ > 2).qux
foo.bar(_ + 1).baz(_ > 2).qux
foo.bar(_ + 1).baz(_ > 2).qux(_ * 12)
foo.bar(_ + 1).baz(_ > 2).qux { _ * 12 }
foo.bar(_ + 1)
  .baz(_ > 2).qux(_ * 12)
```

### `optIn.encloseClassicChains`

Controls what happens if a chain enclosed in parentheses is followed by
additional selects. Those additional selects will be considered part of the
enclosed chain if and only if this flag is false.

> Since v2.6.2.

```scala mdoc:defaults
optIn.encloseClassicChains
```

```scala mdoc:scalafmt
optIn.encloseClassicChains = true
maxColumn = 30
---
(foo.map(_ + 1).map(_ + 1))
  .filter(_ > 2)
```

```scala mdoc:scalafmt
optIn.encloseClassicChains = false
maxColumn = 30
---
(foo.map(_ + 1).map(_ + 1))
  .filter(_ > 2)
```

## Miscellaneous

### `optIn.forceBlankLineBeforeDocstring`

If true, always insert a blank line before docstrings;  
If false, preserves blank line only if one exists before.

```scala mdoc:defaults
optIn.forceBlankLineBeforeDocstring
```

```scala mdoc:scalafmt
optIn.forceBlankLineBeforeDocstring = true
---
object Stuff {
  /** Some function */
  def hello = ()
}
```

```scala mdoc:scalafmt
optIn.forceBlankLineBeforeDocstring = false
---
object Stuff {
  /** Some function */
  def hello = ()
}
```

### `rewriteTokens`

Map of tokens to rewrite. For example, Map("⇒" -> "=>") will rewrite unicode
arrows to regular ascii arrows.

```scala mdoc:defaults
rewriteTokens
```

```scala mdoc:scalafmt
rewriteTokens = {
  "⇒": "=>"
  "→": "->"
  "←": "<-"
}
---
val tuple = "a" → 1
val lambda = (x: Int) ⇒ x + 1
for {
  a ← Option(1)
  b ← Option(2)
} yield a + b
```

### `importSelectors`

This parameter controls formatting of imports.

```scala mdoc:defaults
importSelectors
```

```scala mdoc:scalafmt
maxColumn = 10
importSelectors = noBinPack
---
import a.b.{c, d, e, f, g}
```

```scala mdoc:scalafmt
maxColumn = 10
importSelectors = binPack
---
import a.b.{c, d, e, f, g}
```

```scala mdoc:scalafmt
maxColumn = 10
importSelectors = singleLine
---
import a.b.{c, d, e, f, g}
```

## Edition

> Removed in 2.7.0

Editions are no longer used. They were kept for backwards compatibility with old
configuration files but new changes to the default Scalafmt formatting behavior
will not respect the `edition` setting.

## Other

To find all available configuration options, it's best to browse the source code
of Scalafmt. A good place to start is `ScalafmtConfig`. Observe that this
listing below is the top-level, there are more configuration options if you
visited nested fields like `spaces` and `newlines`.

> The values for parameters below are purely for informational purposes. They use
> pseudo-formatting similar to but incompatible with `scalafmt.conf`.
> Some of them show values which can't be explicitly specified.

```scala mdoc:defaults:all

```
