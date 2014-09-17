# Some of the hard-coded paths could be determined at runtime

LIBPATH=/usr/local/lib64
GOINCLUDE=/usr/lib/go/pkg/linux_amd64

opentxs_wrap.cxx opentxs_gc.c opentxs.go: opentxs.i
	swig -soname libopentxs-golang.so -c++ -go -intgosize 64 opentxs.i

libopentxs-golang.so: opentxs_wrap.cxx
	g++ -g -fpic -shared -std=c++11 -DEXPORT= \
		-Iinclude/ \
		-L$(LIBPATH) \
		-lopentxs-client \
		-lopentxs-cash \
		-lopentxs-basket \
		-lopentxs-ext \
		-lopentxs-core \
		$< -o $@

	# we need to remove this file because
	# it confuses `go install'
	rm $<

opentxs.a: opentxs_gc.c opentxs.go
	rm -f $@
	go tool 6c -I $(GOINCLUDE) -D _64BIT opentxs_gc.c
	go tool 6g opentxs.go
	go tool pack grc opentxs.a opentxs.6 opentxs_gc.6

clean:
	rm -f *.so
	rm -f *.a
	rm -f *.6
	rm -f *.go
	rm -f *.c
	rm -f *.cxx

install: opentxs.a libopentxs-golang.so
	sudo cp -v libopentxs-golang.so $(LIBPATH)
	sudo ldconfig
	go install