+++
title = "Magefiles"
weight = 20
+++


A mage file is any regular go file marked with a build target of "mage" and in
package main.

```go
//go:build mage

package main
```

You can quickly create a template mage file with the `-init` option.

`mage -init`

You may have any number of magefiles in the same directory.  Mage doesn't care
what they're named aside from normal go filename rules.  All they need is to
have the mage build target.  Handily, this also excludes them from your regular
builds, so they can live side by side with your normal go files.  Magefiles may
use any of Go's usual build constraints, so you can include and exclude
magefiles based on OS, arch, etc, whether in the filename or in the +build line.

```go
//go:build mage

// A comment on the package will be output when you list the targets of a
// magefile.
package main

import (
    "log"
    "os"
)


// Build target is any exported function with zero args with no return or an error return.
// If a target has an error return and returns an non-nil error, mage will print
// that error to stdout and return with an exit code of 1.
func Install() error {

}

// The first sentence in the comment will be the short help text shown with mage -l.
// The rest of the comment is long help text that will be shown with mage -h <target>
func Target() {
    // by default, the log stdlib package will be set to discard output.
    // Running with mage -v will set the output to stdout.
    log.Printf("Hi!")
}

// A var named Default indicates which target is the default.  If there is no 
// default, running mage will list the targets available.
var Default = Install


// Because build targets are case insensitive, you may not have two build targets
// that lowercase to the same text.  This would give you an error when you tried
// to run the magefile:
// func BUILD() {}

// Targets may have a context argument, in which case a default context is passed
// to the target, which will be cancelled after a timeout if the -t flag is used.
func Build(ctx context.Context) {
    mg.CtxDeps(ctx, Target)
}

```

```plain
$ mage -l
A comment on the package will be output when you list the targets of a
magefile.

Targets:
  install*   Build target is any exported function with zero args with no return or an error return.
  target     The first sentence in the comment will be the short help text shown with mage -l.

* default target
```

```plain
$ mage -h target
mage target:

The first sentence in the comment will be the short help text shown with mage -l.
The rest of the comment is long help text that will be shown with mage -h <target>
```

### Magefiles directory

Some development IDEs might work erratically with magefiles sharing location with other go files which very well might
have a different package name. 

To simplify development, if you place your Magefiles inside a `magefiles` directory, and none remains in your
main folder, the `mage` command will pick the directory as the source of your targets while keeping the current directory
as the working one.

The `magefiles` directory does not require for you to add build tags to the files in it, it is optional, however we 
encourage you to keep it consistent, either all or none.

Effectively this is the equivalent of running: `mage -d magefiles -w .`