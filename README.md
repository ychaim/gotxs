This is an attempt at creating Go bindings for OpenTransactions (opentxs) using
SWIG.

# Requirements

* Swig 3 (packages stuck at v. 2 in Ubuntu, compile from source)
* Go 1.3 (packages stuck at v. 1.2 in Ubuntu, install manually)
* recent opentxs

# Build and install instructions

In order to install the bindings, you need to link the necessary header files
from `opentxs` to the `include/` subdirectory:

```
cd gotxs/opentxs/

ln -s /path/to/opentxs/deps/containers include/
ln -s /path/to/opentxs/include/opentxs include/

# fix the paths in Makefile
$EDITOR Makefile

make install
```

Then you can install the `gotxs` package. This will install the `gotxs/opentxs`
sub-package as well:

```
# in gotxs/
go install
```

To run the tests you need the gocheck library:

```
go get gopkg.in/check.v1

# match LIBPATH as defined in the Makefile (different on Mac OS/X)
export LD_LIBRARY_PATH=/usr/local/lib/x86_64-linux-gnu

go test
```

## Mac OS/X

Additional changes to the Makefile are required on OS/X:

1. `LIBPATH` should point to the `lib/` dir of your opentxs install.
2. `GOINCLUDE` points to `/usr/local/Cellar/go/1.3.1/libexec/pkg/darwin_amd64`
   (or whatever version you have).
3. `LIBNAME` is `libopentxs-golang.dylib`.
4. Add the magic `-flat_namespace` and `-undefined suppress` to the g++ line.

Run `make`. Ignore the ldconfig error, it's a Linux thing.
Then export an environment variable to point to the installed dylib:

```
$ set -xg DYLD_LIBRARY_PATH ~/opentxs/lib/
```

Possibly `DYLD_FALLBACK_LIBRARY_PATH` is better.

# Development

The bindings consist of three separate packages:

## Package *gotxs/opentxs*

This cointains a module generated by SWIG from an interface file. It exports a
few hundred functions that are translated from C++, which can be seen in
[this text file](opentxs/opentxs.txt).

Low-level API functions start with the prefix `OTAPI_Wrap` and are static
methods of the C++ class with the same.

A higher-level API that provides network synchronization is available through
instances of the class `OT_ME` (*OpenTransactions Made Easy*).

The general rule is that the `OT_ME` bindings should be used when possible.

Unfortunately, the exported methods are difficult to use directly:

* the methods have no documentation;
* there are initialization steps that need to be executed;
* some of the called methods unexpectedly crash on invalid arguments (process
  hangs indefinitely due to the `OT_FAIL` macro);
* the methods sometimes use special return values, like the empty string or `-1`
  to signal error conditions. This is contrary to the Go way of signaling
  errors via multiple return values.

## Packages *gotxs* and *gotxs/easy*

In order to fix this issues, we wrap the SWIG-generated methods by hand in our
own packages. All methods exported by `OTAPI_Wrap` go to the `gotxs` packages.
The higher-level methods exposed by the `OT_ME` go to `gotxs/easy` where we
instantiate a single class instance.

### Naming convention

For Go-exported methods, we simply copy the name and translate it to CamelCase
with a leading capital.

### Documentation

Each method should provide documentation that should go beyond *calls underlying
method foobar*. When the documentation isn't available from the wrapped C++
methods, the implementation needs to be examined.

### Input sanitation

Some wrapped method calls crash hard and freeze the process on invalid input
(see opentxs issue [#196](https://github.com/Open-Transactions/opentxs/issues/196)).
We should be careful to catch these errors early on the Go side, where it makes sense.

If you find an invalid input that causes freeze where it shouldn't, add it to
the opentxs issue linked above or open a new one there.

### Return values

The C++ methods often encode error conditions in the return value. The
Go wrappers translate these into multiple return values.  For now, we use simple
conditionals (`if retval == ""`) to check the C++ return values. The error
should be descriptive and explain what went wrong.

See [Error handling and Go](http://blog.golang.org/error-handling-and-go) for
more information.

### Code Style

We use `go fmt` to format the source.

### Navigating the C++ code

I recommend Doxygen with the call and caller graphs for browsing the opentxs C++
code:

```
# in opentxs/
# install doxygen and graphviz

cmake .. -DDOC_FULLGRAPHS=YES
make doc
```
