# Bazel build rules for Flex

This Bazel ruleset allows [Flex] to be integrated into a Bazel build. It can
be used to generate lexical analyzers ("lexers") in C or C++.

API reference: [docs/rules_flex.md](docs/rules_flex.md)

[Flex]: https://github.com/westes/flex

## Setup (workspace)

### As a workspace dependency

```python
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

http_archive(
    name = "rules_m4",
    sha256 = "10ce41f150ccfbfddc9d2394ee680eb984dc8a3dfea613afd013cfb22ea7445c",
    urls = ["https://github.com/jmillikin/rules_m4/releases/download/v0.2.3/rules_m4-v0.2.3.tar.xz"],
)

load("@rules_m4//m4:m4.bzl", "m4_register_toolchains")

m4_register_toolchains(version = "1.4.18")

http_archive(
    name = "rules_flex",
    # Obtain the package checksum from the release page:
    # https://github.com/jmillikin/rules_flex/releases/tag/v0.2
    sha256 = "",
    urls = ["https://github.com/jmillikin/rules_flex/releases/download/v0.2/rules_flex-v0.2.tar.xz"],
)

load("@rules_flex//flex:flex.bzl", "flex_register_toolchains")

flex_register_toolchains(version = "2.6.4)
```

## Examples

Integrating Flex into a C/C++ dependency graph:

```python
load("@rules_flex//flex:flex.bzl", "flex_cc_library")

flex_cc_library(
    name = "hello",
    src = "hello.l",
)

cc_binary(
    name = "hello_bin",
    deps = [":hello"],
)
```

Generating `.c` / `.h` / `.cc` source files (not as a `CcInfo`):

```python
load("@rules_flex//flex:flex.bzl", "flex")

flex(
    name = "hello_bin_srcs",
    src = "hello.l",
)

cc_binary(
    name = "hello_bin",
    srcs = [":hello_bin_srcs"],
)
```

Running Flex in a `genrule`:

```python
genrule(
    name = "hello_gen",
    srcs = ["hello.l"],
    outs = ["hello_gen.c"],
    cmd = "M4=$(M4) $(FLEX) --outfile=$@ $<",
    toolchains = [
        "@rules_flex//flex:current_flex_toolchain",
        "@rules_m4//m4:current_m4_toolchain",
    ],
)
```

Writing a custom rule that depends on Flex as a toolchain:

```python
load("@rules_flex//flex:flex.bzl", "FLEX_TOOLCHAIN_TYPE", "flex_toolchain")
load("@rules_m4//m4:m4.bzl", "M4_TOOLCHAIN_TYPE")

def _my_rule(ctx):
    flex = flex_toolchain(ctx)
    ctx.actions.run(
        executable = flex.flex_tool,
        env = flex.flex_env,
        # ...
    )

my_rule = rule(
    implementation = _my_rule,
    toolchains = [
        FLEX_TOOLCHAIN_TYPE,
        M4_TOOLCHAIN_TYPE,
    ],
)
```
