#
# Standalone NPF makefile: the kernel component as a library.
# This file is in the Public Domain.
#

CFLAGS=		-std=gnu99 -O2 -flto -g -Wall -Wextra -Werror
CFLAGS+=	-Wstrict-prototypes -Wmissing-prototypes -Wpointer-arith
CFLAGS+=	-I stand -D_NPF_STANDALONE #-DUSE_JUDY
CFLAGS+=	-D__RCSID\(x\)=

CFLAGS+=	-Wno-unused-parameter -Wno-unused-variable -Wno-unused-function
CFLAGS+=	-Wno-unused

SYSNAME:=	$(shell uname -s)

#
# WARNING: All symbols must be hidden by default to not conflict with
# the libnpf(3) library.  The debug version would, however, conflict.
#
ifeq ($(DEBUG),1)
CFLAGS+=	-O1 -DDEBUG -fno-omit-frame-pointer
CFLAGS+=	-D_NPF_TESTING
else
CFLAGS+=	-fvisibility=hidden
CFLAGS+=	-DNDEBUG
LDFLAGS=	-flto
endif
LDFLAGS+=	-lpthread -lpcap -lqsbr -llpm -lcdb -lprop -lbpfjit
ifeq ($(SYSNAME),Linux)
LDFLAGS+=	-ljemalloc #-lJudy
endif

#
# System specific flags
#

ifeq ($(SYSNAME),Linux)
CFLAGS+=	-D_POSIX_C_SOURCE=200809L
CFLAGS+=	-D_BSD_SOURCE -D__FAVOR_BSD=1
endif

#
# Objects to compile
#

OBJS=		$(shell grep ^file files.npf | cut -f2 | \
		    sed 's/\(.*\)net\/npf\/\(.*\).c/\2.o/g' | \
		    grep -v npf_os | grep -v npf_alg_ | \
		    grep -v npf_ext_ | \
		    grep -v if_npflog)

OBJS+=		stand/bpf_filter.o
OBJS+=		stand/rb.o stand/murmurhash.o stand/subr_hash.o

#
# Flags for the library target
#

LIB=		libnpfkern
ifeq ($(DEBUG),1)
LIB=		libnpfkern-dbg
endif

INCS=		npf.h npfkern.h
MANS=		npfkern.3

$(LIB).la:	LDFLAGS+=	-rpath $(LIBDIR)
install/%.la:	ILIBDIR=	$(DESTDIR)/$(LIBDIR)
install:	IINCDIR=	$(DESTDIR)/$(INCDIR)
install:	IMANDIR=	$(DESTDIR)/$(MANDIR)/man3/

#
# Targets
#

lib: $(LIB).la

%.lo: %.c
	libtool --mode=compile --tag CC $(CC) $(CFLAGS) -c $<

$(LIB).la: $(shell echo $(OBJS) | sed 's/\.o/\.lo/g')
	libtool --mode=link --tag CC $(CC) $(LDFLAGS) -o $@ $(notdir $^)

install/%.la: %.la
	mkdir -p $(ILIBDIR)
	libtool --mode=install install -c $(notdir $@) $(ILIBDIR)/$(notdir $@)

install: $(addprefix install/,$(LIB).la)
	libtool --mode=finish $(LIBDIR)
	mkdir -p $(IINCDIR)/net && install -c $(INCS) $(IINCDIR)/net
	mkdir -p $(IMANDIR) && install -c $(MANS) $(IMANDIR)

clean:
	libtool --mode=clean rm
	rm -rf .libs *.o *.lo *.la stand/*.{l,}o stand/*.la

.PHONY: all lib install clean
