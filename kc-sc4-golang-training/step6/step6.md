# Command line arguments

Create a new directory, initialize a new module for our program:

```bash
mkdir -p $HOME/go/src/command-line
cd $HOME/go/src/command-line
go mod init training/basics/command-line
```{{exec}}

We are starting this tutorial with an empty file and we don't reuse the code from [the previous tutorial](../02-environment-variable/README.md).

> From now on, we will use `main.go` as our main program file, our unique entrypoint. So create a new file named `main.go`.

To read command line argument, the native [flag package](https://pkg.go.dev/flag) can help.

Let's say we want to create a program that read the user name from the command line arguments, and print it on the console.

First, you need to declare a variable to host the value. In our case, the name is a string:

```go
var username string
```{{}}

> This `var`{{}} statement can be placed inside or outside the `main`{{}} function, it's up to your preference. If the declaration is inside the function, the variable will be available only inside this function. In the other case, the variable will be available in all files from the same `package`{{}}.

Then, you can use the [`flag.StringVar function`](https://pkg.go.dev/flag#StringVar) to tell the module your are expecting a command line argument:

```go
flag.Stringvar(&username, "firstname", "", "tell us what is your first name")
```{{}}

You pass the argument name to read (`"firstname"`{{}} here), a default value (`""`{{}}) and an usage message (more details on this message later).

You also pass a **pointer** to your `username`{{}} variable. When the `flag`{{}} module will read the command line values, it will populate this variable reference with the read value.

> While the `var`{{}} statement can go outside of the `main`{{}} function, all non-declaration statements should be declared inside a function body. So the `Stringvar`{{}} call should be placed inside the `main`{{}} function body.

Next, you can print the value using your variable:

```go
fmt.Printf("Hello %v !\n", username)
```{{}}

And try the program:

```bash
go run main.go -firstname Romain
# Hello  !
```{{exec}}

> It works... ! Oh no... it does not work at all. What happened ?

We instruct `flag`{{}} what command arguments to expect, but we also need to ask **explicity** to read and parse the command line string.

Add the `flag.Parse()`{{}} call before your print function:

```diff
diff --git a/src/command-line/main.go b/src/command-line/main.go
index 387d7fb..ff12562 100644
@@ -9,5 +9,7 @@ func main() {
        var username string
        flag.StringVar(&username, "firstname", "", "tell us what is your name")

+       flag.Parse()
+
        fmt.Printf("Hello %v !\n", username)
 }
````{{}}

Let's try again:

```bash
go run main.go -firstname Romain
# Hello Romain !
```{{exec}}

As you can see, our command-line argument value is now available inside our Go program. Good job !

There is no way to tell the `firstname`{{}} argument is mandatory or optionnal. If you want this functionnality, you should implement it yourself, for example like this:

```go
if len(username) == 0 {
        fmt.Println("firstname argument is required")
        os.Exit(1)
}
```{{}}

Here is the whole code so far:

```go
package main

import (
        "flag"
        "fmt"
        "os"
)

func main() {
        var username string
        flag.StringVar(&username, "firstname", "", "tell us what is your name")

        flag.Parse()

        if len(username) == 0 {
                fmt.Println("firstname argument is required")
                os.Exit(1)
        }

        fmt.Printf("Hello %v !\n", username)
}
```

## CLI Syntax

The `flag`{{}} package allows command-line arguments with one or two hyphens signs (`-`). This means `go run main.go -firstname xx`{{}} and `go run main.go --firstname xx`{{}} are equivalent.

It also allows the use of the optionnal equal sign (`=`):

```bash
go run main.go -firstname xx
go run main.go -firstname=xx
```{{exec}}

More details about the syntax are listed on the [official documentation chapter](https://pkg.go.dev/flag#hdr-Command_line_flag_syntax).

## Value types

With the flag package, you can read string values, but also integer values, durations, and booleans.

For example, let's ask the user a few (GDPR-compliant) informations:

```bash
go run order-pizza.go --firstname Romain --size large --delivery-time 19h45m --cheese-variant-count 3 --with-tomatoes=false
```{{}}

Here is an extract code on how to read such kind of arguments types:

```go
// read string value (same as above)
var username string
flag.StringVar(&username, "firstname", "", "who order the pizza ?")

// read an int value
// try to run the program without "--cheese-variant-count" arg and see what happens
// try to run the program with "--cheese-variant-count=6" or "-cheese-variant-count 6"
var cheeseVariants int
flag.IntVar(&cheeseVariants, "cheese-variant-count", 2, "how many flavours of cheese do you want?")

// read a boolean value
// without "--with-tomatoes", the default value is true
// please note: for bool argument, the equal sign (=) is MANDATORY
// try to run "--with-tomatoes=false" and "--with-tomatoes false" and see what happens
// in the second case:
//   - the false value is not read
//   - all arguments after the "false" keyword are NOT read
var withTomatoes bool
flag.BoolVar(&withTomatoes, "with-tomatoes", true, "do you want tomatoes on your pizza ?")

// read a Duration value
// Duration must have a specific format to be correclty parsed
// try with "--delivery-time 20h10m" and with ""--delivery-time 20h10" and see what happens
var deliveryTime time.Duration
defaultDeliveryTime, _ := time.ParseDuration("19h30m")
flag.DurationVar(&deliveryTime, "delivery-time", defaultDeliveryTime, "At what time do you want us to deliver your pizza ?")
```

You can also use the `flag.Var`{{}} method to convert the argument value into any Go type you want. In our example, we convert the `--size`{{}} argument into an enum-like value, with only a limited list of values allowed for this argument.

As an example, the whole code of the amazing Pizza Order program is avalaible inside the [order-pizza.go file](./order-pizza.go) in this directory.

## Usage

As seen above, you can specify an usage string for each flag. This usage message may be displayed in various cases:

- if something goes wrong, for example:
    * if you specify an unsupported command line argument when you execute your binary (ex: `go run main.go -invalid-option`{{exec}})
    * if you specify a value with the wrong type (ex: `go run main.go --int-option="string-value"`{{exec}})
    * if you specify a value with an unparseable value (ex: `go run main.go --duration-option="in one hour"`{{exec}})
- to print the manual of the program, using the `-help`{{}} default flag: `go run main.go -help`{{exec}}

Ok, Great !
What about adding some difficulty by learning how to implement a Go Web Server.  
Let's click on Next.