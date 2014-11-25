# `jsonwriter`: functions module for [shellfire]

This module provides a simple framework for writing JSON with a [shellfire] application. It consists of a number of helper methods for correctly generating suitably escaped JSON. The API for this module is not yet stable.

An example user is the [github api] module used by [swaddle].

## Overview

For example, to create a standalone JSON object, suitable for a lot of REST APIs:-

```bash
jsonwriter_object \
	string tag_name "tag" \
	string name "who-are-you" \
	string body "my body" \
	boolean draft true \
	boolean prerelease false \
>/path/to/file.json
```

Alternatively, it is possible to use the functions in templates\*:-
```bash

printf '%s' "{
	$(jsonwriter_string 'tag_name'):$(jsonwriter_string 'tag')
	$(jsonwriter_string 'name'):$(jsonwriter_string 'who-are-you')
	$(jsonwriter_string 'body'):$(jsonwriter_string 'my-body')
}"
```
This approach is useful when building JSON graphs recursively. [jsonwriter]'s primary use case is for REST API calls, which rarely require large, complex or highly nested JSON. (In fact, many REST APIs use such simple objects that the traditional `application/x-www-form-urlencoded` content type would be far more suitable and browser friendly (and arguably more byte-efficient), but many API authors haven't yet seen enough winters).

Strictly speaking, the object's key should always be passed through the `jsonwriter_string` function, but, in practice, nearly all object keys are well-known values that don't contain anything that needs escaping. Of course, if using untrusted input…

\* _Note: We don't use `echo` as, by default, it responds to escape sequences. The options to disable this vary by shell and are inherently non-portable._

## Importing

To import this module, add a git submodule to your repository. From the root of your git repository in the terminal, type:-
```bash
mkdir -p lib/shellfire
cd lib/shellfire
git submodule add "https://github.com/shellfire-dev/jsonwriter.git"
cd -
git submodule init --update
```

You may need to change the url `https://github.com/shellfire-dev/jsonwriter.git` above if using a fork.

You will also need to add paths - include the module [paths.d].


## Namespace `jsonwriter`

This namespace contains the core functionality needed to access the API.

### To use in code

If calling from another [shellfire] module, add to your shell code the line
```bash
core_usesIn jsonwriter
```
in the global scope (ie outside of any functions). A good convention is to put it above any function that depends on functions in this module. If using it directly in a program, put this line _inside_ the `_program()` function:-

```bash
_program()
{
	core_usesIn jsonwriter
	…
}
```

### Functions

***
#### `jsonwriter_object`

|Parameter|Value|Optional|
|---------|-----|--------|
|`…`|Zero or more triplets of `fieldName`, `fieldType` and `fieldValue`|_Yes_|

Writes a JSON object from triplets to standard out.

The values of `fieldType` may be:-

|Value|Description|
|-----|-----------|
|`null`|Output a JSON `null`. `fieldValue` must be null.|
|`boolean`|Output a JSON `true` or `false` from a boolean `fieldValue` (`yes`, `no`, `on`, `off`, `true`, `false`, etc). No validation is currently performed.|
|`number`|Output a JSON number from `fieldValue`. No validation is currently performed.|
|`string`|Output a JSON string, properly escaped, from `fieldValue`. Strings are assumedto be UTF-8 encoded.|
|`objectCallback`|`fieldValue` is the name of a function to callback which will output name-value pairs (commas are auto-generated). Must return 0 if there are no more pairs, or 1 if there are (ie like the `read` built-in).|
|`listCallback`|`fieldValue` is the name of a function to callback which will output values (commas are auto-generated). Must return 0 if there are no more values, or 1 if there are (ie like the `read` built-in).|

***
#### `jsonwriter_list`

|Parameter|Value|Optional|
|---------|-----|--------|
|`…`|Zero or more triplets of `fieldName`, `fieldType` and `fieldValue`|_Yes_|

Writes a JSON list from triplets to standard out.

The values of `fieldType` may be:-

|Value|Description|
|-----|-----------|
|`null`|Output a JSON `null`. `fieldValue` must be null.|
|`boolean`|Output a JSON `true` or `false` from a boolean `fieldValue` (`yes`, `no`, `on`, `off`, `true`, `false`, etc) or null (full list, see `jsonwriter_boolean()`). No validation is currently performed.|
|`number`|Output a JSON number from `fieldValue`. No validation is currently performed.|
|`string`|Output a JSON string, properly escaped, from `fieldValue`. Strings are assumedto be UTF-8 encoded.|
|`objectCallback`|`fieldValue` is the name of a function to callback which will output name-value pairs (commas are auto-generated). Must return 0 if there are no more pairs, or 1 if there are (ie like the `read` built-in).|
|`listCallback`|`fieldValue` is the name of a function to callback which will output values (commas are auto-generated). Must return 0 if there are no more values, or 1 if there are (ie like the `read` built-in).|

***
#### `jsonwriter_null`

|Parameter|Value|Optional|
|---------|-----|--------|
|`null`|Should be `null` but currently ignored|_Yes_|

Writer a JSON null value to standard out. Useful for embedding in heredoc templates.

***
#### `jsonwriter_boolean`

|Parameter|Value|Optional|
|---------|-----|--------|
|`boolean`|`yes`, `no`, `on`, `off`, `true`, `false`, `T`, `F`, `Y`, `N`, `0`, `1` in lower case, upper case and title case.|_No_|

Writes a JSON boolean value to standard out. Useful for embedding in heredoc templates. As an extension, if `boolean` is empty, omitted (unset), `2` or `null` (lower case, upper case or title case), then a JSON null value is written.

***
#### `jsonwriter_number`

|Parameter|Value|Optional|
|---------|-----|--------|
|`boolean`|`yes`, `no`, `on`, `off`, `true`, `false`, `T`, `F`, `Y`, `N`, `0`, `1` in lower case, upper case and title case.|_No_|

Writes a JSON boolean value to standard out. Useful for embedding in heredoc templates.

***
#### `jsonwriter_string`

|Parameter|Value|Optional|
|---------|-----|--------|
|`boolean`|UTF-8 encoded string.|_No_|

Writes a JSON string value to standard out, correctly escaped and with quotes. Useful for embedding in heredoc templates. Suitable for object keys, object values and lists, as well as root values. It is impossible to write a NUL in a string using this function, as the shell does not support ASCII NUL values (unless using zsh). This is unlikely to be a problem in practice.

[swaddle]: https://github.com/raphaelcohn/swaddle "Swaddle homepage"
[shellfire]: https://github.com/shellfire-dev "shellfire homepage"
[core]: https://github.com/shellfire-dev/core "shellfire core module homepage"
[paths.d]: https://github.com/shellfire-dev/paths.d "paths.d shellfire module homepage"
[github api]: https://github.com/shellfire-dev/github "github shellfire module homepage"
[jsonwriter]: https://github.com/shellfire-dev/jsonwriter "jsonwriter shellfire module homepage"
[jsonreader]: https://github.com/shellfire-dev/jsonreader "jsonreader shellfire module homepage"
[urlencode]: https://github.com/shellfire-dev/urlencode "urlencode shellfire module homepage"
[unicode]: https://github.com/shellfire-dev/unicode "unicode shellfire module homepage"
[version]: https://github.com/shellfire-dev/version "version shellfire module homepage"

