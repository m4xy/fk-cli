# fk-cli

Reading a [fckt](https://github.com/m4xy/fckt) program's command line: what it takes is a block, each
part on a line, and its help is written from that.

```kotlin
import uk.m4xy.cli.*
import uk.m4xy.cli.Arguments.*

type Search(pattern: String, files: List<String>, ignoreCase: Boolean, context: Int)

search = Command("grep", "Searches files for lines that match a pattern.", Arguments {
  ignoreCase by flag("ignore-case", "match letters in either case", short = Some('i'))
  context by optionOr("context", "N", "lines to show around each match", "0", short = Some('C')).as(Arguments.int)
  pattern by argument("PATTERN", "what to look for")
  files by arguments("FILE", "the files to search")
  Search(pattern, files, ignoreCase, context)
})

@Console @Process main(args: List<String>) = Cli.run(search, args) { s -> ... }
```

```
$ grep --help
usage: grep [options] PATTERN FILE...

Searches files for lines that match a pattern.

arguments:
  PATTERN            what to look for
  FILE...            the files to search

options:
  -i, --ignore-case  match letters in either case
  -C, --context N    lines to show around each match (otherwise 0)
  -h, --help         show this help

$ grep --colour -C many
grep: there's no option --colour
grep: expected a whole number, not 'many'
grep: PATTERN is missing
usage: grep [options] PATTERN FILE...
try grep --help
```

A [kiln](https://github.com/m4xy/kiln) project uses it as `library("uk.m4xy:fk-cli:0.1.0")`.

## What a command takes

`Arguments` is an applicative, as fckt's `Validated` is: a block's lines don't depend on each other,
so every one is read, and every problem is found, not just the first.

| | |
|---|---|
| `flag(long, help, short)` | `--name`: whether it's there |
| `option(long, valueName, help, short)` | `--name VALUE`: its value, if it's there |
| `optionOr(long, valueName, help, otherwise, short)` | `--name VALUE`, or what it otherwise is |
| `required(long, valueName, help, short)` | `--name VALUE`, which must be there |
| `repeated(long, valueName, help, short)` | `--name VALUE`, any number of times |
| `argument(name, help)` | the next value by position, which must be there |
| `optionalArgument(name, help)` | the next value by position, if there's one |
| `arguments(name, help)` | the rest of the values by position |
| `arg.as(read)` | the value read as something else, as `Arguments.int` reads a whole number |

Options come anywhere, as `--name VALUE`, `--name=VALUE`, `-n VALUE`, `-nVALUE`, or flags together,
`-iv`; `--` ends them. `--help` and `-h` ask for the help.

## Running

`Cli.read(command, args)` gives an `Outcome`: `Read(value)`, `HelpAsked(help)`, or `Wrong(problems,
usage)`. `Cli.run(command, args) { value -> ... }` runs the program with the value, writes the help
when it's asked for, and, when something's wrong, writes each problem and the usage and ends with exit
code 2. `Cli.help` and `Cli.usage` write them for a command.
