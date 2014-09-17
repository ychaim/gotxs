This is attempt at creating Go bindings for OpenTransactions (opentxs).

Instructions
============

In order install the bindings, you need to link the necessary header files from
`opentxs` to the `include/` subdirectory.

````
cd gotxs/opentxs/
ln -s $HOME/path/to/opentxs/deps/containers/ include/
ln -s $HOME/path/to/opentxs/include/opentxs/ include/
# fix the paths in Makefile
make install
````

Then you can install the `gotxs` package

````
go install
````
