# TARDIS Go -> Haxe transpiler

#### Haxe -> JavaScript / ActionScript / Java / C++ / C# / PHP / Neko

[![Build Status](https://travis-ci.org/tardisgo/tardisgo.png?branch=master)](https://travis-ci.org/tardisgo/tardisgo)
[![GoDoc](https://godoc.org/github.com/tardisgo/tardisgo?status.png)](https://godoc.org/github.com/tardisgo/tardisgo)
[![status](https://sourcegraph.com/api/repos/github.com/tardisgo/tardisgo/badges/status.png)](https://sourcegraph.com/github.com/tardisgo/tardisgo)

## Objectives:
The objective of this project is to enable the same [Go](http://golang.org) code to be re-deployed in  as many different execution environments as possible, thus saving development time and effort. 
The long-term vision is to provide a framework that makes it easy to target many languages as part of this project.

The first language targeted is [Haxe](http://haxe.org), because the Haxe compiler generates 7 other languages and is already well-proven for making multi-platform client-side applications, mostly games. 

Planned current use cases: 
- For the Go community: write a library in Go and call it from  existing Haxe, JavaScript, ActionScript, Java, C++, C# or PHP applications. 
- For the Haxe community: provide access to the portable elements of Go's extensive libraries and open-source code base.
- Write a multi-platform client-side application in a mixture of Go and Haxe, using [OpenFL](http://openfl.org) / [Lime](https://github.com/openfl/lime) or [Kha] (http://kha.ktxsoftware.com/) to target a sub-set of: 
Windows,
Mac,
Linux,
iOS,
Android,
BlackBerry,
Tizen,
Emscripten,
HTML5,
webOS,
Flash,
Xbox and PlayStation.

For more background and on-line examples see the links from: http://tardisgo.github.io/

## Project status: 
####  DEMONSTRABLE, EXPERIMENTAL, INCOMPLETE,  UN-OPTIMIZED

> "Premature optimization is the root of all evil (or at least most of it) in programming." - Donald Knuth

All of the core [Go language specification] (http://golang.org/ref/spec) is implemented, including single-threaded goroutines and channels. However the packages "unsafe" and "reflect", which are mentioned in the core specification, are not currently supported. 

Goroutines are implemented as co-operatively scheduled co-routines. Other goroutines are automatically scheduled every time there is a function call or a channel operation, so loops without calls or channel operations will never give up control. The empty function tardisgolib.Gosched() provides a way to give up control without including the full Go runtime.  

Some parts of the Go standard library work, as you can see in the [example TARDIS Go code](http://github.com/tardisgo/tardisgo-samples), but the bulk has not been  tested or implemented yet. If the standard package is not mentioned in the notes below, please assume it does not work. So fmt.Println("Hello world!") will not transpile, instead use the go builtin function: println("Hello world!").  

Some standard Go library packages do not call any runtime C or assembler functions and will probably work OK (though their tests still need to be rewritten and run to validate their correctness), these include:
- errors
- unicode
- unicode/utf8 
- unicode/utf16
- sort
- container/heap
- container/list
- container/ring

Other standard libray packages make limited use of runtime C or assembler functions without using the actual Go "runtime" or "os" packages. These limited runtime functions have been emulated for the following packages (though their tests still need to be rewritten and run to validate their correctness). To use these packages, their corresponding runtime functions need to be included as follows:
```
include ( 
	"bytes" 
	_ "github.com/tardisgo/tardisgo/golibruntime/bytes"
	
	"strings"  // but see issue #19 re JS and Flash
	_ "github.com/tardisgo/tardisgo/golibruntime/strings"
	
	"sync" // partial: only WaitGroup is known to work 
	_ "github.com/tardisgo/tardisgo/golibruntime/sync"
	
	"sync/atomic""
	_ "github.com/tardisgo/tardisgo/golibruntime/sync/atomic"
	
	"math"
	"strconv"  // uses the math package
	_ "github.com/tardisgo/tardisgo/golibruntime/math"
)
```
At present, standard library packages which rely on the Go "runtime", "os", "reflect" or "unsafe" packages are not implemented (although some OSX test code is in the golibruntime tree).

A start has been made on the automated integration with Haxe libraries, but this is currently incomplete see: https://github.com/tardisgo/gohaxelib

TARDIS Go specific runtime functions are available in [tardisgolib](https://github.com/tardisgo/tardisgo/tree/master/tardisgolib):
```
import "github.com/tardisgo/tardisgo/tardisgolib" // runtime functions for TARDIS Go
```

The code is developed on OS X Mavericks‎, using Go 1.2.1 and Haxe 3.1.1. The target platforms tested are Ubuntu 13.10 32-bit & 64-bit, and Windows 7 32-bit. The 64-bit platforms work fine, but compilation to the C# target fails on Win-7; and PHP is flakey, espeecially the 32-bit version (but you probably knew that).

## Installation and use:
 
Dependencies:
```
go get code.google.com/p/go.tools
```

TARDIS Go:
```
go get github.com/tardisgo/tardisgo
```

If tardisgo is not installing and there is a green "build:passing" icon at the top of this page, please e-mail [Elliott](https://github.com/elliott5)!

To translate Go to Haxe, from the directory containing your .go files type the command line: 
```
tardisgo yourfilename.go 
``` 
A single Go.hx file will be created in the tardis subdirectory.

To run your transpiled code you will first need to install [Haxe](http://haxe.org).

Then to run the tardis/Go.hx file generated above, type the command line: 
```
haxe -main tardis.Go --interp
```
... or whatever [Haxe compilation options](http://haxe.org/doc/compiler) you want to use. 
See the [tgoall.sh](https://github.com/tardisgo/tardisgo-samples/blob/master/scripts/tgoall.sh) script for simple examples.

To run cross-target command-line tests as quickly as possible, the "-testall" flag  concurrently runs the Haxe compiler and executes the resulting code for all supported targets (with compiler output suppressed and results appearing in the order they complete):
```
tardisgo -testall myprogram.go
```

PHP specific issues:
* to compile for PHP you currently need to add the haxe compilation option "--php-prefix tgo" to avoid name conflicts
* very long PHP class/file names may cause name resolution problems on some platforms

## Next steps:
Please go to http://github.com/tardisgo/tardisgo-samples for example Go code modified to work with tardisgo.

For public help or discussion please go to the [Google Group](https://groups.google.com/d/forum/tardisgo); or feel free to e-mail [Elliott](https://github.com/elliott5) direct to discuss any issues if you prefer.

The documentation is sparse at present, if there is some aspect of the system that you want to know more about, please let [Elliott](https://github.com/elliott5) know and he will prioritise that area.

If you transpile your own code using TARDIS Go, please report the bugs that you find here, so that they can be fixed.

## Future plans:

Development priorities:
- For all Go standard libraries, report testing and implementation status
- Improve integration with Haxe code and libraries, automating as far as possible
- Improve currently poor execution speeds and update benchmarking results
- Research and publish the best methods to use TARDIS Go to create multi-platform client-side applications
- Improve debug and profiling capabilities
- Add command line flags to control options
- Publish more explanation and documentation
- Move more of the runtime into Go (rather than Haxe) to make it more portable 
- Implement other target languages

If you would like to get involved in helping the project to advance, that would be wonderful. However, please contact [Elliott](https://github.com/elliott5) or discuss your plans in the [tardisgo](https://groups.google.com/d/forum/tardisgo) forum before writing any substantial amounts of code to avoid any conflicts. 

## License:

MIT license, please see the license file.
