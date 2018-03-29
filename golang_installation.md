# Prerequisites

### Install go
```terminal
brew install go
```

### Add go relative paths
```.zshrc
export GOPATH=$HOME/golang
export GOROOT=/usr/local/opt/go/libexec
export PATH=$PATH:$GOPATH/bin
export PATH=$PATH:$GOROOT/bin
```
see more here https://golang.org/doc/code.html

### Install vim-go
https://github.com/fatih/vim-go/

```.vimrc
Plug 'fatih/vim-go'
```

```vimTerminal
:PlugInstall
:GoInstallBinaries
:help vim-go
```
see more here https://robots.thoughtbot.com/writing-go-in-vim

### Write some code
so lets write some code, remember to add the relative paths to the golang workspace see above,
```
mkdir -p $GOPATH/src/hello
vim $GOPATH/src/hello/hello.go
```

```go
package main

import "fmt"

func main() {
fmt.Printf("Hello, world.\n")
}
```

if you are using vim, you will notice that default configuration for vim-go is to run :GoFmt when a file is saved.

```go
package main

import "fmt"

func main() {
	fmt.Printf("Hello, world.\n")
}
```

at this point you can check that your code builds by running 

```
:GoBuild
```

if you are within the workspace you can build with `go build .`
To produce an output file we can run `go install .` which places the package object inside the bin directory
at this point you may ask what's the diff between build and install, see more regarding workspaces https://thenewstack.io/understanding-golang-packages/

