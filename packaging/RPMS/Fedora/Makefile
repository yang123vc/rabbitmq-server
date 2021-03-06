SOURCE_DIST_FILE ?= $(wildcard ../../../rabbitmq-server-*.tar.xz)

ifneq ($(filter-out clean,$(MAKECMDGOALS)),)
ifeq ($(SOURCE_DIST_FILE),)
$(error Cannot find source archive; please specify SOURCE_DIST_FILE)
endif
ifneq ($(words $(SOURCE_DIST_FILE)),1)
$(error Multiple source archives found; please specify SOURCE_DIST_FILE)
endif

VERSION ?= $(patsubst rabbitmq-server-%.tar.xz,%,$(notdir $(SOURCE_DIST_FILE)))
ifeq ($(VERSION),)
$(error Cannot determine version; please specify VERSION)
endif
endif

TOP_DIR = $(shell pwd)
# Under debian we do not want to check build dependencies, since that
# only checks build-dependencies using rpms, not debs
DEFINES = --define '_topdir $(TOP_DIR)' --define '_tmppath $(TOP_DIR)/tmp' \
	  --define '_sysconfdir /etc' --define '_localstatedir /var'

ifndef RPM_OS
RPM_OS = fedora
endif

ifeq "$(RPM_OS)" "suse"
FUNCTION_LIBRARY=
REQUIRES=/sbin/chkconfig /sbin/service
OS_DEFINES=--define '_initrddir /etc/init.d' --define 'dist .suse'
SPEC_DEFINES=--define 'group_tag Productivity/Networking/Other'
START_PROG=startproc
else
FUNCTION_LIBRARY=\# Source function library.\n. /etc/init.d/functions
REQUIRES=chkconfig initscripts
OS_DEFINES=--define '_initrddir /etc/rc.d/init.d'
ifeq "$(RPM_OS)" "rhel6"
SPEC_DEFINES=--define 'group_tag Development/Libraries' --define 'dist .el6' --define 'rhel 6'
else ifeq "$(RPM_OS)" "rhel7"
SPEC_DEFINES=--define 'group_tag Development/Libraries' --define '_unitdir /usr/lib/systemd/system' --define 'dist .el7' --define 'rhel 7'
else
SPEC_DEFINES=--define 'group_tag Development/Libraries'
endif
START_PROG=daemon
endif

unexport DEPS_DIR
unexport ERL_LIBS

.PHONY: all prepare server clean

all:   clean server
	@:

prepare:
	mkdir -p BUILD SOURCES SPECS SRPMS RPMS tmp
	cp $(SOURCE_DIST_FILE) SOURCES
	cp rabbitmq-server.spec SPECS
	sed -i 's|%%VERSION%%|$(VERSION)|;s|%%REQUIRES%%|$(REQUIRES)|' \
		SPECS/rabbitmq-server.spec

	cp rabbitmq-server.service SOURCES/rabbitmq-server.service
	cp rabbitmq-server.tmpfiles SOURCES/rabbitmq-server.tmpfiles
	cp rabbitmq-server.init SOURCES/rabbitmq-server.init
	sed -i \
	    -e 's|^START_PROG=.*$$|START_PROG="$(START_PROG)"|' \
	    -e 's|^@FUNCTION_LIBRARY@|$(FUNCTION_LIBRARY)|' \
	    SOURCES/rabbitmq-server.init
	cp rabbitmq-server.logrotate SOURCES/rabbitmq-server.logrotate

server: prepare
	rpmbuild -ba --nodeps SPECS/rabbitmq-server.spec $(DEFINES) $(OS_DEFINES) $(SPEC_DEFINES)

	if test "$(PACKAGES_DIR)"; then \
		mkdir -p "$(PACKAGES_DIR)"; \
		mv SRPMS/*.rpm RPMS/noarch/*.rpm "$(PACKAGES_DIR)"; \
	fi

clean:
	rm -rf SOURCES SPECS RPMS SRPMS BUILD tmp
