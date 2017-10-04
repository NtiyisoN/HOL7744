# Introduction to Fn

Fn is a lightweight Docker-based serverless functions platform you can
run on your laptop, server, or cloud.  In this introductory tutorial
we'll walk through installing Fn, develop a function using the Go
programming languages (without installing any Go tools!) and
deploy them to a local Fn server.  We'll also learn about the core Fn
concepts like applications and routes.

So let's get started!

As you make your way through this tutorial, look out for this icon.
![](images/userinput.png) Whenever you see it, it's time for you to
perform an action.


## Installing Fn

In this tutorial we'll work in a purely local development
mode.  However, when deploying functions to a remote Fn server, a Docker
Hub (or other Docker registry) account is required.


### Downloading and Installing Fn

Open a terminal window using the desktop link and type the following:


![](images/userinput.png)
>`curl -LSs https://raw.githubusercontent.com/fnproject/cli/master/install | sh`

Once installed you'll see the Fn version printed out.  You should see
something similar to the following displayed:

```sh
fn version 0.3.89
```

### Starting Fn Server

The final install step is to start the Fn server.  Since Fn runs on
Docker it'll need to be up and running too.

To start Fn you can use the `fn` command line interface (cli).  Type the
following but note that the process will run in the foreground so that
it's easy to stop with Ctrl-C:

![user input](images/userinput.png)
>`fn start`

You should see output similar to:

```sh
time="2017-09-18T14:37:13Z" level=info msg="datastore dialed" datastore=sqlite3 max_idle_connections=256
time="2017-09-18T14:37:13Z" level=info msg="available memory" ram=1655975936
time="2017-09-18T14:37:13Z" level=info msg="Serving Functions API on address `:8080`"

      ______
     / ____/___
    / /_  / __ \
   / __/ / / / /
  /_/   /_/ /_/
```

That's it!  The Fn cli is now installed and an Fn server instance
is up and running.

## Your First Function

