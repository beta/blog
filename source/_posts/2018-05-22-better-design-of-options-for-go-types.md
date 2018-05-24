title: Better Design of Options for Go Types
date: 2018-05-22 16:24:53
tags:
- Go
- Programming
---
Options specify the user-controllable values and switches of types. For example, a window has its title and size, a logger has its verbosity, and an HTTP server has its address and port to bind onto.

Imagine we are designing a logger which has a dynamic verbosity, represented by an integer. To allow user to modify the verbosity, the simplest way is to make the verbosity field accessible.

```go
type Logger struct {
    Verbosity int
}
```

In this way the user can modify the verbosity as will.

```go
logger.Verbosity = 1
```

This is the easiest and most flexible way, but also the most dangerous and clumsy way. You lose all control of the value set by the user, and you will not be notified if the option is changed.

A better way to provide the options is to use setter functions, which is really popular in Java.

```go
type Logger struct {
    verbosity int
}

func (l *Logger) Verbosity(v int) {
    // Check the value.
    l.verbosity = v
    // Do something else.
}
```

A setter will make you able to hide the `verbosity` field from the user, and allow you to perform check or any other action that is related to the option. However, as the amount of options grows, too many setters will be created for the type which will easily mess up the code. Also, you can only modify one option at a time, so you often need several lines of code that deals with the options. (Fluent interface can make it easier, but that requires extra work.)

- - -

Recently I came into a [blog post](https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html) by Rob Pike presenting his new design of options for Go types. He uses one single `Option` method as the interface, and self-referential functions to update the options.

```go
type Logger struct {
    verbosity int
}

// option is a self-referential function which updates the option
// and returns another function to restore the change.
type option func(l *Logger) option

// Option executes the option functions to update the options,
// and returns the revert function of the last option.
func (l *Logger) Option(opts ...option) (previous option) {
    for _, opt := range opts {
        previous = opt(l)
    }
    return previous
}

func Verbosity(v int) option {
    return func(l *Logger) option {
        previous := l.verbosity
        l.verbosity = v
        // Returns the restore function.
        return Verbosity(previous)
    }
}
```

`option` is a self-referential function type which returns a new function with the same type of itself. The returned function reverts the modified option. `Option` function receives a variable count of options, executes them and returns a revert function of the last option.

Usage looks like:

```go
func DoSomethingVerbosely(l *Logger, verbosity int) {
    previous := l.Option(logger.Verbosity(verbosity)) // logger is the package name
    defer l.Option(previous)
    // Do something with new verbosity.
}
```

Rob Pike’s design seems pretty good in that:

1. The user will get only one API, `Option`, along with several built-in option functions. The option functions can be well placed in a package.
2. Multiple options can be modified with one function call.
3. Modifications to options can be easily reverted.

However, the revert function returned by `Option` only works for the last option. What if there are multiple modified options that need to be reverted?

Well, this is not easy to be done in Rob Pike’s solution. Although each `option` function returns its own revert function, it is impossible to simply create a new `option` function and execute the revert functions one by one, as our new `option` function also needs to return its revert function. And we don't know what to return.

Things will be easier if we can merge multiple `option`s into one.

```go
// merge merges two options o and opt.
func (o option) merge(opt option) option {
    return func(l *Logger) option {
        previous := o(l)
        return previous.merge(opt(l))
    }
}

// Option sets the options specified.
// It returns an option to revert the modification to the options.
func (l *Logger) Option(opts ...option) (previous option) {
    for i, opt := range opts {
        if i == 0 {
            previous = opt(l)
        } else {
            previous = previous.merge(opt(l))
        }
    }
    return previous
}
```

Now the revert function returned by `Option` can restore all the modified options.

A complete demo for this new design can be found at my [GitHub repo](https://github.com/beta/go-better-options).