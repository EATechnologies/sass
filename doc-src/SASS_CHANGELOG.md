# Sass Changelog

* Table of contents
{:toc}

## 3.3.14 (Unreleased)

* Fix a bug where some `@media` queries would be followed by newlines in
  compressed mode.

## 3.3.14 (1 August 2014)

* Improved edge-case parsing of special-syntax functions like `calc()` and
  `expression()`.

* Fixed a bug when using `--update` with absolute paths on Windows.

## 3.3.13 (31 July 2014)

* Fixed a bug on ruby 2.0 where watching several folders was broken.
  Work around for a [bug in listen](https://github.com/guard/listen/issues/243).

## 3.3.12 (29 July 2014)

* The Sass::Compiler class has a number of new minor features to support
  Compass's compilation needs:

  * The template_deleted event of the Sass Compiler class now runs before the
    side-effect events.
  * The Sass Compiler class can now be used to clean output files.
  * The Sass Compiler class now exposes an updating_stylesheets callback
    that runs before stylesheets are mass-updated.
  * The Sass Compiler class now exposes a the compilation_starting callback
    that runs before an individual stylesheet is compiled.
  * The Sass Compiler class now runs the updated_stylesheets callback
    after stylesheets are mass-updated.
  * The Sass Compiler can now be made to skip the initial update
    when watching.

## 3.3.11 (25 July 2014)

* `str-slice()` now correctly returns an empty string when `$end-at` is 0.

## 3.3.10 (11 July 2014)

* Properly encode URLs in sourcemaps.

## 3.3.9 (27 June 2014)

* Defining a function named "and", "or", or "not" is now an error at
  definition-time. This isn't considered a backwards-incompatible change because
  although these functions could be defined previously, they could never be
  successfully invoked.

* Fix deprecation warnings for using `File.exists?` on recent Ruby versions.

* Fix a bug where `@extend` within `@at-root` could crash the compiler.

* Gracefully handle the inability to change cache files' permissions.

## 3.3.8 (20 May 2014)

* When a use of `@at-root` doesn't add any new rules, it will no longer split
  its containing block in two ([issue 1239][]).

* Fix a `sass-convert` bug where rules would occasionally be folded together
  incorrectly when converting from CSS to Sass.

* Fix error messages for dynamically-generated `@media` queries with empty
  queries.

[issue 1239]: https://github.com/nex3/sass/issues/1239

## 3.3.7 (2 May 2014)

* Properly set the default `Sass::Plugin` options in Rails 3.0.

* Fix a few cases where source ranges were being computed incorrectly for
  SassScript expressions.

## 3.3.6 (25 April 2014)

* The `inspect()` function will only interpret `/` between numbers as division
  under the same circumstances that it would be interpreted as division when
  used in a property.

* Fix several cases where parsing pathological comments would cause Sass to take
  exponential time and consume all available CPU.

## 3.3.5 (14 April 2014)

* Fix `LoadError`s when using `--watch` with the bundled version of Listen.

* Properly parse negative numbers preceded by a comment.

* Avoid unnecessary interpolation when running `sass-convert` on media queries.

* Avoid freezing Ruby's `true` or `false` values.

## 3.3.4 (21 March 2014)

* Improve the warning message for `index(...) == false`.

* Fix the use of directives like `@font-face` within `@at-root`.

* Fix a `sass --watch` issue on Windows where too many files would be updated
  on every change.

* Avoid freezing Ruby's `nil` value.

## 3.3.3 (14 March 2014)

* Fix a bug in Sass that was causing caching errors when unserializable objects
  were in the Ruby options hash. Note that these errors may persist when using
  Sass with Sprockets until the Sprockets importer is made serializable.

## 3.3.2 (11 March 2014)

* Fix a bug with loading the bundled version of Listen.

## 3.3.1 (10 March 2014)

This release includes a number of fixes for issues that popped up in the
immediate aftermath of the 3.3.0 release.

### Re-bundle [listen](http://github.com/guard/listen)

With 3.3.0, we un-bundled the listen library from Sass. We did so hoping that it
would make it easier for users to keep up to date with the latest features and
bug fixes, but unfortunately listen 2.0 and on have dropped support for Ruby
1.8.7, which Sass continues to support. Further complicating things, RubyGems
lacks the ability to install only the version of listen supported by the current
Ruby installation, so we were unable to use a standard Gem dependency on listen.

To work around this, we tried to piggyback on RubyGems' native extension support
to install the correct version of listen when Sass was installed. This is what
we released in 3.3.0. However, this caused numerous problems in practice,
especially for users on Windows. It quickly became clear that this wasn't a
viable long-term solution.

As such, we're going back to the bundling strategy. While not perfect, this
worked well enough for the duration of the Sass 3.2 release, and we expect it to
cause much less havoc than un-bundling. We'll bundle listen 1.3.1, the most
recent version that retains Ruby 1.8.7 compatibility. If a user of Sass has a
more recent version of listen installed, that will be preferred to the bundled
version. Listen versions through 2.7.0 have been tested, and we expect the code
to work without modification on versions up to 3.0.0, assuming no major API
changes.

### Smaller Changes

* Fixed a small interface incompatibility with listen 2.7.0.

* Fix some corner cases of path handling on Windows.

* Avoid errors when trying to watch read-only directories using listen 1.x.

## 3.3.0 (7 March 2014)

### SassScript Maps

SassScript has a new data type: maps. These are associations from SassScript
values (often strings, but potentially any value) to other SassScript values.
They look like this:

    $map: (key1: value1, key2: value2, key3: value3);

Unlike lists, maps must always be surrounded by parentheses. `()` is now an
empty map in addition to an empty list.

Maps will allow users to collect values into named groups and access those
groups dynamically. For example, you could use them to manage themes for your
stylesheet:

    $themes: (
      mist: (
        header: #DCFAC0,
        text:   #00968B,
        border: #85C79C
      ),
      spring: (
        header: #F4FAC7,
        text:   #C2454E,
        border: #FFB158
      ),
      // ...
    );

    @mixin themed-header($theme-name) {
      h1 {
        color: map-get(map-get($themes, $theme-name), header);
      }
    }

There are a variety of functions for working with maps:

* The {Sass::Script::Functions#map_get `map-get($map, $key)` function} returns
  the value in the map associated with the given key. If no value is found, it
  returns `null`.

* The {Sass::Script::Functions#map_merge `map-merge($map1, $map2)` function}
  merges two maps together into a new map. If there are any conflicts, the
  second map takes precedence, making this a good way to modify values in a map
  as well.

* The {Sass::Script::Functions#map_remove `map-remove($map, $key)` function}
  returns a new map with a key removed.

* The {Sass::Script::Functions#map_keys `map-keys($map)` function} returns all
  the keys in a map as a comma-separated list.

* The {Sass::Script::Functions#map_values `map-values($map)` function} returns
  all the values in a map as a comma-separated list.

* The {Sass::Script::Functions#map_has_key `map-has-key($map, $key)` function}
  returns whether or not a map contains a pair with the given key.

All the existing list functions also work on maps, treating them as lists of
pairs. For example, `nth((foo: 1, bar: 2), 1)` returns `foo 1`. Maps can also be
used with `@each`, using the new multiple assignment feature (see below):

    @each $header, $size in (h1: 2em, h2: 1.5em, h3: 1.2em) {
      #{$header} {
        font-size: $size;
      }
    }

Produces:

    h1 {
      font-size: 2em;
    }

    h2 {
      font-size: 1.5em;
    }

    h3 {
      font-size: 1.2em;
    }

#### Variable Keyword Arguments

Maps can be passed as variable arguments, just like lists. For example, if
`$map` is `(alpha: -10%, "blue": 30%)`, you can write `scale-color($color,
$map...)` and it will do the same thing as `scale-color($color, $alpha: -10%,
$blue: 30%)`. To pass a variable argument list and map at the same time, just do
the list first, then the map, as in `fn($list..., $map...)`.

You can also access the keywords passed to a function that accepts a variable
argument list using the new {Sass::Script::Functions#keywords `keywords($args)`
function}. For example:

    @function create-map($args...) {
      @return keywords($args);
    }

    create-map($foo: 10, $bar: 11); // returns (foo: 10, bar: 11)

#### Lists of Pairs as Maps

The new map functions work on lists of pairs as well, for the time being. This
feature exists to help libraries that previously used lists of pairs to simulate
maps. These libraries can now use map functions internally without introducing
backwards-incompatibility. For example:

    $themes: (
      mist (
        header #DCFAC0,
        text   #00968B,
        border #85C79C
      ),
      spring (
        header #F4FAC7,
        text   #C2454E,
        border #FFB158
      ),
      // ...
    );

    @mixin themed-header($theme-name) {
      h1 {
        color: map-get(map-get($themes, $theme-name), header);
      }
    }

Since it's just a migration feature, using lists of pairs in place of maps is
already deprecated. Library authors should encourage their users to use actual
maps instead.

### Source Maps

Sass now has the ability to generate standard JSON [source maps][] of a format
that will soon be supported in most major browsers. These source maps tell the
browser how to find the Sass styles that caused each CSS style to be generated.
They're much more fine-grained than the old Sass-specific debug info that was
generated; rather than providing the source location of entire CSS rules at a
time, source maps provide the source location of each individual selector and
property.

Source maps can be generated by passing the `--sourcemap` flag to the `sass`
executable, by passing the {file:SASS_REFERENCE.md#sourcemap-option `:sourcemap`
option} to \{Sass::Plugin}, or by using the
\{Sass::Engine#render\_with\_sourcemap} method. By default, Sass assumes that
the source stylesheets will be made available on whatever server you're using,
and that their relative location will be the same as it is on the local
filesystem. If this isn't the case, you'll need to make a custom class that
extends \{Sass::Importers::Base} or \{Sass::Importers::Filesystem} and overrides
\{Sass::Importers::Base#public\_url `#public_url`}.

Thanks to Alexander Pavlov for implementing this.

[source maps]: https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit?hl=en_US&pli=1&pli=1

#### `@at-root`

Sass 3.3 adds the `@at-root` directive, which is a way to tell Sass to
put a collection of rules at the top-level root of the document. The
easiest way to use it is with a selector:

    .badge {
      @at-root .info { ... }
      @at-root .header { ... }
    }

In addition to using `@at-root` on a single selector, you can also use it on a
whole block of them. For example:

    .badge {
      @at-root {
        .info { ... }
        .header { ... }
      }
    }

Also produces:

    .info { ... }
    .header { ... }

#### `@at-root (without: ...)` and `@at-root (with: ...)`

By default, `@at-root` just excludes selectors. However, it's also
possible to use `@at-root` to move outside of nested directives such
as `@media` as well. For example:

    @media print {
      .page {
        width: 8in;
        @at-root (without: media) {
          color: red;
        }
      }
    }

produces:

    @media print {
      .page {
        width: 8in;
      }
    }
    .page {
      color: red;
    }

You can use `@at-root (without: ...)` to move outside of any
directive. You can also do it with multiple directives separated by a
space: `@at-root (without: media supports)` moves outside of both
`@media` and `@supports` queries.

There are two special values you can pass to `@at-root`. "rule" refers
to normal CSS rules; `@at-root (without: rule)` is the same as
`@at-root` with no query. `@at-root (without: all)` means that the
styles should be moved outside of *all* directives and CSS rules.

If you want to specify which directives or rules to include, rather
than listing which ones should be excluded, you can use `with` instead
of `without`. For example, `@at-root (with: rule)` will move outside
of all directives, but will preserve any CSS rules.

### Smaller Improvements

* The parent selector, `&`, can be used with an identifier suffix. For
  example, `&-suffix` and `&_suffix` are now legal. The suffix will be
  added to the end of the parent selector, and will throw an error if
  this isn't possible. `&` must still appear at the beginning of a
  compound selector -- that is, `.foo-&` is still illegal.

* [listen](http://github.com/guard/listen) is no longer bundled with
  Sass, nor is it a standard RubyGems dependency. Instead, it's
  automatically installed along with Sass in order to ensure that the
  user ends up with a version of Listen that works with their local
  Ruby version.

* Sass now has numerous functions for working with strings:
  \{Sass::Script::Functions#str_length `str-length`} will return the length of a
  string; \{Sass::Script::Functions#str_insert `str-insert`} will insert one
  string into another; \{Sass::Script::Functions#str_index `str-index`} will
  return the index of a substring within another string;
  \{Sass::Script::Functions#str_slice `str-slice`} will slice a substring from a
  string; \{Sass::Script::Functions#to_upper_case `to-upper-case`} will
  transform a string to upper case characters; and
  \{Sass::Script::Functions#to_lower_case `to-lower-case`} will transform a
  string to lower case characters.

* A \{Sass::Script::Functions#list_separator `list-separator`} function has been
  added to determine what separator a list uses. Thanks to [Sam
  Richard](https://github.com/Snugug).

* Custom Ruby functions can now access the global environment, which
  allows them the same power as Sass-based functions with respect to
  reading and setting variables defined elsewhere in the stylesheet.

* The `set-nth($list, $n, $value)` function lets you construct a new
  list based on `$list`, with the nth element changed to the value
  specified.

* In order to make it easier for mixins to process maps, they may now
  recursively call themselves and one another. It is no longer an
  error to have a mixin `@include` loop.

* Add "grey" and "transparent" as recognized SassScript colors. Thanks to [Rob
  Wierzbowski](https://github.com/robwierzbowski).

* Add a function \{Sass::Script::Functions#unique\_id `unique-id()`} that will
  return a CSS identifier that is unique within the scope of a single CSS file.

* Allow negative indices into lists when using `nth()`.

* You can now detect the presence of a Sass feature using the new function
  `feature-exists($feature-name)`. There are no detectable features in this
  release, this is provided so that subsequent releases can begin to
  use it. Additionally, plugins can now expose their functionality
  through `feature-exists` by calling `Sass.add_feature(feature_name)`. Features
  exposed by plugins must begin with a dash to distinguish them from
  official features.

* It is now possible to determine the existence of different Sass
  constructs using these new functions:

  * `variable-exists($name)` checks if a variable resolves in the
    current scope.
  * `global-variable-exists($name)` checks if a global variable of the
    given name exists.
  * `function-exists($name)` checks if a function exists.
  * `mixin-exists($name)` checks if a mixin exists.

* You can call a function by name by passing the function name to the
  call function. For example, `call(nth, a b c, 2)` returns `b`.

* Comments following selectors in the indented syntax will be correctly
  converted using `sass-convert`.

* `@each` now supports "multiple assignment", which makes it easier to iterate
  over lists of lists. If you write `@each $var1, $var2, $var3 in a b c, d e f,
  g h i`, the elements of the sub-lists will be assigned individually to the
  variables. `$var1`, `$var2`, and `$var3` will be `a`, `b` and `c`; then `d`,
  `e`, and `f`; and then `g`, `h`, and `i`. For more information, see
  {file:SASS_REFERENCE.md#each-multi-assign the `@each` reference}.

* `@for` loops can now go downward as well as upward. For example,
  `@for $var from 5 through 1` will set `$var` to `5`, `4`, `3`, `2`,
  and `1`. Thanks to [Robin Roestenburg](http://twitter.com/robinroest).

* There is a new {Sass::Script::Value::Helpers convenience API} for creating
  Sass values from within ruby extensions.

* The `if()` function now only evaluates the argument corresponding to
  the value of the first argument.

* Comma-separated lists may now have trailing commas (e.g. `1, 2,
  3,`). This also allows you to use a trailing comma to distinguish a
  list with a single element from that element itself -- for example,
  `(1,)` is explicitly a list containing the value `1`.

* All directives that are nested in CSS rules or properties and that
  contain more CSS rules or properties are now bubbled up through
  their parent rules.

* A new `random()` function returns a random number.

* A new function inspect($value) is provided for debugging the current
  sass representation of a value.

* The `@debug` directive now automatically inspects sass objects that are not
  strings.

* Numbers will no longer be emitted in scientific notation.

* `sass-convert` will now correctly handle silent (`//`-style) comments
  contained within loud (`/* */`-style) comments.

* Allow modulo arithmetic for numbers with compatible units. Thanks to
  [Isaac Devine](http://www.devinesystems.co.nz).

* Keyword arguments to mixins and functions that contain hyphens will have the
  hyphens preserved when using `sass-convert`.

### Backwards Incompatibilities -- Must Read!

* Sass will now throw an error when `@extend` is used to extend a selector
  outside the `@media` context of the extending selector. This means the
  following will be an error:

      @media screen {
        .foo { @extend .bar; }
      }
      .bar { color: blue; }

* Sass will now throw an error when an `@extend` that has no effect is used. The
  `!optional` flag may be used to avoid this behavior for a single `@extend`.

* Sass will now throw an error when it encounters a single `@import` statement
  that tries to import more than one file. For example, if you have `@import
  "screen"` and both `screen.scss` and `_screen.scss` exist, a warning will be
  printed.

* `grey` and `transparent` are no longer interpreted as strings; they're now
  interpreted as colors, as per the CSS spec.

* The automatic placement of the current working directory onto the Sass
  load path is now deprecated as this causes unpredictable build
  processes.  If you need the current working directory to be available,
  set `SASS_PATH=.` in your shell's environment.

* `Sass::Compiler.on_updating_stylesheet` has been removed.

* `Sass::Plugin.options=` has been removed.

* `Sass::Script::Number::PRECISION` has been removed.

* The methods in the `Sass::Util` module can no longer be used by
  including it. They must be invoked on the module itself for
  performance reasons.

* Sass values have always been immutable. The ruby object that backs
  each sass value is now "frozen" to prevent accidental modification and
  for performance.

* Many classes in the \{Sass::Script} have been rearranged. All the value
  classes have been moved into \{Sass::Script::Value} (e.g.
  \{Sass::Script::Value::Color}, \{Sass::Script::Value::String}, etc). Their
  base class is now \{Sass::Script::Value::Base} instead of
  `Sass::Script::Literal`. All the parse tree classes have been moved into
  \{Sass::Script::Tree} (e.g. \{Sass::Script::Tree::Node},
  \{Sass::Script::Tree::Operation}, etc).

  The old names will continue to work for the next couple releases, but they
  will be removed eventually. Any code using them should upgrade to the new
  names.

* As part of a migration to cleaner variable semantics, assigning to
  global variables in a local context by default is deprecated. If
  there's a global variable named `$color` and you write `$color:
  blue` within a CSS rule, Sass will now print a warning; in the
  future, it will create a new local variable named `$color`. You may
  now explicitly assign to global variables using the `!global` flag;
  for example, `$color: blue !global` will always assign to the global
  `$color` variable.

* Two numbers separated by a hyphen with no whitespace will now be
  parsed as a subtraction operation rather than two numbers in a list.
  That is, `2px-1px` will parse the same as `2px - 1px` rather than
  `2px -1px`.

* `index()`'s `false` return value when a value isn't found is deprecated. In
  future Sass releases it will be `null` instead, so it should be used in ways
  that are compatible with both `false` and `null`.

* `mix()`'s arguments are now `$color1` and `$color2` rather than
  `$color-1` and `$color-2`, in keeping with other functions.

* `comparable()`'s arguments are now `$number1` and `$number2` rather than
  `$number-1` and `$number-2`, in keeping with other functions.

* `percentage()`, `round()`, `ceil()`, `floor()`, and `abs()` now
  take arguments named '$number' instead of '$value'.

## 3.2.16 (17 March 2014)

* Handle a race condition in the filesystem cache store when a cache
  entry becomes invalidated.

## 3.2.15 (7 March 2014)

* Support `&.foo` when the parent selector has a newline followed by a comma.

## 3.2.14 (24 January 2014)

* Don't crash when parsing a directive with no name in the indented syntax.

* Clean up file paths when importing to avoid errors for overlong path
  names.

* Parse calls to functions named `true`, `false`, and `null` as function calls.

* Don't move CSS `@import`s to the top of the file unless it's necessary.

## 3.2.13 (19 December 2013)

* Numbers returned by user-defined functions now trigger division, just like
  numbers stored in variables.

* Support importing files in paths with open brackets.

* Fix `sass-convert`'s handling of rules with empty bodies when converting from
  CSS.

* Fix CSS imports using `url()` with a quoted string and media queries.

## 3.2.12 (4 October 2013)

* Add a couple missing `require`s, fixing some load errors, especially when
  using the command-line interface.

* Tune up some heuristics for eliminating redundant generated selectors. This
  will prevent some selector elimination in cases where multi-layered `@extend`
  is being used and where it seems intuitively like selectors shouldn't be
  eliminated.

## 3.2.11 (27 September 2013)

* Fix `@extend`'s semantics with respect to pseudo-elements. They are no longer
  treated identically to pseudo-classes.

* A more understandable error is now provided when the `-E` option is
  passed to the Sass command line in ruby 1.8

* Fixed a bug in the output of lists containing unary plus or minus
  operations during sass <=> scss conversion.

* Avoid the [IE7 `content: counter` bug][cc bug] with `content: counters` as
  well.

* Fix some thread-safety issues.

## 3.2.10 (26 July 2013)

* Use the Sass logger infrastructure for `@debug` directives.

* When printing a Sass error into a CSS comment, escape `*/` so the comment
  doesn't end prematurely.

* Preserve the `!` in `/*! ... */`-style comments.

* Fix a bug where selectors were being incorrectly trimmed when using `@extend`.

* Fix a bug where `sass --unix-newlines` and `sass-convert --in-place` are not
  working on Windows (thanks [SATO Kentaro](http://www.ranvis.com)).

## 3.2.9 (10 May 2013)

* Fix a bug where `@extend`s would occasionally cause a selector to be generated
  with the incorrect specificity.

* Avoid loading [listen](http://github.com/guard/listen) v1.0, even if it's
  installed as a Gem (see [issue 719](https://github.com/nex3/sass/issues/719)).

* Update the bundled version of [listen](http://github.com/guard/listen) to
  0.7.3.

* Automatically avoid the [IE7 `content: counter` bug][cc bug].

[cc bug]: http://jes.st/2013/ie7s-css-breaking-content-counter-bug/

## 3.2.8 (22 April 2013)

* Fix some edge cases where redundant selectors were emitted when using
  `@extend`.

* Fix a bug where comma-separated lists with interpolation could lose elements.

* Fix a bug in `sass-convert` where lists being passed as arguments to functions
  or mixins would lose their surrounding parentheses.

* Fix a bug in `sass-convert` where `null` wasn't being converted correctly.

* Fix a bug where multiple spaces in a string literal would sometimes be folded
  together.

* `sass` and `sass-convert` won't create an empty file before writing to it.
  This fixes a flash of unstyled content when using LiveReload and similar
  tools.

* Fix a case where a corrupted cache could produce fatal errors on some versions
  of Ruby.

* Fix a case where a mixin loop error would be incorrectly reported when using
  `@content`.

## 3.2.7 (8 March 2013)

* The \{Sass::Script::Functions#index `index`} and \{Sass::Script::Functions#zip
  `zip`} functions now work like all other list functions and treat individual
  values as single-element lists.

* Avoid stack overflow errors caused by very long function or mixin argument
  lists.

* Emit relative paths when using the `--line-comments` flag of the `sass`
  executable.

* Fix a case where very long numbers would cause the SCSS parser to
  take exponential time.

## 3.2.6 (22 February 2013)

* Support for Rubinius 2.0.0.rc1. All tests pass in 1.8 mode. 1.9 mode has some
  tests blocked on [Rubinius issue
  2139](https://github.com/rubinius/rubinius/issues/2139).

* Support for JRuby 1.7.2.

* Support for symlinked executables. Thanks to [Yin-So
  Chen](http://yinsochen.com/).

* Support for bubbling `@supports` queries in the indented syntax.

* Fix an incorrect warning when using `@extend` from within nested `@media`
  queries.

* Update the bundled version of [listen](http://github.com/guard/listen) to
  0.7.2.

## 3.2.5 (4 January 2013)

* Fix a bug where bogus `@extend` warnings were being generated.

* Fix an `@import` bug on Windows. Thanks to [Darryl
  Miles](https://github.com/dlmiles).

* Ruby 2.0.0-preview compatibility. Thanks to [Eric
  Saxby](http://www.livinginthepast.org/).

* Fix incorrect line numbering when using DOS line endings with the indented
  syntax.

## 3.2.4 (21 December 2012)

* Fix imports from `.jar` files in JRuby. Thanks to [Alex
  Hvostov](https://github.com/argv-minus-one).

* Allow comments within `@import` statements in SCSS.

* Fix a parsing performance bug where long decimals would occasionally take many
  minutes to parse.

## 3.2.3 (9 November 2012)

* `sass --watch` no longer crashs when a file in a watched directory is deleted.

* Allow `@extend` within bubbling nodes such as `@media`.

* Fix various JRuby incompatibilities and test failures.

* Work around a performance bug that arises from using `@extend` with
  deeply-nested selectors.

## 3.2.2 (2 November 2012)

* Add a `--poll` option to force `sass --watch` to use the polling backend to
  [Listen](https://github.com/guard/listen).

* Fix some error reporting bugs related to `@import`.

* Treat [protocol-relative URLs][pru] in `@import`s as static URLs, just like
  `http` and `https` URLs.

* Improve the error message for misplaced simple selectors.

* Fix an option-handling bug that was causing errors with the Compass URL
  helpers.

* Fix a performance issue with `@import` that only appears when
  ActiveSupport is loaded.

* Fix flushing of actions to stdout. Thanks to
  [Russell Davis](http://github.com/russelldavis).

* Fix the documentation for the `max()` function.

* Fix a `@media` parsing bug.

[pru]: http://paulirish.com/2010/the-protocol-relative-url/

### Deprecations -- Must Read!

* Sass will now print a warning when it encounters a single `@import` statement
  that tries to import more than one file. For example, if you have `@import
  "screen"` and both `screen.scss` and `_screen.scss` exist, a warning will be
  printed. This will become an error in future versions of Sass.

## 3.2.1 (15 August 2012)

* Fix a buggy interaction with Pow and Capybara that caused `EOFError`s.

## 3.2.0 (10 August 2012)

### `@content`

A mixin include can now accept a block of content ({file:SASS_REFERENCE.md#mixin-content Reference Documentation}).
The style block will be passed to the mixin and can be placed at the point @content is used. E.g.:

    @mixin iphone {
      @media only screen and (max-width: 480px) {
        @content;
      }
    }

    @include iphone {
      body { color: red }
    }

Or in `.sass` syntax:

    =iphone
      @media only screen and (max-width: 480px)
        @content

    +iphone
      body
        color: red

Produces:

    @media only screen and (max-width: 480px) {
      body { color: red }
    }

Note that the contents passed to the mixin are evaluated in the scope they are used,
not the scope of the mixin. {file:SASS_REFERENCE.md#variable_scope_and_content_blocks More on variable scoping.}

### Placeholder Selectors: `%foo`

Sass supports a new, special type of selector called a "placeholder selector".
These look like class and id selectors, except the `#` or `.` is replaced by `%`.
They're meant to be used with the {file:SASS_REFERENCE.md#extend `@extend` directive},
when you want to write styles to be extended
but you don't want the base styles to appear in the CSS.

On its own, a placeholder selector just causes a ruleset not to be rendered.
For example:

    // This ruleset won't be rendered on its own.
    #context a%extreme {
      color: blue;
      font-weight: bold;
      font-size: 2em;
    }

However, placeholder selectors can be extended, just like classes and ids.
The extended selectors will be generated, but the base placeholder selector will not.
For example:

    .notice { @extend %extreme; }

Is compiled to:

    #context a.notice {
      color: blue;
      font-weight: bold;
      font-size: 2em;
    }

### Variable Arguments

Mixins and functions now both support variable arguments. When defining a mixin
or function, you can add `...` after the final argument to have it accept an
unbounded number of arguments and package them into a list. When calling a mixin
or function, you can add `...` to expand the final argument (if it's a list) so
that each value is passed as a separate argument. For example:

    @mixin box-shadow($shadows...) {
      // $shadows is a list of all arguments passed to box-shadow
      -moz-box-shadow: $shadows;
      -webkit-box-shadow: $shadows;
      box-shadow: $shadows;      
    }

    // This is the same as "@include spacing(1, 2, 3);"
    $values: 1, 2, 3;
    @include spacing($values...);

Finally, if a variable argument list is passed directly on to another mixin or
function, it will also pass along any keyword arguments. This means that you can
wrap a pre-existing mixin or function and add new functionality without changing
the call signature.

### Directive Interpolation

`#{}` interpolation is now allowed in all plain CSS directives
(such as `@font-face`, `@keyframes`, and of course `@media`).

In addition, `@media` gets some special treatment.
In addition to allowing `#{}` interpolation,
expressions may be used directly in media feature queries.
This means that you can write e.g.:

    $media: screen;
    $feature: -webkit-min-device-pixel-ratio;
    $value: 1.5;

    @media #{$media} and ($feature: $value) {
      ...
    }

This is intended to allow authors to easily write mixins
that make use of `@media` and other directives dynamically.

### Smaller Improvements

* Mixins and functions may now be defined in a nested context, for example
  within `@media` rules. This also allows files containing them to be imported
  in such contexts.

* Previously, only the `:-moz-any` selector was supported; this has been
  expanded to support any vendor prefix, as well as the plain `:any` selector.

* All proposed [CSS4 selectors](http://dev.w3.org/csswg/selectors4/) are now
  supported, including reference selectors (e.g. `.foo /attr/ .bar`) and subject
  selectors (e.g. `.foo!`).

* Sass now supports a global list of load paths, accessible via
  {Sass.load_paths}. This allows plugins and libraries to easily register their
  Sass files such that they're accessible to all {Sass::Engine} instances.

* `Sass.load_paths` is initialized to the value of the `SASS_PATH` environment
  variable. This variable should contain a colon-separated list of load paths
  (semicolon-separated on Windows).

* In certain cases, redundant selectors used to be created as a result of a
  single rule having multiple `@extend`s. That redundancy has been eliminated.

* Redundant selectors were also sometimes created by nested selectors
  using `@extend`. That redundancy has been eliminated as well.

* There is now much more comprehensive support for using `@extend` alongside
  CSS3 selector combinators (`+`, `~`, and `>`). These combinators will now be
  merged as much as possible.

* The full set of [extended color keywords](http://www.w3.org/TR/css3-color/#svg-color)
  are now supported by Sass. They may be used to refer to color objects, and
  colors will render using those color names when appropriate.

* Sass 3.2 adds the \{Sass::Script::Functions#ie_hex_str `ie-hex-str`} function
  which returns a hex string for a color suitable for use with IE filters.

* Sass 3.2 adds the \{Sass::Script::Functions#min `min`} and
  \{Sass::Script::Functions#max `max`} functions, which return the minimum and
  maximum of several values.

* Sass functions are now more strict about how keyword arguments can be passed.

* Decimal numbers now default to five digits of precision after the decimal
  point.

* The \{Sass::Script::Functions::EvaluationContext#options options hash}
  available to Sass functions now contains the filename of the file that the
  function was executed in, rather than the top-level file.

### Backwards Incompatibilities -- Must Read!

#### `@extend` Warnings

Any `@extend` that doesn't match any selectors in the document will now print a
warning. These warnings will become errors in future versions of Sass. This will
help protect against typos and make it clearer why broken styles aren't working.
For example:

    h1.notice {color: red}
    a.important {@extend .notice}

This will print a warning, since the only use of `.notice` can't be merged with
`a`.

You can declare that you don't want warnings for a specific `@extend` by using
the `!optional` flag. For example:

    h1.notice {color: red}
    a.important {@extend .notice !optional}

This will not print a warning.

#### Smaller Incompatibilities

* Parent selectors followed immediately by identifiers (e.g. `&foo`)
  are fully disallowed.
  They were deprecated in 3.1.8.

* `#{}` interpolation is now allowed in all comments.

* The `!` flag may not be used with `//` comments (e.g. `//!`).

* `#{}` interpolation is now disallowed in all `@import` statements
  except for those using `url()`.

* `sass-convert` no longer supports converting files from LessCSS.

## 3.1.21 (10 August 2012)

* Preserve single-line comments that are embedded within multi-line comments.
* Preserve newlines in nested selectors when those selectors are used multiple
  times in the same document.
* Allow tests to be run without the `LANG` environment variable set.
* Update the bundled version of [Listen](https://github.com/guard/listen) to
  0.4.7.
* Sass will now convert `px` to other absolute units using the
  conversion ratio of `96px == 1in` as dictated by the
  [CSS Spec](http://www.w3.org/TR/CSS21/syndata.html#length-units)

## 3.1.20

* Don't crash if a UTF encoding isn't found. Thanks to [Andrew
  Garbutt](http://github.com/techsplicer).
* Properly watch files recursively with `sass --watch`. Thanks to [Sébastien
  Tisserant](https://github.com/sebweaver).
* Fix the documentation for the \{Sass::Script::Functions#append append()}
  function.
* Support the `saturate()`, `opacity()`, and `invert()` functions when used as
  in the [Filter Effects][filter] spec.
* Support MacRuby. Thanks to [Will Glynn](http://github.com/delta407).

[filter]: https://dvcs.w3.org/hg/FXTF/raw-file/tip/filters/index.html

## 3.1.19

* Fix an `uninitialized constant Sass::Exec::Sass::Util` error when using the
  command-line tool.
* Allow `@extend` within directives such as `@media` as long as it only extends
  selectors that are within the same directive.

## 3.1.18

* Ruby 2.0 compatibility. Thanks to [Jeremy
  Kemper](https://github.com/jeremy).

### Deprecations -- Must Read!

* Deprecate the use of `@extend` within directives such as `@media`. This has
  never worked correctly, and now it's officially deprecated. It will be an
  error in 3.2.

## 3.1.17

* Don't crash when calling `#inspect` on an internal Sass tree object in Ruby
  1.9.
* Fix some bugs in `sass --watch` introduced in 3.1.16. Thanks to [Maher
  Sallam](https://github.com/Maher4Ever).
* Support bare interpolation in the value portion of attribute
  selectors (e.g. `[name=#{$value}]`).
* Support keyword arguments for the `invert()` function.
* Handle backslash-separated paths better on Windows.
* Fix `rake install` on Ruby 1.9.
* Properly convert nested `@if` statements with `sass-convert`.

## 3.1.16

* Fix some bugs in `sass-convert` selector parsing when converting from CSS.
* Substantially improve compilation performance on Ruby 1.8.
* Support the `@-moz-document` directive's non-standard `url-prefix` and
  `domain` function syntax.
* Support the [`@supports` directive](http://www.w3.org/TR/css3-conditional/#at-supports).
* Fix a performance issue when using `/*! */` comments with the Rails asset
  pipeline.
* Support `-moz-element`.
* Properly handle empty lists in `sass-convert`.
* Move from [FSSM](https://github.com/ttilley/fssm) to
  [Listen](https://github.com/guard/listen) for file-system monitoring.

## 3.1.15

* Support extending multiple comma-separated selectors (e.g. `@extend .foo, .bar`).
  This is just a terser way to write multiple `@extend`s
  (e.g. `@extend .foo; @extend .bar`).
  This wasn't previously intended to work, but it did in the indented syntax only.
* Avoid more stack overflows when there are import loops in files.
* Update the bundled [FSSM](https://github.com/ttilley/fssm) to version 0.2.8.1.
* Make the `grayscale` function work with `-webkit-filter`.
* Provide a better error message for selectors beginning with `/`
  in the indented syntax.
* Flush standard output after printing notifications in `sass --watch`.
* Fix variable definitions in the REPL.

## 3.1.14

* Fix a typo that was causing crashes on Ruby 1.9.

## 3.1.13

* Fix a smattering of subtle bugs that would crop up when using multibyte
  character sets.
* Fix a bug when using `@extend` with selectors containing newlines.
* Make boolean operators short-circuit.
* Remove unnecessary whitespace in selectors in `:compressed` mode.
* Don't output debug info within non-`@media` directives.
* Make sure `:after` and `:before` selectors end up on the end of
  selectors resulting from `@extend`.
* Fix a bug when using imports containing invalid path characters on Windows.
* Bubble CSS `@import` statements to the top of stylesheets.

## 3.1.12

* Compatibility with the `mathn` library
  (thanks to [Thomas Walpole](https://github.com/twalpole)).
* Fix some infinite loops with mixins that were previously uncaught.
* Catch infinite `@import` loops.
* Fix a deprecation warning in `sass --update` and `--watch`
  (thanks to [Marcel Köppen](https://github.com/Marzelpan)).
* Don't make `$important` a special pre-initialized variable.
* Fix exponential parsing time of certain complex property values and selectors.
* Properly merge `@media` directives with comma-separated queries.
  E.g. `@media foo, bar { @media baz { ... } }` now becomes
  `@media foo and baz, bar and baz { ... }`.

## 3.1.11

* Allow control directives (such as `@if`) to be nested beneath properties.
* Allow property names to begin with a hyphen followed by interpolation (e.g. `-#{...}`).
* Fix a parsing error with interpolation in comma-separated lists.
* Make `--cache-store` with with `--update`.
* Properly report `ArgumentError`s that occur within user-defined functions.
* Don't crash on JRuby if the underlying Java doesn't support every Unicode encoding.
* Add new `updated_stylesheet` callback, which is run after each stylesheet has
  been successfully compiled. Thanks to [Christian Peters](https://github.com/ChristianPeters).
* Allow absolute paths to be used in an importer with a different root.
* Don't destructively modify the options when running `Sass::Plugin.force_update`.
* Prevent Regexp buffer overflows when parsing long strings
  (thanks to [Agworld](https://github.com/Agworld).

### Deprecations -- Must Read!

* The `updating_stylesheet` is deprecated and will be removed in a
  future release. Use the new `updated_stylesheet` callback instead.

## 3.1.10

* Fix another aspect of the 3.1.8 regression relating to `+`.

## 3.1.9

* Fix a regression in 3.1.8 that broke the `+` combinator in selectors.

* Deprecate the loud-comment flag when used with silent comments (e.g. `//!`).
  Using it with multi-line comments (e.g. `/*!`) still works.

## 3.1.8

* Deprecate parent selectors followed immediately by identifiers (e.g. `&foo`).
  This should never have worked, since it violates the rule
  of `&` only being usable where an element selector would.

* Add a `--force` option to the `sass` executable which makes `--update`
  always compile all stylesheets, even if the CSS is newer.

* Disallow semicolons at the end of `@import` directives in the indented syntax.

* Don't error out when being used as a library without requiring `fileutil`.

* Don't crash when Compass-style sprite imports are used with `StalenessChecker`
  (thanks to [Matthias Bauer](https://github.com/moeffju)).

* The numeric precision of numbers in Sass can now be set using the
  `--precision` option to the command line. Additionally, the default
  number of digits of precision in Sass output can now be
  changed by setting `Sass::Script::Number.precision` to an integer
  (defaults to 3). Since this value can now be changed, the `PRECISION`
  constant in `Sass::Script::Number` has been deprecated. In the unlikely
  event that you were using it in your code, you should now use
   `Sass::Script::Number.precision_factor` instead.

* Don't crash when running `sass-convert` with selectors with two commas in a row.

* Explicitly require Ruby >= 1.8.7 (thanks [Eric Mason](https://github.com/ericmason)).

* Properly validate the nesting of elements in imported stylesheets.

* Properly compile files in parent directories with `--watch` and `--update`.

* Properly null out options in mixin definitions before caching them. This fixes
  a caching bug that has been plaguing some Rails 3.1 users.

## 3.1.7

* Don't crash when doing certain operations with `@function`s.

## 3.1.6

* The option `:trace_selectors` can now be used to emit a full trace
  before each selector. This can be helpful for in-browser debugging of
  stylesheet imports and mixin includes. This option supersedes the
  `:line_comments` option and is superseded by the `:debug_info`
  option.

* Fix a bug where long `@if`/`@else` chains would cause exponential slowdown
  under some circumstances.

## 3.1.5

* Updated the vendored FSSM version, which will avoid segfaults on OS
  X Lion when using `--watch`.

## 3.1.4

* Sass no longer unnecessarily caches the sass options hash.
  This allows objects that cannot be marshaled to be placed into the
  options hash.

## 3.1.3

* Sass now logs message thru a logger object which can be changed to
  provide integration with other frameworks' logging infrastructure.


## 3.1.2

* Fix some issues that were breaking Sass when running within Rubinius.
* Fix some issues that were affecting Rails 3.1 integration.
* New function `zip` allows several lists to be combined into one
  list of lists. For example:
  `zip(1px 1px 3px, solid dashed solid, red green blue)` becomes
  `1px solid red, 1px dashed green, 3px solid blue`
* New function `index` returns the list index of a value
  within a list. For example: `index(1px solid red, solid)`
  returns `2`. When the value is not found `false` is returned.

## 3.1.1

* Make sure `Sass::Plugin` is loaded at the correct time in Rails 3.

## 3.1.0

* Add an {Sass::Script::Functions#invert `invert` function} that takes the inverse of colors.

* A new sass function called `if` can be used to emit one of two values
  based on the truth value of the first argument.
  For example, `if(true, 1px, 2px)` returns `1px` and `if(false, 1px, 2px)` returns `2px`.

* Compass users can now use the `--compass` flag
  to make the Compass libraries available for import.
  This will also load the Compass project configuration
  if run from the project root.

* Many performance optimizations have been made by [thedarkone](http://github.com/thedarkone).

* Allow selectors to contain extra commas to make them easier to modify.
  Extra commas will be removed when the selectors are converted to CSS.

* `@import` may now be used within CSS or `@media` rules.
  The imported file will be treated as though it were nested within the rule.
  Files with mixins may not be imported in nested contexts.

* If a comment starts with `!`, that comment will now be interpolated
  (`#{...}` will be replaced with the resulting value of the expression
  inside) and the comment will always be printed out in the generated CSS
  file -- even with compressed output. This is useful for adding copyright
  notices to your stylesheets.

* A new executable named `scss` is now available. It is exactly like the
  `sass` executable except it defaults to assuming input is in the SCSS syntax.
  Both programs will use the source file's extension to determine the syntax where
  possible.

### Sass-based Functions

While it has always been possible to add functions to Sass with Ruby, this release adds the ability to define new functions within Sass files directly.
For example:

    $grid-width: 40px;
    $gutter-width: 10px;

    @function grid-width($n) {
      @return $n * $grid-width + ($n - 1) * $gutter-width;
    }

    #sidebar { width: grid-width(5); }

Becomes:

    #sidebar {
      width: 240px; }

### Keyword Arguments

Both mixins and Sass functions now support the ability to pass in keyword arguments.
For example, with mixins:

    @mixin border-radius($value, $moz: true, $webkit: true, $css3: true) {
      @if $moz { -moz-border-radius: $value }
      @if $webkit { -webkit-border-radius: $value }
      @if $css3 { border-radius: $value }
    }

    @include border-radius(10px, $webkit: false);

And with functions:

    p {
      color: hsl($hue: 180, $saturation: 78%, $lightness: 57%);
    }

Keyword arguments are of the form `$name: value` and come after normal arguments.
They can be used for either optional or required arguments.
For mixins, the names are the same as the argument names for the mixins.
For functions, the names are defined along with the functions.
The argument names for the built-in functions are listed
{Sass::Script::Functions in the function documentation}.

Sass functions defined in Ruby can use the {Sass::Script::Functions.declare} method
to declare the names of the arguments they take.

#### New Keyword Functions

The new keyword argument functionality enables new Sass color functions
that use keywords to encompass a large amount of functionality in one function.

* The {Sass::Script::Functions#adjust_color adjust-color} function works like the old
  `lighten`, `saturate`, and `adjust-hue` methods.
  It increases and/or decreases the values of a color's properties by fixed amounts.
  For example, `adjust-color($color, $lightness: 10%)` is the same as `lighten($color, 10%)`:
  it returns `$color` with its lightness increased by 10%.

* The {Sass::Script::Functions#scale_color scale_color} function
  is similar to {Sass::Script::Functions#adjust_color adjust_color},
  but instead of increasing and/or decreasing a color's properties by fixed amounts,
  it scales them fluidly by percentages.
  The closer the percentage is to 100% (or -100%),
  the closer the new property value will be to its maximum (or minimum).
  For example, `scale-color(hsl(120, 70, 80), $lightness: 50%)`
  will change the lightness from 80% to 90%,
  because 90% is halfway between 80% and 100%.
  Similarly, `scale-color(hsl(120, 70, 50), $lightness: 50%)`
  will change the lightness from 50% to 75%.

* The {Sass::Script::Functions#change_color change-color} function simply changes a color's properties
  regardless of their old values.
  For example `change-color($color, $lightness: 10%)` returns `$color` with 10% lightness,
  and `change-color($color, $alpha: 0.7)` returns color with opacity 0.7.

Each keyword function accepts `$hue`, `$saturation`, `$value`,
`$red`, `$green`, `$blue`, and `$alpha` keywords,
with the exception of `scale-color()` which doesn't accept `$hue`.
These keywords modify the respective properties of the given color.

Each keyword function can modify multiple properties at once.
For example, `adjust-color($color, $lightness: 15%, $saturation: -10%)`
both lightens and desaturates `$color`.
HSL properties cannot be modified at the same time as RGB properties, though.

### Lists

Lists are now a first-class data type in Sass,
alongside strings, numbers, colors, and booleans.
They can be assigned to variables, passed to mixins,
and used in CSS declarations.
Just like the other data types (except booleans),
Sass lists look just like their CSS counterparts.
They can be separated either by spaces (e.g. `1px 2px 0 10px`)
or by commas (e.g. `Helvetica, Arial, sans-serif`).
In addition, individual values count as single-item lists.

Lists won't behave any differently in Sass 3.1 than they did in 3.0.
However, you can now do more with them using the new [list functions](Sass/Script/Functions.html#list-functions):

* The {Sass::Script::Functions#nth `nth($list, $n)` function} returns the nth item in a list.
  For example, `nth(1px 2px 10px, 2)` returns the second item, `2px`.
  Note that lists in Sass start at 1, not at 0 like they do in some other languages.

* The {Sass::Script::Functions#join `join($list1, $list2)` function}
  joins together two lists into one.
  For example, `join(1px 2px, 10px 5px)` returns `1px 2px 10px 5px`.

* The {Sass::Script::Functions#append `append($list, $val)` function}
  appends values to the end of a list.
  For example, `append(1px 2px, 10px)` returns `1px 2px 10px`.

* The {Sass::Script::Functions#join `length($list)` function}
  returns the length of a list.
  For example, `length(1px 2px 10px 5px)` returns `4`.

For more details about lists see {file:SASS_REFERENCE.md#lists the reference}.

#### `@each`

There's also a new directive that makes use of lists.
The {file:SASS_REFERENCE.md#each-directive `@each` directive} assigns a variable to each item in a list in turn,
like `@for` does for numbers.
This is useful for writing a bunch of similar styles
without having to go to the trouble of creating a mixin.
For example:

    @each $animal in puma, sea-slug, egret, salamander {
      .#{$animal}-icon {
        background-image: url('/images/#{$animal}.png');
      }
    }

is compiled to:

    .puma-icon {
      background-image: url('/images/puma.png'); }
    .sea-slug-icon {
      background-image: url('/images/sea-slug.png'); }
    .egret-icon {
      background-image: url('/images/egret.png'); }
    .salamander-icon {
      background-image: url('/images/salamander.png'); }

### `@media` Bubbling

Modern stylesheets often use `@media` rules to target styles
at certain sorts of devices, screen resolutions, or even orientations.
They're also useful for print and aural styling.
Unfortunately, it's annoying and repetitive to break the flow of a stylesheet
and add a `@media` rule containing selectors you've already written
just to tweak the style a little.

Thus, Sass 3.1 now allows you to nest `@media` rules within selectors.
It will automatically bubble them up to the top level,
putting all the selectors on the way inside the rule.
For example:

    .sidebar {
      width: 300px;
      @media screen and (orientation: landscape) {
        width: 500px;
      }
    }

is compiled to:

    .sidebar {
      width: 300px;
    }
    @media screen and (orientation: landscape) {
      .sidebar {
        width: 500px;
      }
    }

You can also nest `@media` directives within one another.
The queries will then be combined using the `and` operator.
For example:

    @media screen {
      .sidebar {
        @media (orientation: landscape) {
          width: 500px;
        }
      }
    }

is compiled to:

    @media screen and (orientation: landscape) {
      .sidebar {
        width: 500px;
      }
    }

### Nested `@import`

The `@import` statement can now be nested within other structures
such as CSS rules and `@media` rules. For example:

    @media print {
      @import "print";
    }

This imports `print.scss` and places all rules so imported within the `@media print` block.
This makes it easier to create stylesheets for specific media or sections of the document
and distributing those stylesheets across multiple files.

### Backwards Incompatibilities -- Must Read!

* When `@import` is given a path without `.sass`, `.scss`, or `.css` extension,
  and no file exists at that path, it will now throw an error.
  The old behavior of becoming a plain-CSS `@import` was deprecated
  and has now been removed.

* Get rid of the `--rails` flag for the `sass` executable.
  This flag hasn't been necessary since Rails 2.0.
  Existing Rails 2.0 installations will continue to work.

* Removed deprecated support for ! prefixed variables. Use $ to prefix variables now.

* Removed the deprecated css2sass executable. Use sass-convert now.

* Removed support for the equals operator in variable assignment. Use : now.

* Removed the sass2 mode from sass-convert. Users who have to migrate from sass2
  should install Sass 3.0 and quiet all deprecation warnings before installing Sass 3.1.

### Sass Internals

* It is now possible to define a custom importer that can be used to find imports using different import semantics than the default filesystem importer that Sass provides. For instance, you can use this to generate imports on the fly, look them up from a database, or implement different file naming conventions. See the {Sass::Importers::Base Importer Base class} for more information.

* It is now possible to define a custom cache store to allow for efficient caching of Sass files using alternative cache stores like memcached in environments where a writable filesystem is not available or where the cache need to be shared across many servers for dynamically generated stylesheet environments. See the {Sass::CacheStores::Base CacheStore Base class} for more information.

## 3.0.26 (Unreleased)

* Fix a performance bug in large SCSS stylesheets with many nested selectors.
  This should dramatically decrease compilation time of such stylesheets.

* Upgrade the bundled FSSM to version 0.2.3.
  This means `sass --watch` will work out of the box with Rubinius.

## 3.0.25

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.25).

* When displaying a Sass error in an imported stylesheet,
  use the imported stylesheet's contents rather than the top-level stylesheet.

* Fix a bug that caused some lines with non-ASCII characters to be ignored in Ruby 1.8.

* Fix a bug where boolean operators (`and`, `or`, and `not`) wouldn't work at the end of a line
  in a multiline SassScript expression.

* When using `sass --update`, only update individual files when they've changed.

## 3.0.24

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.24).

* Raise an error when `@else` appears without an `@if` in SCSS.

* Fix some cases where `@if` rules were causing the line numbers in error reports
  to become incorrect.

## 3.0.23

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.23).

* Fix the error message for unloadable modules when running the executables under Ruby 1.9.2.

### `@charset` Change

The behavior of `@charset` has changed in version 3.0.23
in order to work around a bug in Safari,
where `@charset` declarations placed anywhere other than the beginning of the document
cause some CSS rules to be ignored.
This change also makes `@charset`s in imported files behave in a more useful way.

#### Ruby 1.9

When using Ruby 1.9, which keeps track of the character encoding of the Sass document internally,
`@charset` directive in the Sass stylesheet and any stylesheets it imports
are no longer directly output to the generated CSS.
They're still used for determining the encoding of the input and output stylesheets,
but they aren't rendered in the same way other directives are.

Instead, Sass adds a single `@charset` directive at the beginning of the output stylesheet
if necessary, whether or not the input stylesheet had a `@charset` directive.
It will add this directive if and only if the output stylesheet contains non-ASCII characters.
By default, the declared charset will be UTF-8,
but if the Sass stylesheet declares a different charset then that will be used instead if possible.

One important consequence of this scheme is that it's possible for a Sass file
to import partials with different encodings (e.g. one encoded as UTF-8 and one as IBM866).
The output will then be UTF-8, unless the importing stylesheet
declares a different charset.

#### Ruby 1.8

Ruby 1.8 doesn't have good support for encodings, so it uses a simpler but less accurate
scheme for figuring out what `@charset` declaration to use for the output stylesheet.
It just takes the first `@charset` declaration to appear in the stylesheet
or any of its imports and moves it to the beginning of the document.
This means that under Ruby 1.8 it's *not* safe to import files with different encodings.

## 3.0.22

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.22).

* Remove `vendor/sass`, which snuck into the gem by mistake
  and was causing trouble for Heroku users (thanks to [Jacques Crocker](http://railsjedi.com/)).

* `sass-convert` now understands better when it's acceptable
  to remove parentheses from expressions.

## 3.0.21

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.21).

* Fix the permissions errors for good.

* Fix more `#options` attribute errors.

## 3.0.20

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.20).

* Fix some permissions errors.

* Fix `#options` attribute errors when CSS functions were used with commas.

## 3.0.19

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.19).

* Make the alpha value for `rgba` colors respect
  {Sass::Script::Value::Number.precision}.

* Remove all newlines in selectors in `:compressed` mode.

* Make color names case-insensitive.

* Properly detect SCSS files when using `sass -c`.

* Remove spaces after commas in `:compressed` mode.

* Allow the `--unix-newlines` flag to work on Unix, where it's a no-op.

## 3.0.18

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.18).

* Don't require `rake` in the gemspec, for bundler compatibility under
  JRuby. Thanks to [Gordon McCreight](http://www.gmccreight.com/blog).

* Add a command-line option `--stop-on-error` that causes Sass to exit
  when a file fails to compile using `--watch` or `--update`.

* Fix a bug in `haml_tag` that would allow duplicate attributes to be added
  and make `data-` attributes not work.

* Get rid of the annoying RDoc errors on install.

* Disambiguate references to the `Rails` module when `haml-rails` is installed.

* Allow `@import` in SCSS to import multiple files in the same `@import` rule.

## 3.0.17

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.17).

* Disallow `#{}` interpolation in `@media` queries or unrecognized directives.
  This was never allowed, but now it explicitly throws an error
  rather than just producing invalid CSS.

* Make `sass --watch` not throw an error when passed a single file or directory.

* Understand that mingw counts as Windows.

* Make `sass --update` return a non-0 exit code if one or more files being updated
  contained an error.

## 3.0.16

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.16).

* Fix a bug where certain sorts of comments would get improperly
  rendered in the `:compact` style.

* Always allow a trailing `*/` in loud comments in the indented syntax.

* Fix a performance issue with SCSS parsing in rare cases.
  Thanks to [Chris Eppstein](http://chriseppstein.github.com).

* Use better heuristics for figuring out when someone might be using
  the wrong syntax with `sass --watch`.

## 3.0.15

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.15).

* Fix a bug where `sass --watch` and `sass --update` were completely broken.

* Allow `@import`ed values to contain commas.

## 3.0.14

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.14).

* Properly parse paths with drive letters on Windows (e.g. `C:\Foo\Bar.sass`)
  in the Sass executable.

* Compile Sass files in a deterministic order.

* Fix a bug where comments after `@if` statements in SCSS
  weren't getting passed through to the output document.

## 3.0.13

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.13).

## CSS `@import` Directives

Sass is now more intelligent about when to compile `@import` directives to plain CSS.
Any of the following conditions will cause a literal CSS `@import`:

* Importing a path with a `.css` extension (e.g. `@import "foo.css"`).
* Importing a path with a media type (e.g. `@import "foo" screen;`).
* Importing an HTTP path (e.g. `@import "http://foo.com/style.css"`).
* Importing any URL (e.g. `@import url(foo)`).

The former two conditions always worked, but the latter two are new.

## `-moz-calc` Support

The new [`-moz-calc()` function](http://hacks.mozilla.org/2010/06/css3-calc/) in Firefox 4
will now be properly parsed by Sass.
`calc()` was already supported, but because the parsing rules are different
than for normal CSS functions, this had to be expanded to include `-moz-calc`.

In anticipation of wider browser support, in fact,
*any* function named `-*-calc` (such as `-webkit-calc` or `-ms-calc`)
will be parsed the same as the `calc` function.

## `:-moz-any` Support

The [`:-moz-any` pseudoclass selector](http://hacks.mozilla.org/2010/05/moz-any-selector-grouping/)
is now parsed by Sass.

## `--require` Flag

The Sass command-line executable can now require Ruby files
using the `--require` flag (or `-r` for short).

## Rails Support

Make sure the default Rails options take precedence over the default non-Rails options.
This makes `./script/server --daemon` work again.

### Rails 3 Support

Support for Rails 3 versions prior to beta 4 has been removed.
Upgrade to Rails 3.0.0.beta4 if you haven't already.

## 3.0.12

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.12).

## Rails 3 Support

Apparently the last version broke in new and exciting ways under Rails 3,
due to the inconsistent load order caused by certain combinations of gems.
3.0.12 hacks around that inconsistency, and *should* be fully Rails 3-compatible.

### Deprecated: Rails 3 Beta 3

Haml's support for Rails 3.0.0.beta.3 has been deprecated.
Haml 3.0.13 will only support 3.0.0.beta.4.

## 3.0.11

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.11).

There were no changes made to Haml between versions 3.0.10 and 3.0.11.

## Rails 3 Support

Make sure Sass *actually* regenerates stylesheets under Rails 3.
The fix in 3.0.10 didn't work because the Rack stack we were modifying
wasn't reloaded at the proper time.

## Bug Fixes

* Give a decent error message when `--recursive` is used
  in `sass-convert` without a directory.

## 3.0.10

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.10).

### Appengine-JRuby Support

The way we determine the location of the Haml installation
no longer breaks the version of JRuby
used by [`appengine-jruby`](http://code.google.com/p/appengine-jruby/).

### Rails 3 Support

Sass will regenerate stylesheets under Rails 3
even when no controllers are being accessed.

### Other Improvements

* When using `sass-convert --from sass2 --to sass --recursive`,
  suggest the use of `--in-place` as well.

## 3.0.9

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.9).

There were no changes made to Sass between versions 3.0.8 and 3.0.9.
A bug in Gemcutter caused the gem to be uploaded improperly.

## 3.0.8

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.8).

* Fix a bug with Rails versions prior to Rails 3.

## 3.0.7

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.7).

### Encoding Support

Sass 3.0.7 adds support for `@charset` for declaring the encoding of a stylesheet.
For details see {file:SASS_REFERENCE.md#encodings the reference}.

The `sass` and `sass-convert` executables also now take an `-E` option
for specifying the encoding of Sass/SCSS/CSS files.

### Bug Fixes

* When compiling a file named `.sass` but with SCSS syntax specified,
  use the latter (and vice versa).

* Fix a bug where interpolation would cause some selectors to render improperly.

* If a line in a Sass comment starts with `*foo`,
  render it as `*foo` rather than `* *foo`.

## 3.0.6

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.6).

There were no changes made to Sass between versions 3.0.5 and 3.0.6.

## 3.0.5

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.5).

### `#{}` Interpolation in Properties

Previously, using `#{}` in some places in properties
would cause a syntax error.
Now it can be used just about anywhere.

Note that when `#{}` is used near operators like `/`,
those operators are treated as plain CSS
rather than math operators.
For example:

    p {
      $font-size: 12px;
      $line-height: 30px;
      font: #{$font-size}/#{$line-height};
    }

is compiled to:

    p {
      font: 12px/30px;
    }

This is useful, since normally {file:SASS_REFERENCE.md#division-and-slash
a slash with variables is treated as division}.

### Recursive Mixins

Mixins that include themselves will now print
much more informative error messages.
For example:

    @mixin foo {@include bar}
    @mixin bar {@include foo}
    @include foo

will print:

    An @include loop has been found:
        foo includes bar
        bar includes foo

Although it was previously possible to use recursive mixins
without causing infinite looping, this is now disallowed,
since there's no good reason to do it.

### Rails 3 Support

Fix Sass configuration under Rails 3.
Thanks [Dan Cheail](http://github.com/codeape).

### `sass --no-cache`

Make the `--no-cache` flag properly forbid Sass from writing `.sass-cache` files.

## 3.0.4

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.4).

* Raise an informative error when function arguments have a mispaced comma,
  as in `foo(bar, )`.

* Fix a performance problem when using long function names
  such as `-moz-linear-gradient`.

## 3.0.3

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.3).

### Rails 3 Support

Make sure Sass is loaded properly when using Rails 3
along with non-Rails-3-compatible plugins like some versions of `will_paginate`.

Also, In order to make some Rails loading errors like the above easier to debug,
Sass will now raise an error if `Rails.root` is `nil` when Sass is loading.
Previously, this would just cause the paths to be mis-set.

### Merb Support

Merb, including 1.1.0 as well as earlier versions,
should *really* work with this release.

### Bug Fixes

* Raise an informative error when mixin arguments have a mispaced comma,
  as in `@include foo(bar, )`.

* Make sure SassScript subtraction happens even when nothing else dynamic is going on.

* Raise an error when colors are used with the wrong number of digits.

## 3.0.2

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.2).

### Merb 1.1.0 Support

Fixed a bug inserting the Sass plugin into the Merb 1.1.0 Rack application.

### Bug Fixes

* Allow identifiers to begin with multiple underscores.

* Don't raise an error when using `haml --rails` with older Rails versions.

## 3.0.1

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.1).

### Installation in Rails

`haml --rails` is no longer necessary for installing Sass in Rails.
Now all you need to do is add `gem "haml"` to the Gemfile for Rails 3,
or add `config.gem "haml"` to `config/environment.rb` for previous versions.

`haml --rails` will still work,
but it has been deprecated and will print an error message.
It will not work in the next version of Sass.

### Rails 3 Beta Integration

* Make sure manually importing the Sass Rack plugin still works with Rails,
  even though it's not necessary now.

* Allow Sass to be configured in Rails even when it's being lazy-loaded.

### `:template_location` Methods

The {file:SASS_REFERENCE.md#template_location-option `:template_location` option}
can be either a String, a Hash, or an Array.
This makes it difficult to modify or use with confidence.
Thus, three new methods have been added for handling it:

* {Sass::Plugin::Configuration#template_location_array Sass::Plugin#template_location_array} --
  Returns the template locations and CSS locations formatted as an array.

* {Sass::Plugin::Configuration#add_template_location Sass::Plugin#add_template_location} --
  Converts the template location option to an array and adds a new location.

* {Sass::Plugin::Configuration#remove_template_location Sass::Plugin#remove_template_location} --
  Converts the template location option to an array and removes an existing location.

## 3.0.0
{#3-0-0}

[Tagged on GitHub](http://github.com/nex3/sass/commit/3.0.0).

### Deprecations -- Must Read!
{#3-0-0-deprecations}

* Using `=` for SassScript properties and variables is deprecated,
  and will be removed in Sass 3.2.
  Use `:` instead.
  See also [this changelog entry](#3-0-0-sass-script-context)

* Because of the above, property values using `:`
  will be parsed more thoroughly than they were before.
  Although all valid CSS3 properties
  as well as most hacks and proprietary syntax should be supported,
  it's possible that some properties will break.
  If this happens, please report it to [the Sass mailing list](http://groups.google.com/group/haml).

* In addition, setting the default value of variables
  with `||=` is now deprecated
  and will be removed in Sass 3.2.
  Instead, add `!default` to the end of the value.
  See also [this changelog entry](#3-0-0-default-flag)

* The `!` prefix for variables is deprecated,
  and will be removed in Sass 3.2.
  Use `$` as a prefix instead.
  See also [this changelog entry](#3-0-0-dollar-prefix).

* The `css2sass` command-line tool has been deprecated,
  and will be removed in Sass 3.2.
  Use the new `sass-convert` tool instead.
  See also [this changelog entry](#3-0-0-sass-convert).

* Selector parent references using `&` can now only be used
  where element names are valid.
  This is because Sass 3 fully parses selectors
  to support the new [`@extend` directive](#3-0-0-extend),
  and it's possible that the `&` could be replaced by an element name.

### SCSS (Sassy CSS)

Sass 3 introduces a new syntax known as SCSS
which is fully compatible with the syntax of CSS3,
while still supporting the full power of Sass.
This means that every valid CSS3 stylesheet
is a valid SCSS file with the same meaning.
In addition, SCSS understands most CSS hacks
and vendor-specific syntax, such as [IE's old `filter` syntax](http://msdn.microsoft.com/en-us/library/ms533754%28VS.85%29.aspx).

SCSS files use the `.scss` extension.
They can import `.sass` files, and vice-versa.
Their syntax is fully described in the {file:SASS_REFERENCE.md Sass reference};
if you're already familiar with Sass, though,
you may prefer the {file:SCSS_FOR_SASS_USERS.md intro to SCSS for Sass users}.

Since SCSS is a much more approachable syntax for those new to Sass,
it will be used as the default syntax for the reference,
as well as for most other Sass documentation.
The indented syntax will continue to be fully supported, however.

Sass files can be converted to SCSS using the new `sass-convert` command-line tool.
For example:

    # Convert a Sass file to SCSS
    $ sass-convert style.sass style.scss

**Note that if you're converting a Sass file written for Sass 2**,
you should use the `--from sass2` flag.
For example:

    # Convert a Sass file to SCSS
    $ sass-convert --from sass2 style.sass style.scss

    # Convert all Sass files to SCSS
    $ sass-convert --recursive --in-place --from sass2 --to scss stylesheets/

### Syntax Changes {#3-0-0-syntax-changes}

#### SassScript Context
{#3-0-0-sass-script-context}

The `=` character is no longer required for properties that use SassScript
(that is, variables and operations).
All properties now use SassScript automatically;
this means that `:` should be used instead.
Variables should also be set with `:`.
For example, what used to be

    // Indented syntax
    .page
      color = 5px + 9px

should now be

    // Indented syntax
    .page
      color: 5px + 9px

This means that SassScript is now an extension of the CSS3 property syntax.
All valid CSS3 properties are valid SassScript,
and will compile without modification
(some invalid properties work as well, such as Microsoft's proprietary `filter` syntax).
This entails a few changes to SassScript to make it fully CSS3-compatible,
which are detailed below.

This also means that Sass will now be fully parsing all property values,
rather than passing them through unchanged to the CSS.
Although care has been taken to support all valid CSS3,
as well as hacks and proprietary syntax,
it's possible that a property that worked in Sass 2 won't work in Sass 3.
If this happens, please report it to [the Sass mailing list](http://groups.google.com/group/haml).

Note that if `=` is used,
SassScript will be interpreted as backwards-compatibly as posssible.
In particular, the changes listed below don't apply in an `=` context.

The `sass-convert` command-line tool can be used
to upgrade Sass files to the new syntax using the `--in-place` flag.
For example:

    # Upgrade style.sass:
    $ sass-convert --in-place style.sass

    # Upgrade all Sass files:
    $ sass-convert --recursive --in-place --from sass2 --to sass stylesheets/

##### Quoted Strings

Quoted strings (e.g. `"foo"`) in SassScript now render with quotes.
In addition, unquoted strings are no longer deprecated,
and render without quotes.
This means that almost all strings that had quotes in Sass 2
should not have quotes in Sass 3.

Although quoted strings render with quotes when used with `:`,
they do not render with quotes when used with `#{}`.
This allows quoted strings to be used for e.g. selectors
that are passed to mixins.

Strings can be forced to be quoted and unquoted using the new
\{Sass::Script::Functions#unquote unquote} and \{Sass::Script::Functions#quote quote}
functions.

##### Division and `/`

Two numbers separated by a `/` character
are allowed as property syntax in CSS,
e.g. for the `font` property.
SassScript also uses `/` for division, however,
which means it must decide what to do
when it encounters numbers separated by `/`.

For CSS compatibility, SassScript does not perform division by default.
However, division will be done in almost all cases where division is intended.
In particular, SassScript will perform division
in the following three situations:

1. If the value, or any part of it, is stored in a variable.
2. If the value is surrounded by parentheses.
3. If the value is used as part of another arithmetic expression.

For example:

    p
      font: 10px/8px
      $width: 1000px
      width: $width/2
      height: (500px/2)
      margin-left: 5px + 8px/2px

is compiled to:

    p {
      font: 10px/8px;
      width: 500px;
      height: 250px;
      margin-left: 9px; }

##### Variable Defaults

Since `=` is no longer used for variable assignment,
assigning defaults to variables with `||=` no longer makes sense.
Instead, the `!default` flag
should be added to the end of the variable value.
This syntax is meant to be similar to CSS's `!important` flag.
For example:

    $var: 12px !default;

#### Variable Prefix Character
{#3-0-0-dollar-prefix}

The Sass variable character has been changed from `!`
to the more aesthetically-appealing `$`.
For example, what used to be

    !width = 13px
    .icon
      width = !width

should now be

    $width: 13px
    .icon
      width: $width

The `sass-convert` command-line tool can be used
to upgrade Sass files to the new syntax using the `--in-place` flag.
For example:

    # Upgrade style.sass:
    $ sass-convert --in-place style.sass

    # Upgrade all Sass files:
    $ sass-convert --recursive --in-place --from sass2 --to sass stylesheets/

`!` may still be used, but it's deprecated and will print a warning.
It will be removed in the next version of Sass, 3.2.

#### Variable and Mixin Names

SassScript variable and mixin names may now contain hyphens.
In fact, they may be any valid CSS3 identifier.
For example:

    $prettiest-color: #542FA9
    =pretty-text
      color: $prettiest-color

In order to allow frameworks like [Compass](http://compass-style.org)
to use hyphens in variable names
while maintaining backwards-compatibility,
variables and mixins using hyphens may be referred to
with underscores, and vice versa.
For example:

    $prettiest-color: #542FA9
    .pretty
      // Using an underscore instead of a hyphen works
      color: $prettiest_color

#### Single-Quoted Strings

SassScript now supports single-quoted strings.
They behave identically to double-quoted strings,
except that single quotes need to be backslash-escaped
and double quotes do not.

#### Mixin Definition and Inclusion

Sass now supports the `@mixin` directive as a way of defining mixins (like `=`),
as well as the `@include` directive as a way of including them (like `+`).
The old syntax is *not* deprecated,
and the two are fully compatible.
For example:

    @mixin pretty-text
      color: $prettiest-color

    a
      @include pretty-text

is the same as:

    =pretty-text
      color: $prettiest-color

    a
      +pretty-text

#### Sass Properties

New-style properties (with the colon after the name) in indented syntax
now allow whitespace before the colon. For example:

    foo
      color : blue

#### Sass `@import`

The Sass `@import` statement now allows non-CSS files to be specified with quotes,
for similarity with the SCSS syntax. For example, `@import "foo.sass"`
will now import the `foo.sass` file, rather than compiling to `@import "foo.sass";`.

### `@extend`
{#3-0-0-extend}

There are often cases when designing a page
when one class should have all the styles of another class,
as well as its own specific styles.
The most common way of handling this is to use both the more general class
and the more specific class in the HTML.
For example, suppose we have a design for a normal error
and also for a serious error. We might write our markup like so:

    <div class="error seriousError">
      Oh no! You've been hacked!
    </div>

And our styles like so:

    .error {
      border: 1px #f00;
      background-color: #fdd;
    }
    .seriousError {
      border-width: 3px;
    }

Unfortunately, this means that we have to always remember
to use `.error` with `.seriousError`.
This is a maintenance burden, leads to tricky bugs,
and can bring non-semantic style concerns into the markup.

The `@extend` directive avoids these problems
by telling Sass that one selector should inherit the styles of another selector.
For example:

    .error {
      border: 1px #f00;
      background-color: #fdd;
    }
    .seriousError {
      @extend .error;
      border-width: 3px;
    }

This means that all styles defined for `.error`
are also applied to `.seriousError`,
in addition to the styles specific to `.seriousError`.
In effect, everything with class `.seriousError` also has class `.error`.

Other rules that use `.error` will work for `.seriousError` as well.
For example, if we have special styles for errors caused by hackers:

    .error.intrusion {
      background-image: url("/image/hacked.png");
    }

Then `<div class="seriousError intrusion">`
will have the `hacked.png` background image as well.

#### How it Works

`@extend` works by inserting the extending selector (e.g. `.seriousError`)
anywhere in the stylesheet that the extended selector (.e.g `.error`) appears.
Thus the example above:

    .error {
      border: 1px #f00;
      background-color: #fdd;
    }
    .error.intrusion {
      background-image: url("/image/hacked.png");
    }
    .seriousError {
      @extend .error;
      border-width: 3px;
    }

is compiled to:

    .error, .seriousError {
      border: 1px #f00;
      background-color: #fdd; }

    .error.intrusion, .seriousError.intrusion {
      background-image: url("/image/hacked.png"); }

    .seriousError {
      border-width: 3px; }

When merging selectors, `@extend` is smart enough
to avoid unnecessary duplication,
so something like `.seriousError.seriousError` gets translated to `.seriousError`.
In addition, it won't produce selectors that can't match anything, like `#main#footer`.

See also {file:SASS_REFERENCE.md#extend the `@extend` reference documentation}.

### Colors

SassScript color values are much more powerful than they were before.
Support was added for alpha channels,
and most of Chris Eppstein's [compass-colors](http://chriseppstein.github.com/compass-colors) plugin
was merged in, providing color-theoretic functions for modifying colors.

One of the most interesting of these functions is {Sass::Script::Functions#mix mix},
which mixes two colors together.
This provides a much better way of combining colors and creating themes
than standard color arithmetic.

#### Alpha Channels

Sass now supports colors with alpha channels,
constructed via the {Sass::Script::Functions#rgba rgba}
and {Sass::Script::Functions#hsla hsla} functions.
Alpha channels are unaffected by color arithmetic.
However, the {Sass::Script::Functions#opacify opacify}
and {Sass::Script::Functions#transparentize transparentize} functions
allow colors to be made more and less opaque, respectively.

Sass now also supports functions that return the values of the
{Sass::Script::Functions#red red},
{Sass::Script::Functions#blue blue},
{Sass::Script::Functions#green green},
and {Sass::Script::Functions#alpha alpha}
components of colors.

#### HSL Colors

Sass has many new functions for using the HSL values of colors.
For an overview of HSL colors, check out [the CSS3 Spec](http://www.w3.org/TR/css3-color/#hsl-color).
All these functions work just as well on RGB colors
as on colors constructed with the {Sass::Script::Functions#hsl hsl} function.

* The {Sass::Script::Functions#lighten lighten}
  and {Sass::Script::Functions#darken darken}
  functions adjust the lightness of a color.

* The {Sass::Script::Functions#saturate saturate}
  and {Sass::Script::Functions#desaturate desaturate}
  functions adjust the saturation of a color.

* The {Sass::Script::Functions#adjust_hue adjust-hue}
  function adjusts the hue of a color.

* The {Sass::Script::Functions#hue hue},
  {Sass::Script::Functions#saturation saturation},
  and {Sass::Script::Functions#lightness lightness}
  functions return the corresponding HSL values of the color.

* The {Sass::Script::Functions#grayscale grayscale}
  function converts a color to grayscale.

* The {Sass::Script::Functions#complement complement}
  function returns the complement of a color.

### Other New Functions

Several other new functions were added to make it easier to have
more flexible arguments to mixins and to enable deprecation
of obsolete APIs.

* {Sass::Script::Functions#type_of `type-of`} -- Returns the type of a value.
* {Sass::Script::Functions#unit `unit`} --
  Returns the units associated with a number.
* {Sass::Script::Functions#unitless `unitless`} --
  Returns whether a number has units or not.
* {Sass::Script::Functions#comparable `comparable`} --
  Returns whether two numbers can be added or compared.

### Watching for Updates
{#3-0-0-watch}

The `sass` command-line utility has a new flag: `--watch`.
`sass --watch` monitors files or directories for updated Sass files
and compiles those files to CSS automatically.
This will allow people not using Ruby or [Compass](http://compass-style.org)
to use Sass without having to manually recompile all the time.

Here's the syntax for watching a directory full of Sass files:

    sass --watch app/stylesheets:public/stylesheets

This will watch every Sass file in `app/stylesheets`.
Whenever one of them changes,
the corresponding CSS file in `public/stylesheets` will be regenerated.
Any files that import that file will be regenerated, too.

The syntax for watching individual files is the same:

    sass --watch style.sass:out.css

You can also omit the output filename if you just want it to compile to name.css.
For example:

    sass --watch style.sass

This will update `style.css` whenever `style.sass` changes.

You can list more than one file and/or directory,
and all of them will be watched:

    sass --watch foo/style:public/foo bar/style:public/bar
    sass --watch screen.sass print.sass awful-hacks.sass:ie.css
    sass --watch app/stylesheets:public/stylesheets public/stylesheets/test.sass

File and directory watching is accessible from Ruby,
using the {Sass::Plugin::Compiler#watch Sass::Plugin#watch} function.

#### Bulk Updating

Another new flag for the `sass` command-line utility is `--update`.
It checks a group of Sass files to see if their CSS needs to be updated,
and updates if so.

The syntax for `--update` is just like watch:

    sass --update app/stylesheets:public/stylesheets
    sass --update style.sass:out.css
    sass --watch screen.sass print.sass awful-hacks.sass:ie.css

In fact, `--update` work exactly the same as `--watch`,
except that it doesn't continue watching the files
after the first check.

### `sass-convert` (née `css2sass`) {#3-0-0-sass-convert}

The `sass-convert` tool, which used to be known as `css2sass`,
has been greatly improved in various ways.
It now uses a full-fledged CSS3 parser,
so it should be able to handle any valid CSS3,
as well as most hacks and proprietary syntax.

`sass-convert` can now convert between Sass and SCSS.
This is normally inferred from the filename,
but it can also be specified using the `--from` and `--to` flags.
For example:

    $ generate-sass | sass-convert --from sass --to scss | consume-scss

It's also now possible to convert a file in-place --
that is, overwrite the old file with the new file.
This is useful for converting files in the [Sass 2 syntax](#3-0-0-deprecations)
to the new Sass 3 syntax,
e.g. by doing `sass-convert --in-place --from sass2 style.sass`.

#### `--recursive`

The `--recursive` option allows `sass-convert` to convert an entire directory of files.
`--recursive` requires both the `--from` and `--to` flags to be specified.
For example:

    # Convert all .sass files in stylesheets/ to SCSS.
    # "sass2" means that these files are assumed to use the Sass 2 syntax.
    $ sass-convert --recursive --from sass2 --to scss stylesheets/

#### `--dasherize`

The `--dasherize` options converts all underscores to hyphens,
which are now allowed as part of identifiers in Sass.
Note that since underscores may still be used in place of hyphens
when referring to mixins and variables,
this won't cause any backwards-incompatibilities.

#### Convert Less to SCSS

`sass-convert` can also convert [Less](http://lesscss.org) files
to SCSS (or the indented syntax, although I anticipate less interest in that).
For example:

    # Convert all .less files in the current directory into .scss files
    sass-convert --from less --to scss --recursive .

This is done using the Less parser, so it requires that the `less` RubyGem be installed.

##### Incompatibilities

Because of the reasonably substantial differences between Sass and Less,
there are some things that can't be directly translated,
and one feature that can't be translated at all.
In the tests I've run on open-source Less stylesheets,
none of these have presented issues, but it's good to be aware of them.

First, Less doesn't distinguish fully between mixins and selector inheritance.
In Less, all classes and some other selectors may be used as mixins,
alongside more Sass-like mixins.
If a class is being used as a mixin,
it may also be used directly in the HTML,
so it's not safe to translate it into a Sass mixin.
What `sass-convert` does instead is leave the class in the stylesheet as a class,
and use {file:SASS_REFERENCE.md#extend `@extend`}
rather than {file:SASS_REFERENCE.md#including_a_mixin `@include`}
to take on the styles of that class.
Although `@extend` and mixins work quite differently,
using `@extend` here doesn't actually seem to make a difference in practice.

Another issue with Less mixins is that Less allows nested selectors
(such as `.body .button` or `.colors > .teal`) to be used
as a means of "namespacing" mixins.
Sass's `@extend` doesn't work that way,
so it does away with the namespacing and just extends the base class
(so `.colors > .teal` becomes simply `@extend .teal`).
In practice, this feature doesn't seem to be widely-used,
but `sass-convert` will print a warning and leave a comment
when it encounters it just in case.

Finally, Less has the ability to directly access variables and property values
defined in other selectors, which Sass does not support.
Whenever such an accessor is used,
`sass-convert` will print a warning
and comment it out in the SCSS output.
Like namespaced mixins, though,
this does not seem to be a widely-used feature.

### `@warn` Directive

A new directive `@warn` has been added that allows Sass libraries to emit warnings.
This can be used to issue deprecation warnings, discourage sloppy use of mixins, etc.
`@warn` takes a single argument: a SassScript expression that will be
displayed on the console along with a stylesheet trace for locating the warning.
For example:

    @mixin blue-text {
      @warn "The blue-text mixin is deprecated. Use new-blue-text instead.";
      color: #00f;
    }

Warnings may be silenced with the new `--quiet` command line option,
or the corresponding {file:SASS_REFERENCE.md#quiet-option `:quiey` Sass option}.
This option will also affect warnings printed by Sass itself.
Warnings are off by default in the Rails, Rack, and Merb production environments.

### Sass::Plugin API

{Sass::Plugin} now has a large collection of callbacks that allow users
to run code when various actions are performed.
For example:

    Sass::Plugin.on_updating_stylesheet do |template, css|
      puts "#{template} has been compiled to #{css}!"
    end

For a full list of callbacks and usage notes, see the {Sass::Plugin} documentation.

{Sass::Plugin} also has a new method,
{Sass::Plugin#force_update_stylesheets force_update_stylesheets}.
This works just like {Sass::Plugin#update_stylesheets},
except that it doesn't check modification times and doesn't use the cache;
all stylesheets are always compiled anew.

### Output Formatting

Properties with a value and *also* nested properties
are now rendered with the nested properties indented.
For example:

    margin: auto
      top: 10px
      bottom: 20px

is now compiled to:

    margin: auto;
      margin-top: 10px;
      margin-bottom: 20px;

#### `:compressed` Style

When the `:compressed` style is used,
colors will be output as the minimal possible representation.
This means whichever is smallest of the HTML4 color name
and the hex representation (shortened to the three-letter version if possible).

### Stylesheet Updating Speed

Several caching layers were added to Sass's stylesheet updater.
This means that it should run significantly faster.
This benefit will be seen by people using Sass in development mode
with Rails, Rack, and Merb,
as well as people using `sass --watch` from the command line,
and to a lesser (but still significant) extent `sass --update`.
Thanks to [thedarkone](http://github.com/thedarkone).

### Error Backtraces

Numerous bugs were fixed with the backtraces given for Sass errors,
especially when importing files and using mixins.
All imports and mixins will now show up in the Ruby backtrace,
with the proper filename and line number.

In addition, when the `sass` executable encounters an error,
it now prints the filename where the error occurs,
as well as a backtrace of Sass imports and mixins.

### Ruby 1.9 Support

* Sass and `css2sass` now produce more descriptive errors
  when given a template with invalid byte sequences for that template's encoding,
  including the line number and the offending character.

* Sass and `css2sass` now accept Unicode documents with a
  [byte-order-mark](http://en.wikipedia.org/wiki/Byte_order_mark).

### Firebug Support

A new {file:SASS_REFERENCE.md#debug_info-option `:debug_info` option}
has been added that emits line-number and filename information
to the CSS file in a browser-readable format.
This can be used with the new [FireSass Firebug extension](https://addons.mozilla.org/en-US/firefox/addon/103988)
to report the Sass filename and line number for generated CSS files.

This is also available via the `--debug-info` command-line flag.

### Minor Improvements

* If a CSS or Sass function is used that has the name of a color,
  it will now be parsed as a function rather than as a color.
  For example, `fuchsia(12)` now renders as `fuchsia(12)`
  rather than `fuchsia 12`,
  and `tealbang(12)` now renders as `tealbang(12)`
  rather than `teal bang(12)`.

* The Sass Rails and Merb plugins now use Rack middleware by default.

* Haml is now compatible with the [Rip](http://hellorip.com/) package management system.
  Thanks to [Josh Peek](http://joshpeek.com/).

* Indented-syntax `/*` comments may now include `*` on lines beyond the first.

* A {file:SASS_REFERENCE.md#read_cache-option `:read_cache`} option has been added
  to allow the Sass cache to be read from but not written to.

* Stylesheets are no longer checked during each request
  when running tests in Rails.
  This should speed up some tests significantly.

## 2.2.24

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.24).

* Parent references -- the `&` character --
  may only be placed at the beginning of simple selector sequences in Sass 3.
  Placing them elsewhere is deprecated in 2.2.24 and will print a warning.
  For example, `foo &.bar` is allowed, but `foo .bar&` is not.

## 2.2.23

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.23).

* Don't crash when `rake gems` is run in Rails with Sass installed.
  Thanks to [Florian Frank](http://github.com/flori).

* When raising a file-not-found error,
  add a list of load paths that were checked.

* If an import isn't found for a cached Sass file and the
  {file:SASS_REFERENCE.md#full_exception `:full_exception option`} is enabled,
  print the full exception rather than raising it.

* Fix a bug with a weird interaction with Haml, DataMapper, and Rails 3
  that caused some tag helpers to go into infinite recursion.

## 2.2.22

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.22).

* Add a railtie so Haml and Sass will be automatically loaded in Rails 3.
  Thanks to [Daniel Neighman](http://pancakestacks.wordpress.com/).

* Make loading the gemspec not crash on read-only filesystems like Heroku's.

## 2.2.21

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.21).

* Fix a few bugs in the git-revision-reporting in {Sass::Version#version}.
  In particular, it will still work if `git gc` has been called recently,
  or if various files are missing.

* Always use `__FILE__` when reading files within the Haml repo in the `Rakefile`.
  According to [this bug report](http://github.com/carlhuda/bundler/issues/issue/44),
  this should make Sass work better with Bundler.

## 2.2.20

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.20).

* If the cache file for a given Sass file is corrupt
  because it doesn't have enough content,
  produce a warning and read the Sass file
  rather than letting the exception bubble up.
  This is consistent with other sorts of sassc corruption handling.

* Calls to `defined?` shouldn't interfere with Rails' autoloading
  in very old versions (1.2.x).

## 2.2.19

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.18).

There were no changes made to Sass between versions 2.2.18 and 2.2.19.

## 2.2.18

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.18).

* Use `Rails.env` rather than `RAILS_ENV` when running under Rails 3.0.
  Thanks to [Duncan Grazier](http://duncangrazier.com/).

* Support `:line_numbers` as an alias for {file:SASS_REFERENCE.md#line_numbers-option `:line_comments`},
  since that's what the docs have said forever.
  Similarly, support `--line-numbers` as a command-line option.

* Add a `--unix-newlines` flag to all executables
  for outputting Unix-style newlines on Windows.

* Add a {file:SASS_REFERENCE.md#unix_newlines-option `:unix_newlines` option}
  for {Sass::Plugin} for outputting Unix-style newlines on Windows.

* Fix the `--cache-location` flag, which was previously throwing errors.
  Thanks to [tav](http://tav.espians.com/).

* Allow comments at the beginning of the document to have arbitrary indentation,
  just like comments elsewhere.
  Similarly, comment parsing is a little nicer than before.

## 2.2.17

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.16).

* When the {file:SASS_REFERENCE.md#full_exception-option `:full_exception` option}
  is false, raise the error in Ruby code rather than swallowing it
  and printing something uninformative.

* Fixed error-reporting when something goes wrong when loading Sass
  using the `sass` executable.
  This used to raise a NameError because `Sass::SyntaxError` wasn't defined.
  Now it'll raise the correct exception instead.

* Report the filename in warnings about selectors without properties.

* `nil` values for Sass options are now ignored,
  rather than raising errors.

* Fix a bug that appears when Plugin template locations
  have multiple trailing slashes.
  Thanks to [Jared Grippe](http://jaredgrippe.com/).

### Must Read!

* When `@import` is given a filename without an extension,
  the behavior of rendering a CSS `@import` if no Sass file is found
  is deprecated.
  In future versions, `@import foo` will either import the template
  or raise an error.

## 2.2.16

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.16).

* Fixed a bug where modules containing user-defined Sass functions
  weren't made available when simply included in {Sass::Script::Functions}
  ({Sass::Script::Functions Functions} needed to be re-included in
  {Sass::Script::Functions::EvaluationContext Functions::EvaluationContext}).
  Now the module simply needs to be included in {Sass::Script::Functions}.

## 2.2.15

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.15).

* Added {Sass::Script::Value::Color#with} for a way of setting color channels
  that's easier than manually constructing a new color
  and is forwards-compatible with alpha-channel colors
  (to be introduced in Sass 2.4).

* Added a missing require in Sass that caused crashes
  when it was being run standalone.

## 2.2.14

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.14).

* All Sass functions now raise explicit errors if their inputs
  are of the incorrect type.

* Allow the SassScript `rgb()` function to take percentages
  in addition to numerical values.

* Fixed a bug where SassScript strings with `#` followed by `#{}` interpolation
  didn't evaluate the interpolation.

### SassScript Ruby API

These changes only affect people defining their own Sass functions
using {Sass::Script::Functions}.

* `Sass::Script::Color#value` attribute is deprecated.
  Use {Sass::Script::Value::Color#rgb} instead.
  The returned array is now frozen as well.

* Add an `assert_type` function that's available to {Sass::Script::Functions}.
  This is useful for typechecking the inputs to functions.

### Rack Support

Sass 2.2.14 includes Rack middleware for running Sass,
meaning that all Rack-enabled frameworks can now use Sass.
To activate this, just add

    require 'sass/plugin/rack'
    use Sass::Plugin::Rack

to your `config.ru`.
See the {Sass::Plugin::Rack} documentation for more details.

## 2.2.13

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.13).

There were no changes made to Sass between versions 2.2.12 and 2.2.13.

## 2.2.12

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.12).

* Fix a stupid bug introduced in 2.2.11 that broke the Sass Rails plugin.

## 2.2.11

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.11).

* Added a note to errors on properties that could be pseudo-classes (e.g. `:focus`)
  indicating that they should be backslash-escaped.

* Automatically interpret properties that could be pseudo-classes as such
  if {file:SASS_REFERENCE.md.html#property_syntax-option `:property_syntax`}
  is set to `:new`.

* Fixed `css2sass`'s generation of pseudo-classes so that they're backslash-escaped.

* Don't crash if the Haml plugin skeleton is installed and `rake gems:install` is run.

* Don't use `RAILS_ROOT` directly.
  This no longer exists in Rails 3.0.
  Instead abstract this out as `Haml::Util.rails_root`.
  This changes makes Haml fully compatible with edge Rails as of this writing.

* Make use of a Rails callback rather than a monkeypatch to check for stylesheet updates
  in Rails 3.0+.

## 2.2.10

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.10).

* Add support for attribute selectors with spaces around the `=`.
  For example:

      a[href = http://google.com]
        color: blue

## 2.2.9

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.9).

There were no changes made to Sass between versions 2.2.8 and 2.2.9.

## 2.2.8

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.8).

There were no changes made to Sass between versions 2.2.7 and 2.2.8.

## 2.2.7

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.7).

There were no changes made to Sass between versions 2.2.6 and 2.2.7.

## 2.2.6

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.6).

* Don't crash when the `__FILE__` constant of a Ruby file is a relative path,
  as apparently happens sometimes in TextMate
  (thanks to [Karl Varga](http://github.com/kjvarga)).

* Add "Sass" to the `--version` string for the executables.

## 2.2.5

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.5).

There were no changes made to Sass between versions 2.2.4 and 2.2.5.

## 2.2.4

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.4).

* Don't add `require 'rubygems'` to the top of init.rb when installed
  via `sass --rails`. This isn't necessary, and actually gets
  clobbered as soon as haml/template is loaded.

* Document the previously-undocumented {file:SASS_REFERENCE.md#line-option `:line` option},
  which allows the number of the first line of a Sass file to be set for error reporting.

## 2.2.3

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.3).

Sass 2.2.3 prints line numbers for warnings about selectors
with no properties.

## 2.2.2

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.2).

Sass 2.2.2 is a minor bug-fix release.
Notable changes include better parsing of mixin definitions and inclusions
and better support for Ruby 1.9.

## 2.2.1

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.1).

Sass 2.2.1 is a minor bug-fix release.

### Must Read!

* It used to be acceptable to use `-` immediately following variable names,
  without any whitespace in between (for example, `!foo-!bar`).
  This is now deprecated, so that in the future variables with hyphens
  can be supported. Surround `-` with spaces.

## 2.2.0

[Tagged on GitHub](http://github.com/nex3/sass/commit/2.2.0).

The 2.2 release marks a significant step in the evolution of the Sass
language. The focus has been to increase the power of Sass to keep
your stylesheets maintainable by allowing new forms of abstraction to
be created within your stylesheets and the stylesheets provided by
others that you can download and import into your own. The fundamental
units of abstraction in Sass are variables and mixins. Please read
below for a list of changes:

### Must Read!

* Sass Comments (//) used to only comment out a single line. This was deprecated
  in 2.0.10 and starting in 2.2, Sass comments will comment out any lines indented
  under them. Upgrade to 2.0.10 in order to see deprecation warnings where this change
  affects you.

* Implicit Strings within SassScript are now deprecated and will be removed in 2.4.
  For example: `border= !width solid #00F` should now be written as `border: #{!width} solid #00F`
  or as `border= !width "solid" #00F`. After upgrading to 2.2, you will see deprecation warnings
  if you have sass files that use implicit strings.


### Sass Syntax Changes

#### Flexible Indentation

The indentation of Sass documents is now flexible. The first indent
that is detected will determine the indentation style for that
document. Tabs and spaces may never be mixed, but within a document,
you may choose to use tabs or a flexible number of spaces.

#### Multiline Sass Comments

Sass Comments (//) will now comment out whatever is indented beneath
them. Previously they were single line when used at the top level of a
document. Upgrading to the latest stable version will give you
deprecation warnings if you have silent comments with indentation
underneath them.

#### Mixin Arguments

Sass Mixins now accept any number of arguments. To define a mixin with
arguments, specify the arguments as a comma-delimited list of
variables like so:

    =my-mixin(!arg1, !arg2, !arg3)

As before, the definition of the mixin is indented below the mixin
declaration. The variables declared in the argument list may be used
and will be bound to the values passed to the mixin when it is
invoked.  Trailing arguments may have default values as part of the
declaration:

    =my-mixin(!arg1, !arg2 = 1px, !arg3 = blue)

In the example above, the mixin may be invoked by passing 1, 2 or 3
arguments to it. A similar syntax is used to invoke a mixin that
accepts arguments:

    div.foo
      +my-mixin(1em, 3px)

When a mixin has no required arguments, the parenthesis are optional.

The default values for mixin arguments are evaluated in the global
context at the time when the mixin is invoked, they may also reference
the previous arguments in the declaration. For example:

    !default_width = 30px
    =my-fancy-mixin(!width = !default_width, !height = !width)
      width= !width
      height= !height

    .default-box
      +my-fancy-mixin

    .square-box
      +my-fancy-mixin(50px)

    .rectangle-box
      +my-fancy-mixin(25px, 75px)

    !default_width = 10px
    .small-default-box
      +my-fancy-mixin


compiles to:

    .default-box {
      width: 30px;
      height: 30px; }

    .square-box {
      width: 50px;
      height: 50px; }

    .rectangle-box {
      width: 25px;
      height: 75px; }

    .small-default-box {
      width: 10px;
      height: 10px; }


### Sass, Interactive

The sass command line option -i now allows you to quickly and
interactively experiment with SassScript expressions. The value of the
expression you enter will be printed out after each line. Example:

    $ sass -i
    >> 5px
    5px
    >> 5px + 10px
    15px
    >> !five_pixels = 5px
    5px
    >> !five_pixels + 10px
    15px

### SassScript

The features of SassScript have been greatly enhanced with new control
directives, new fundamental data types, and variable scoping.

#### New Data Types

SassScript now has four fundamental data types:

1. Number
2. String
3. Boolean (New in 2.2)
4. Colors

#### More Flexible Numbers

Like JavaScript, SassScript numbers can now change between floating
point and integers. No explicit casting or decimal syntax is
required. When a number is emitted into a CSS file it will be rounded
to the nearest thousandth, however the internal representation
maintains much higher precision.

#### Improved Handling of Units

While Sass has long supported numbers with units, it now has a much
deeper understanding of them. The following are examples of legal
numbers in SassScript:

    0, 1000, 6%, -2px, 5pc, 20em, or 2foo.

Numbers of the same unit may always be added and subtracted. Numbers
that have units that Sass understands and finds comparable, can be
combined, taking the unit of the first number. Numbers that have
non-comparable units may not be added nor subtracted -- any attempt to
do so will cause an error. However, a unitless number takes on the
unit of the other number during a mathematical operation. For example:

    >> 3mm + 4cm
    43mm
    >> 4cm + 3mm
    4.3cm
    >> 3cm + 2in
    8.08cm
    >> 5foo + 6foo
    11foo
    >> 4% + 5px
    SyntaxError: Incompatible units: 'px' and '%'.
    >> 5 + 10px
    15px

Sass allows compound units to be stored in any intermediate form, but
will raise an error if you try to emit a compound unit into your css
file.

    >> !em_ratio = 1em / 16px
    0.063em/px
    >> !em_ratio * 32px
    2em
    >> !em_ratio * 40px
    2.5em

#### Colors

A color value can be declared using a color name, hexadecimal,
shorthand hexadecimal, the rgb function, or the hsl function. When
outputting a color into css, the color name is used, if any, otherwise
it is emitted as hexadecimal value. Examples:

    > #fff
    white
    >> white
    white
    >> #FFFFFF
    white
    >> hsl(180, 100, 100)
    white
    >> rgb(255, 255, 255)
    white
    >> #AAA
    #aaaaaa

Math on color objects is performed piecewise on the rgb
components. However, these operations rarely have meaning in the
design domain (mostly they make sense for gray-scale colors).

    >> #aaa + #123
    #bbccdd
    >> #333 * 2
    #666666

#### Booleans

Boolean objects can be created by comparison operators or via the
`true` and `false` keywords.  Booleans can be combined using the
`and`, `or`, and `not` keywords.

    >> true
    true
    >> true and false
    false
    >> 5 < 10
    true
    >> not (5 < 10)
    false
    >> not (5 < 10) or not (10 < 5)
    true
    >> 30mm == 3cm
    true
    >> 1px == 1em
    false

#### Strings

Unicode escapes are now allowed within SassScript strings.

### Control Directives

New directives provide branching and looping within a sass stylesheet
based on SassScript expressions. See the [Sass
Reference](SASS_REFERENCE.md.html#control_directives) for complete
details.

#### @for

The `@for` directive loops over a set of numbers in sequence, defining
the current number into the variable specified for each loop. The
`through` keyword means that the last iteration will include the
number, the `to` keyword means that it will stop just before that
number.

    @for !x from 1px through 5px
      .border-#{!x}
        border-width= !x

compiles to:

    .border-1px {
      border-width: 1px; }

    .border-2px {
      border-width: 2px; }

    .border-3px {
      border-width: 3px; }

    .border-4px {
      border-width: 4px; }

    .border-5px {
      border-width: 5px; }

#### @if / @else if / @else

The branching directives `@if`, `@else if`, and `@else` let you select
between several branches of sass to be emitted, based on the result of
a SassScript expression. Example:

    !type = "monster"
    p
      @if !type == "ocean"
        color: blue
      @else if !type == "matador"
        color: red
      @else if !type == "monster"
        color: green
      @else
        color: black

is compiled to:

    p {
      color: green; }

#### @while

The `@while` directive lets you iterate until a condition is
met. Example:

    !i = 6
    @while !i > 0
      .item-#{!i}
        width = 2em * !i
      !i = !i - 2

is compiled to:

    .item-6 {
      width: 12em; }

    .item-4 {
      width: 8em; }

    .item-2 {
      width: 4em; }

### Variable Scoping

The term "constant" has been renamed to "variable." Variables can be
declared at any scope (a.k.a. nesting level) and they will only be
visible to the code until the next outdent. However, if a variable is
already defined in a higher level scope, setting it will overwrite the
value stored previously.

In this code, the `!local_var` variable is scoped and hidden from
other higher level scopes or sibling scopes:

    .foo
      .bar
        !local_var = 1px
        width= !local_var
      .baz
        // this will raise an undefined variable error.
        width= !local_var
      // as will this
      width= !local_var

In this example, since the `!global_var` variable is first declared at
a higher scope, it is shared among all lower scopes:

    !global_var = 1px
    .foo
      .bar
        !global_var = 2px
        width= !global_var
      .baz
        width= !global_var
      width= !global_var

compiles to:

    .foo {
      width: 2px; }
      .foo .bar {
        width: 2px; }
      .foo .baz {
        width: 2px; }


### Interpolation

Interpolation has been added. This allows SassScript to be used to
create dynamic properties and selectors.  It also cleans up some uses
of dynamic values when dealing with compound properties. Using
interpolation, the result of a SassScript expression can be placed
anywhere:

    !x = 1
    !d = 3
    !property = "border"
    div.#{!property}
      #{!property}: #{!x + !d}px solid
      #{!property}-color: blue

is compiled to:

    div.border {
      border: 4px solid;
      border-color: blue; }

### Sass Functions

SassScript defines some useful functions that are called using the
normal CSS function syntax:

    p
      color = hsl(0, 100%, 50%)

is compiled to:

    #main {
      color: #ff0000; }

The following functions are provided: `hsl`, `percentage`, `round`,
`ceil`, `floor`, and `abs`.  You can define additional functions in
ruby.

See {Sass::Script::Functions} for more information.


### New Options

#### `:line_comments`

To aid in debugging, You may set the `:line_comments` option to
`true`. This will cause the sass engine to insert a comment before
each selector saying where that selector was defined in your sass
code.

#### `:template_location`

The {Sass::Plugin} `:template_location` option now accepts a hash of
sass paths to corresponding css paths. Please be aware that it is
possible to import sass files between these separate locations -- they
are not isolated from each other.

### Miscellaneous Features

#### `@debug` Directive

The `@debug` directive accepts a SassScript expression and emits the
value of that expression to the terminal (stderr).

Example:

    @debug 1px + 2px

During compilation the following will be printed:

    Line 1 DEBUG: 3px

#### Ruby 1.9 Support

Sass now fully supports Ruby 1.9.1.

#### Sass Cache

By default, Sass caches compiled templates and
[partials](SASS_REFERENCE.md.html#partials).  This dramatically speeds
up re-compilation of large collections of Sass files, and works best
if the Sass templates are split up into separate files that are all
[`@import`](SASS_REFERENCE.md.html#import)ed into one large file.

Without a framework, Sass puts the cached templates in the
`.sass-cache` directory.  In Rails and Merb, they go in
`tmp/sass-cache`.  The directory can be customized with the
[`:cache_location`](#cache_location-option) option.  If you don't want
Sass to use caching at all, set the [`:cache`](#cache-option) option
to `false`.
