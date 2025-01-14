# WANIX

Experimental, local-first, web-native, Unix-like operating and development environment

[🎬 Demo from Mozilla Rise 25](https://www.youtube.com/watch?v=KJcd9IckJj8)

## Features

* Run and create command line, TUI, and web apps in the environment itself
* Go compiler that runs in-browser, capable of cross-compiling to native platforms
* Pluggable filesystem API for custom filesystem behavior like FUSE/Plan9
* Unix-like shell that can be live-edited/recompiled or replaced entirely
* Built-in [micro](https://github.com/zyedidia/micro) editor, similar to nano
* Supports TypeScript and JSX in web applications without extra tooling
* Bootstraps entire system with a single JS file and include
* ...lots more

## Getting started

Until a bundled release or live demo is online, you'll need to run Wanix locally as a developer, which means you'll need [Go installed](https://go.dev/doc/install) on your system. Then you can run:

```
make dev
```

This will build everything and run a dev server to access a Wanix environment on localhost in your browser. Here are a few things you can do in the environment in this early state:

### See available built-in commands

A number of common Unix command are available in the shell, which you can see by running `help`. Currently this does not list user or system commands found in `/cmd` or `/sys/cmd`, but you can list those using `ls`. Notable system commands include `build` and `micro`.

### Work with the filesystem

Use common Unix commands (`cd`, `ls`, `pwd`, ...) to navigate around the filesystem. The root filesystem is [indexedfs](internal/indexedfs), which stores files in IndexedDB. However, by implementing the [filesystem API](https://github.com/tractordev/toolkit-go/tree/main/engine/fs) you can create new filesystems to mount. For example, `/sys/dev` is mounted when run with the dev server using [httpfs](internal/httpfs) to expose the project repository from the host. 

The Wanix filesystem layout is different than Unix/Linux, so here are some relevant paths used by Wanix:

 * `/app` - For user applications.
 * `/cmd` - For user commands.
 * `/sys` - For Wanix system paths:
   * `/sys/app` - For system applications.
   * `/sys/cmd` - For system commands.
   * `/sys/bin` - Cache for wasm binaries.
   * `/sys/tmp` - For temporary files, held in-memory.
   * `/sys/dev` - Read-only mount of project repo from host dev server.

### Create a shell script command

You can create commands by putting them in the `/cmd` directory. Commands can be shell scripts, wasm binaries, or buildable source directories. Shell scripts ending in `.sh` can contain commands that would run on the shell. For example, you can use `micro` to create a script to `echo` "Hello world":

```
micro /cmd/hello.sh
```

Add this line to the newly created file in micro:

```
echo Hello world
```

Use `ctrl-s` to save and `ctrl-q` to quit. Now you can run `hello` in the shell.

### Create a command from source directory

You can use `build` to compile a wasm binary from any directory containing a Go main package, then move or copy it into `/cmd`. However, you can also just create a directory in `/cmd` with Go source and use it like a command. Wanix will compile and run it for you. Make a new directory under `/cmd` with `mkdir`:

```
mkdir /cmd/gohello
```

Use `micro` to create and edit a `main.go` file:

```
micro /cmd/gohello/main.go
```

Then write a small Go program:

```go
package main

import "fmt"

func main() {
        fmt.Println("Hello world from Go")
}

```

Use `ctrl-s` then `ctrl-q` to save and quit. Now if you run `gohello`, the program will be compiled and run. If you run it again, it will run the cached binary. If you modify the source and run the command again, it *should* recompile and use an updated binary, but there is currently [an issue](https://github.com/tractordev/wanix/issues/66) waiting for you to implement this.

### Create a web application

Wanix applications are simply web applications under `/app`. If you create a directory like `/app/webapp` with an `index.html`, you can then open the application with `open webapp`.

This will load `/app/webapp/index.html` into a full page iframe. You can get back to and toggle the console in visor mode with Control + the `~` key. Changes made to any file in `/app/webapp` will cause the frame to reload. 

Any TypeScript source files ending in `.ts` will be converted to JavaScript when loaded in this frame. The same for any files using JSX ending in `.jsx` or `.tsx`.

### Build a 3rd party Go program for Wanix

Wanix will eventually [support any WASI program](https://github.com/tractordev/wanix/issues/67), but for now it runs Go programs compiled with `GOOS=js` and `GOARCH=wasm`. If it compiles without a problem, it should be able to run in Wanix. However, larger programs may have dependencies that won't build for this target without some modification. You can see the changes we made to `micro` under [external/micro](external/micro), but it will really be different for every project. For now, here is an example program compiled outside Wanix brought in via the `/sys/dev` mount.

We use the `local` directory in checkouts of this repo for local changes not intended to be contributed. This is a good place to build a program that will be easy to copy into Wanix. For example, you can create `./local/example` with a `main.go`:

```go
package main

import "fmt"

func main() {
        fmt.Println("Hello world from a Go program made outside Wanix")
}

```

Build it with js/wasm target, *outside* Wanix:

```
GOOS=js GOARCH=wasm go build -o example.wasm
```

Now from *inside* Wanix, you can copy the resulting file into `/cmd` from `/sys/dev`:

```
cp /sys/dev/local/example/example.wasm /cmd/example.wasm
```

You should now be able to run `example`.

## Contributing

We are currently developing a roadmap to convey the direction we're exploring with Wanix, but this is an open and modular project that you can take and experiment with for your own purposes. If you're interested in how it works, take a look at our quick [overview](doc/overview.md) and then check out the source. Most of Wanix exists in the `kernel` directory, which is a good place to start. The `build` and `shell` directories are where you'll find the source for those two components.

Take a look at our [issues](https://github.com/tractordev/wanix/issues) to see how you can help out. You can also ask questions and participate in [discussions](https://github.com/tractordev/wanix/discussions), however right now most discussion takes place in our [Discord](https://discord.gg/nbrwNXVvVa).

## License

MIT