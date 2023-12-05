# inshpect

`inshpect` a glorified grep, primarily for C++ code. It can be used as a
poor-man's version of [include-what-you-use](https://include-what-you-use.org/).

`inshpect` helps you check for missing license and copyright headers, as well as
for use of names and includes that should or shouldn't be there.

`inshpect` is written in bash and requires:

- [fd](https://github.com/sharkdp/fd), for quickly finding files (with
  .gitignore taken into account as is the default behaviour of fd)
- [ripgrep](https://github.com/BurntSushi/ripgrep), for quickly searching files
- [dasel](https://github.com/TomWright/dasel) version 2, for reading the
  configuration file (which can be any type supported by dasel (currently toml,
  yaml, and json))

# Usage

```shell
inshpect [directory to inspect (default: .)] [configuration file (default: ./inshpect.toml)]
```

If violations are found the program exits with a non-zero exit code. Otherwise
it exits with zero.

# Configuration

`inshpect` is configured using a configuration file written in any language that
dasel can read. See the `inshpect-example.toml` file for an example
configuration. The configuration file should contain the following:

- `extensions`: an array of strings of extensions to be checked.
- `copyright.enable`: when `true` enables the copyright check. It checks that
  the word `copyright` or `Copyright` exists somewhere in the file.
- `license.enable`: when `true` enables the license check. It checks that the
  corresponding pattern exists somewhere in the file.
- `license.pattern` (required when the license check is enabled): regex pattern
  which represents the license text.
- `spdx.enable`: when `true` enables the SPDX license identifier check. It
  checks that the corresponding pattern exists somewhere in the file, preceded
  by `SPDX-License-Identifier`.
- `spdx.pattern` (required when the license check is enabled): regex pattern
  which represents the SPDX license identifier.
- `pragma_once.enable`: when `true` enables the pragma once check. It checks that
  the word `#pragma once` exists in the file.
- `pragma_once.extensions` (required when the pragma once check is enabled):
  extensions to check for `#pragma once`. These extensions should represent
  header files.
- `deprecated_includes.enable`: when `true` enables the deprecated includes
  check. It checks that any pattern given in the patterns does not exist, and
  suggests the replacement otherwise.
- `deprecated_includes.patterns` (required when the deprecated includes check is
  enabled): array of key-value pairs of `pattern` and `replacement`, where
  `pattern` represents the include that should not be used and `replacement` the
  replacement include.
- `deprecated_names.enable`: when `true` enables the deprecated names check. It
  checks that any pattern given in the patterns does not exist, and suggests the
  replacement otherwise.
- `deprecated_names.patterns` (required when the deprecated names check is
  enabled): array of key-value pairs of `pattern` and `replacement`, where
  `pattern` represents the name that should not be used and `replacement` the
  replacement name.
- `disallowed_macros.enable`: when `true` enables the disallowed macros check.
  It checks that any pattern given in the patterns does not exist.
- `disallowed_macros.patterns` (required when the disallowed macros check is
  enabled): array of macros that should not exist.
- `includes.enable`: when `true` enables the includes check. It checks that if
  any pattern given in the patterns exists, the corresponding include should
  also exist. This is a poor-man's include-what-you-use.
- `includes.patterns` (required when the includes check is enabled): array of
  key-value pairs of `pattern` and `include`, where `pattern` represents the
  name that should have the corresponding `include`.

See [pika](https://github.com/pika-org/pika) for an example of slightly more
involved configuration files.

`inshpect` can further be controlled by the following environment variables:

- `INSHPECT_RG`: the name of the ripgrep executable, defaults to `rg`
- `INSHPECT_FD`: the name of the fd executable, defaults to `fd`
- `INSHPECT_DASEL`: the name of the dasel executable, defaults to `dasel`
- `INSHPECT_NUMTHREADS`: the number of threads to use in `ripgrep` and `fd`,
  defaults to `1`
- `INSHPECT_VERBOSE`: when set to `1` (exactly!) prints each pattern that
  `inshpect` checks

# Current limitations

This is a very minimally viable piece of software and is missing a lot of
polish. Maybe it shouldn't have been written in bash. If you find something that
you'd like to see improved, please open an issue or even better a pull request!
Non-exhaustive list of current limitations:

- Almost all configuration options must be present in the config file, there are
  no fallback values. The `pattern` keys can be omitted if the corresponding
  check is disabled.
- Speed! ripgrep and fd are very fast, but especially the includes check can be
  very slow when there are many patterns and can likely be sped up.
- There is no possibilty for new checks to be defined in the config file,
  although one can probably abuse the existing checkers to check for things they
  weren't intended for.
- Files without extensions are not supported.

# History

`inshpect` is a simplified version of the [Boost inspect
tool](https://www.boost.org/doc/libs/1_80_0/tools/inspect/), also used in
modified forms in [HPX](https://github.com/STEllAR-GROUP/hpx) and (previously)
[pika](https://github.com/pika-org/pika), and owes its existence to those tools.