Let's start with a very simple "hello world" function written in
[Go](https://golang.org/). Don't worry, you don't need to know Go!  In
fact you don't even need to have Go installed on your development
machine as Fn provides the necessary Go compiler and tools as a Docker
container.  Let's walk through your first function to become familiar
with the process and how Fn supports development.


![user input](images/userinput.png)
>Open a new terminal window using the desktop shortcut.

Before we start developing we need to set the `FN_REGISTRY`
environment variable.  Normally, it's set to your Docker repository and
Docker Hub username.  However in this tutorial we'll work in local
development mode so we can set the `FN_REGISTRY` variable to an invented
 value. Let's use `fndemouser`.

![user input](images/userinput.png)
>`export FN_REGISTRY=fndemouser`


With that out of the way, create a new directory in your home
directory named "hello" and cd into it:

![user input](images/userinput.png)
>`cd ~`
>
>`mkdir hello`
>
>`cd hello`

Copy and paste the Go code below into a file named `func.go` by
opening the gedit editor:

![user input](images/userinput.png)
>`gedit func.go & `

Here's the function source to paste into gedit and then save:

![user input](images/userinput.png)
>```go
>package main
>
>import (
>  "fmt"
>)
>
>func main() {
>  fmt.Println("Hello from Fn!")
>}
>```

This function just prints "Hello from Fn!" to standard output.  It takes
no arguments and returns no results. So it's as simple as possible.  Of
course, you can write functions that accept a number of different types
of arguments and this is explored in other Fn tutorials.


#### Initializing your Function Configuration

Let's use the `fn` CLI to initialize this function's configuration. In
your terminal window type:

![user input](images/userinput.png)
> `fn init`

```sh
Found go, assuming go runtime.
func.yaml created
```

`fn` found your `func.go` file and generated a `func.yaml` file.
 Display the file contents by typing:

![user input](images/userinput.png)
> `cat func.yaml`

```yaml
version: 0.0.1
runtime: go
entrypoint: ./func
build_image: ""
run_image: ""
expects:
  config: []
```

#### Understanding func.yaml

The generated `func.yaml` file contains metadata about your function and
declares a number of properties including:

* the version--automatically starting at 0.0.1
* the name of the runtime/language--which was set
automatically based on the presence of `func.go`
* the name of the function to invoke--in this case `./func` which will
be the name of the compiled Go file.

There are other user specifiable properties but these will suffice for
this example.

### Running Your First Function

With the `hello` directory containing `func.go` and `func.yaml` you've
got everything you need to run the "hello" function.  So let's run it and
observe the output.  Note that the first time you build a
function of a particular language it takes longer as Fn downloads
the necessary Docker images.

![user input](images/userinput.png)
> `fn run`

```sh
Building image fndemouser/hello:0.0.1
Sending build context to Docker daemon  4.096kB
Step 1/8 : FROM funcy/go:dev as build-stage
 ---> 4cccab7fc828
Step 2/8 : WORKDIR /function
 ---> Using cache
 ---> 617fce473c5b
Step 3/8 : ADD . /go/src/func/
 ---> 7d74280c46eb
Step 4/8 : RUN cd /go/src/func/ && go build -o func
 ---> Running in bd63803dffd5
 ---> e528f3d33dde
Removing intermediate container bd63803dffd5
Step 5/8 : FROM funcy/go
 ---> 573e8a7edc05
Step 6/8 : WORKDIR /function
 ---> Using cache
 ---> d8c99b2722e4
Step 7/8 : COPY --from=build-stage /go/src/func/func /function/
 ---> Using cache
 ---> ca329aa33fbe
Step 8/8 : ENTRYPOINT ./func
 ---> Using cache
 ---> e309b9711693
Successfully built e309b9711693
Successfully tagged fndemouser/hello:0.0.1
Hello from Fn!
```

The last line of output should be "Hello from Fn!" that was produced
by the Go statement `fmt.Println("Hello from Fn!")`.

### Understanding fn run

If you have used Docker before the output of `fn run` should look
familiar--it looks like the output you see when running `docker build`
with a Dockerfile.  Of course this is exactly what's happening!  When
you run a function like this Fn is dynamically generating a Dockerfile
for your function, building a container, and then running it.

> __NOTE__: Fn is actually using two images.  The first contains
the language compiler and is used to generate a binary.  The second
image packages only the generated binary and any necessary language
runtime components. Using this strategy, the final function image size
can be kept as small as possible.  Smaller Docker images are naturally
faster to push and pull from a repository which improves overall
performance.

`fn run` is a local operation.  It builds and packages your function
into a container image which resides on your local machine.  As Fn is
built on Docker you can use the `docker` cli to see the local
container image you just generated.

You may have a number of Docker images so use the following command
to see only those created by fndemouser:

![user input](images/userinput.png)
>`docker images | grep fndemouser`

You should see something like:

```sh
fndemouser/hello                               0.0.1               d64b4a1a15b9        2 minutes ago      6.98MB
```

## Deploying Your First Function

When we used `fn run` your function was run in your local environment.
Now let's deploy your function to the Fn server we started previously.
This server could be running in the cloud, in your datacenter, or on
your local machine like we're doing here.

Deploying your function is how you publish your function and make it
accessible to other users and systems.

In your terminal type the following:

![user input](images/userinput.png)
> `fn deploy --app myapp --local`

You should see output similar to:

```sh
bumping version in func file at:  /Users/shaun/hello/func.yaml
Bumped to version 0.0.2
Building image fndemouser/hello:0.0.2
Sending build context to Docker daemon  75.26kB
Step 1/8 : FROM funcy/go:dev as build-stage
 ---> 4cccab7fc828
Step 2/8 : WORKDIR /function
 ---> Using cache
 ---> 617fce473c5b
Step 3/8 : ADD . /go/src/func/
 ---> 83e541ea8cea
Step 4/8 : RUN cd /go/src/func/ && go build -o func
 ---> Running in ebcd32b6b6a4
 ---> 566b8929732c
Removing intermediate container ebcd32b6b6a4
Step 5/8 : FROM funcy/go
 ---> 573e8a7edc05
Step 6/8 : WORKDIR /function
 ---> Using cache
 ---> d8c99b2722e4
Step 7/8 : COPY --from=build-stage /go/src/func/func /function/
 ---> Using cache
 ---> aef557ab859a
Step 8/8 : ENTRYPOINT ./func
 ---> Using cache
 ---> d64b4a1a15b9
Successfully built d64b4a1a15b9
Successfully tagged fndemouser/hello:0.0.2
Updating route /hello using image fndemouser/hello:0.0.2...

```


Functions are grouped into applications so by specifying `--app myapp`
we're implicitly creating the application "myapp" and associating our
function with it.

Specifying `--local` does the deployment to the local server but does
not push the function image to a registry--which would be necessary if
we were deploying to a remote Fn server.

Once again you see output from the underlying Docker build but you also
see Fn related messages like
`Updating route /hello using image fndemouser/hello:0.0.2`. This
let's us know that the function packaged in the image
"fndemouser/hello:0.0.2" has been bound by the Fn server to the route
"/hello".  We'll see how to use the route below.

## Calling Your Deployed Function

There are two ways to call your deployed function.  The first is using
the `fn` cli which makes invoking your function relatively easy.  Type
the following:

![user input](images/userinput.png)
>`fn call myapp /hello`

which results in our familiar message.

```sh
Hello from Fn!
```

Of course this is unchanged from when you ran the function locally.
However when you called "myapp /hello" the fn server looked up the
"myapp" application and then looked for the function bound to the
"/hello" route.

The other way to call your function is via HTTP.  The Fn server
exposes our deployed function at "http://localhost:8080/r/myapp/hello"
using the application and route as path elements.

Use curl to invoke the function:

![user input](images/userinput.png)
>`curl http://localhost:8080/r/myapp/hello`

The result is once again the same.

```sh
Hello from Fn!
```

## Wrapping Up

Congratulations!  In this tutorial you've accomplished a lot.  You've
installed Fn, started up an Fn server, created your first function,
run it locally, and then deployed it where it can be invoked over HTTP.

In the next tutorial you'll learn about Fn's Java FDK (Function
Development Kit) and build and test a function with the FDK's JUnit
support.

**Go to:** [Java FDK Introduction](../JavaFDKIntroduction/README.md)
