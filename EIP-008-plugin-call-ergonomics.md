#  Name of proposal

Plugin Call Ergonomics

## Purpose

Evaluate possible improvements to the ergonomics of calling plugin functions.

Some notes for discussion:

1) Instead of the plugin function selection being stringly-typed (eww), make it typed using a mechanism appropriate to each language (e.g. enums)
2) Some languages have ways of hoisting code into the environment that would make calling plugin functions feel much more natural than having to go through the `plugin.call` wrapper.
3) ???

## Solution

TBD

## Considerations

- The use of Extism plugins should feel idiomatic in each language. Don't repeat the mistake of OTel, which makes it feel like you're using a Java API in every supported language.
