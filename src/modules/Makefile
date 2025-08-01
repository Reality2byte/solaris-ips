#
# CDDL HEADER START
#
# The contents of this file are subject to the terms of the
# Common Development and Distribution License (the "License").
# You may not use this file except in compliance with the License.
#
# You can obtain a copy of the license at usr/src/OPENSOLARIS.LICENSE
# or http://www.opensolaris.org/os/licensing.
# See the License for the specific language governing permissions
# and limitations under the License.
#
# When distributing Covered Code, include this CDDL HEADER in each
# file and include the License file at usr/src/OPENSOLARIS.LICENSE.
# If applicable, add the following below this CDDL HEADER, with the
# fields enclosed by brackets "[]" replaced with your own identifying
# information: Portions Copyright [yyyy] [name of copyright owner]
#
# CDDL HEADER END
#

#
# Copyright (c) 2024, 2025, Oracle and/or its affiliates.
#

include ../Makefile.constants

EXTENSIONS = actions/_actions actions/_common _varcet _misc solver elf

PYFILES:sh = find . -type f -name '*.py'

all: $(EXTENSIONS:=.cpython-311.so)

PYTHON311CFLAGS:sh =	python3.11-config --cflags

PYTHON311LDFLAGS:sh =	python3.11-config --ldflags

%-311.so := PYTHONCFLAGS=$(PYTHON311CFLAGS)

%-311.so := PYTHONLDFLAGS=$(PYTHON311LDFLAGS) -shared -zstrip-class=nonalloc

_misc.cpython-%.so: _misc.c
	$(GCC) $^ $(PYTHONCFLAGS) $(PYTHONLDFLAGS) -o $@

_varcet.cpython-%.so: _varcet.c
	$(GCC) $^ $(PYTHONCFLAGS) $(PYTHONLDFLAGS) -o $@

actions/_actions.cpython-%.so: actions/_actions.c
	$(GCC) $^ $(PYTHONCFLAGS) $(PYTHONLDFLAGS) -o $@

actions/_common.cpython-%.so: actions/_common.c
	$(GCC) $^ $(PYTHONCFLAGS) $(PYTHONLDFLAGS) -o $@

elf.cpython-%.so: elf.c elfextract.c liblist.c
	$(GCC) $^ $(PYTHONCFLAGS) -I. -lelf -lmd $(PYTHONLDFLAGS) -o $@

solver.cpython-%.so: solver/solver.c solver/py_solver.c
	$(GCC) $^ -D_FILE_OFFSET_BITS=64 -Wno-return-type -Wno-strict-aliasing -Wno-unused-function -Wno-unused-variable $(PYTHONCFLAGS) -ztext -lm -lc $(PYTHONLDFLAGS) -o $@

# PRIVATE_BUILD set in the environment tells us to not strip
# the prefix of proto area paths recorded in the .pyc files.
$(PRIVATE_BUILD)PYCOMPILE_FLAGS = -s $(ROOT)


$(ROOTPY311PKG)/%.so: %.so
	$(INS.file)

$(ROOTPY311PKG)/%.py: %.py
	$(INS.file.pstrip)


ROOT311FILES = $(PYFILES:%=$(ROOTPY311PKG)/%)
ROOT311EXTENSIONS = $(EXTENSIONS:%=$(ROOTPY311PKG)/%.cpython-311.so)

install: $(ROOT311FILES) $(ROOT311EXTENSIONS)
	@# substitute VERSION with mercurial revision in pkg/__init__.py
	@hg_revision=`hg id -i`; \
	echo "doing version substitution: $$hg_revision"; \
	$(GSED) -i "s|VERSION = \"unknown\"|VERSION = \"$$hg_revision\"|" $(ROOTPY311PKG)/__init__.py

	@# compile all sources to bytecode
	$(PYTHON311) -m compileall $(PYCOMPILE_FLAGS) -j8 --invalidation-mode=checked-hash $(ROOTPY311PKG)

check:

clint: all
	$(LINT.c) -I. -lelf -lmd elf.c elfextract.c liblist.c
	$(LINT.c) actions/_actions.c
	$(LINT.c) actions/_common.c
	$(LINT.c) _varcet.c
	$(LINT.c) _misc.c
	$(LINT.c) solver/solver.c solver/py_solver.c

lint: clint

clean:
	rm -f $(EXTENSIONS:=.cpython-311.so)

clobber: clean
