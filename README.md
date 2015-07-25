This repository is an illustration of how to use
[Bazel](http://bazel.io) with [Travis-CI](https://travis-ci.org/).

[![Build Status](https://travis-ci.org/korfuri/bazel-travis.svg?branch=master)](https://travis-ci.org/korfuri/bazel-travis)

# Caveat

Currently (as of 2015/07/24), Bazel doesn't provide binary
releases. The `.travis.yml` configuration does a git clone/build from
source for each run, which is quite slow and wasteful.

# How it works

There's a couple of tricks in here to make things work.

Basically, `.travis.yml` lists a couple of apt packages that need to
be installed to compile Bazel, and in `before_install` it clones the
latest version of Bazel and compiles it.

To do so it provides its own `bazelrc` file, which limits the RAM
allocated to Bazel. It also adds more verbosity to the build
process. Note that this `bazelrc` is not used to compile your code:
it's used during the self-hosting process of Bazel.

Since Travis doesn't currently support a GCC more recent than 4.6, and
since Bazel won't build with such an old compiler, a custom crosstool
is provided, and is passed to Bazel via the bazelrc.

# How do I use this?

You will need:

  * Everything that's in `.travis.yml`. Note that you can just merge
    that into your existing `.travis.yml` just fine - it doesn't
    specify any exclusive option, except that it requires Linux
    (either container-based or legacy).
  * Everything that's in `.bazel-travis`, this includes the bazelrc
    and the custom crosstool.
  * A (possibly empty) `WORKSPACE` file. See below.
  * Your own `script` section in the `.travis.yml` config file.

## Where to put the WORKSPACE file

If you don't know what a `WORKSPACE` file is, please refer to
[the relevant Bazel documentation](http://bazel.io/docs/getting-started.html).

In this example, I chose to put `WORKSPACE` at the root of the
repository. This has the advantage that I can follow the convention of
having my code in `/src/` at the root of the repository and have that
be `//src/` for Bazel.

The disadvantage is that since bazel is cloned under the local
directory, `//bazel` is a valid package and commands like `bazel build
//...` will include `//bazel`, resulting in a lot of extra work, and
slower builds. To avoid that, you may wish to keep your WORKSPACE file
under `/src` or whatever subdirectory your code lives in. Note that
Bazel likes to have tests and code side-by-side, so you don't need a
toplevel `/tests` directory for the tests.

## Using Bazel in your Travis config

The bazel binary is available at `bazel/output/bazel`. You can do
stuff like:

```yaml
script:
  - bazel/output/bazel test //src/...
```

but see the caveat in the "Where to put the WORKSPACE file" section
above if you plan to use `//...`.
