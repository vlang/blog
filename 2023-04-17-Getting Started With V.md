![getting-started-with-v](https://user-images.githubusercontent.com/46458389/232466829-f06fe796-7903-49c4-95d9-88820724a02b.jpg)

# Introduction

V is a simple, fast, safe, compiled general purpose programming language for developing maintainable software. There are several compelling reasons for using V, which you can learn more about here: [The V Programming Language](https://vlang.io/)

In this post, we focus on getting new users up to speed so that they can start working with V right away.

# Installing (and Updating) V

Since V is in active development, there will be a lot of breaking changes often. Therefore, it is recommended that you install V from the source repository at github.com/vlang/v.

## Installing V from Source

It is recommended to go through the steps [listed here](https://github.com/vlang/v#installing-v-from-source). Briefly, these are the steps you need to follow:

```bash
cd to/some/appropriate/location
git clone https://github.com/vlang/v # Use --depth 1 if you have bandwidth or space constraints
cd v
make # On Windows, run make.bat in a cmd shell or PowerShell instance
```

V compiles itself really fast and requires no dependencies and libraries for self compilation. It ships with TCC by default but it is recommended to install an established [C compiler](https://github.com/vlang/v#c-compiler) so that you can make optimised builds.

## Weekly Release

V makes weekly releases (besides the normal semantic version update releases) if you do not prefer compiling V yourself. You can find the releases here: [Releases](https://github.com/vlang/v/releases).

## Symlinking

The V executable generated (or obtained) is not automatically added to your `PATH`. V provides a convenient solution to this.

```bash
# On Linux, MacOS, and other similar systems
sudo ./v symlink

# On Windows, open an Administrator shell and run the following.
# NOTE: This is the ONLY command that needs to be run from an Administrator shell.
.\v.exe symlink

# Test your installation by running:
v version
```

If this did not work correctly, refer to the [official instructions](https://github.com/vlang/v#symlinking) for more information.

## Updating V

Similar to `rustup`, V supports the following command:

```bash
v up
```

This command updates the V compiler to the latest version available from the main [Github repository](https://github.com/vlang/v).

# Learning V

Syntactically, V is similar to Go. Therefore Go devs can feel at home when they write V code. Knowledge of Go is not necessary; the language is very simple and the fundamental concepts can be learned over a weekend.

The official documentation for the language is written in a single markdown document: [V Documentation](https://github.com/vlang/v/blob/master/doc/docs.md).

The documentation for all the modules included in the V standard library (vlib) is available here: [vlib Documentation](https://modules.vlang.io/). You are encouraged to use the _search bar_ to find the functions and structs you need for your project.

For a taste of V, here is a truncated example of TOML file parsing. The sample TOML file is available here: [TOML Example](https://github.com/toml-lang/toml/blob/3b11f6921da7b6f5db37af039aa021fee450c091/README.md#Example)

```v
import toml
import toml.to

// Multiline string
const toml_text = '# This is a TOML document.

title = "TOML Example"

[owner]
name = "Tom Preston-Werner"
dob = 1979-05-27T07:32:00-08:00 # First class dates

[database]
server = "192.168.1.1"
ports = [ 8000, 8001, 8002 ]
connection_max = 5000
enabled = true

[servers]

  # Indentation (tabs and/or spaces) is allowed but not required
  [servers.alpha]
  ip = "10.0.0.1"
  dc = "eqdc10"

  [servers.beta]
  ip = "10.0.0.2"
  dc = "eqdc10"

[clients]
data = [ ["gamma", "delta"], [1, 2] ]

# Line breaks are OK when inside arrays
hosts = [
  "alpha",
  "omega"
]
'

fn main() {
	doc := toml.parse_text(toml_text)!
	title := doc.value('title').string()
	println('title: "${title}"')
	ip := doc.value('servers.alpha.ip').string()
	println('Server IP: "${ip}"')

	toml_json := to.json(doc)
	println(toml_json)
}

```

# Setting up your Development Environment

## Code Editor

As of 2023, [Visual Studio Code](https://code.visualstudio.com/) is still a popular choice in the community. However, the [IntelliJ V Plugin](https://intellij-v.github.io/) for JetBrains IDEs is also well-favoured. For help with other editors, please join our [Discord Server](https://discord.gg/vlang).

## Running a V Program

Your project should have at least one `main` function. Then you can compile and run your code in one of two ways:

1. Directly run the program with `v run file.v` or `v run directory`. You can even do `v run .` to run the only `main` function in the directory. Note that a `module main` needs to be present at the top of your entry file when you want to execute a directory.
2. Compile the program with `v file.v` (optionally as `v -prod file.v`) and then run the generated executable.

## Miscellaneous

You need `git` to clone and update V. Make sure that it is installed on your machine.

Additionally, use `v new` and `v init` as necessary, and use `v fmt -w /path/` to automatically format your source code. Consult `v help init` and `v help fmt` for more information.

Bonus: look at all the tools accessible using the `v` front-end here: [cmd/tools](https://github.com/vlang/v/tree/4a22d4a65dd4e7f28eceb9243abb06dfeeab4d26/cmd/tools).

# Getting Help and Reaching Out

It is recommended that you make frequent use of `v help`, consult the [language documentation](https://github.com/vlang/v/blob/master/doc/docs.md) and the [modules documentation](https://modules.vlang.io/). If you want to get specific help, reach out to us directly, or just hang out with the V community members, join our [Discord Server](https://discord.gg/vlang). Use the appropriate channels for your needs.

While developing in V, you may encounter bugs. Please report them in the `#bugs` channel, and file [Github issues](https://github.com/vlang/v/issues) about them with `v bug file.v`.
