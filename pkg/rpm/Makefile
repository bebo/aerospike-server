# Build Aerospike Server RPM Distribution.

DEPTH = ../..
include $(DEPTH)/make_in/Makefile.vars

PKG = $(realpath $(DEPTH)/pkg)
SOURCE_ROOT = $(PKG)/dist
BUILD_ROOT = $(SOURCE_ROOT)/BUILD
OPT_AS = $(BUILD_ROOT)/opt/aerospike

REV = $(shell $(DEPTH)/build/version | sed 's/-/_/g')
OS = $(shell $(DEPTH)/build/os_version)

ifeq ($(USE_XDR),1)
all:	dist-xdr package clean
else
all:	dist package clean
endif

.PHONY:dist
dist:
	install -d $(BUILD_ROOT)/etc/aerospike
	install -d $(BUILD_ROOT)/etc/init.d
	install -d $(BUILD_ROOT)/etc/logrotate.d
	install -d $(BUILD_ROOT)/usr/bin
	install -d $(BUILD_ROOT)/var/log/aerospike
	install -d $(BUILD_ROOT)/var/run/aerospike

	install -pm 755 $(BIN_DIR)/asd $(BUILD_ROOT)/usr/bin/asd
	install -pm 755 $(DEPTH)/tools/citrus2aero/upgrade2to3 $(BUILD_ROOT)/usr/bin/asmigrate2to3
	install -pm 755 $(DEPTH)/tools/fixownership/fixownership.py $(BUILD_ROOT)/usr/bin/asfixownership

	install -pm 644 $(DEPTH)/as/etc/aerospike.conf $(BUILD_ROOT)/etc/aerospike
	install -pm 644 $(DEPTH)/as/etc/aerospike_ssd.conf $(BUILD_ROOT)/etc/aerospike
	install -pm 644 $(DEPTH)/as/etc/aerospike_mesh.conf $(BUILD_ROOT)/etc/aerospike

	install -pm 755 $(DEPTH)/as/etc/init-script $(BUILD_ROOT)/etc/init.d/aerospike
	install -pm 644 $(DEPTH)/as/etc/logrotate_asd $(BUILD_ROOT)/etc/logrotate.d/aerospike

ifeq ($(USE_ASM),1)
	install -pm 644 $(DEPTH)/as/etc/initfns $(BUILD_ROOT)/etc/aerospike
	install -d $(OPT_AS)/{bin,lib}
	install -pm 755 $(DEPTH)/as/etc/run-under-gdb $(OPT_AS)/bin
	install -pm 644 $(DEPTH)/modules/asmalloc/lib/asmalloc.jem.so $(OPT_AS)/lib
	install -pm 644 $(DEPTH)/modules/jemalloc/lib/libjemalloc.so $(OPT_AS)/lib
endif

	install -d $(OPT_AS)/data
	install -d $(OPT_AS)/smd
	install -d $(OPT_AS)/sys/udf/lua/external
	install -d $(OPT_AS)/sys/udf/lua/ldt
	install -d $(OPT_AS)/usr/udf/lua
	install -pm 644 $(DEPTH)/modules/lua-core/src/*.lua $(OPT_AS)/sys/udf/lua
	install -pm 644 $(DEPTH)/modules/lua-core/src/external/* $(OPT_AS)/sys/udf/lua/external
	install -pm 644 $(DEPTH)/modules/lua-core/src/ldt/* $(OPT_AS)/sys/udf/lua/ldt

	install -d $(OPT_AS)/doc
	install -pm 644 $(DEPTH)/LICENSE $(OPT_AS)/doc

package:
	install -d $(SOURCE_ROOT)/RPMS/x86_64

	sed 's/@VERSION@/'$(REV)'/g' < $(PKG)/rpm/server-spec-base > $(PKG)/rpm/aerospike.spec
ifeq ($(USE_ASM),1)
	cat $(PKG)/rpm/asm-files >> $(PKG)/rpm/aerospike.spec
endif
ifeq ($(USE_XDR),1)
	cat $(EEREPO)/pkg/rpm/xdr-files >> $(PKG)/rpm/aerospike.spec
endif
	cat $(PKG)/rpm/server-spec-scripts >> $(PKG)/rpm/aerospike.spec

	sed -i 's/@RELEASE@/'$(OS)'/g' $(PKG)/rpm/aerospike.spec
	sed -i 's/@EDITION@/'$(EDITION)'/g' $(PKG)/rpm/aerospike.spec
	cd $(DEPTH); rpmbuild -bb -vv --buildroot $(BUILD_ROOT) $(PKG)/rpm/aerospike.spec

	find $(SOURCE_ROOT)/RPMS -type f -exec mv {} $(PKG)/packages \;

clean:
	rm -rf $(PKG)/rpm/aerospike.spec
	rm -rf $(SOURCE_ROOT)/*

ifeq ($(USE_XDR),1)
  include $(EEREPO)/xdr/make_in/Makefile.rpm.in
endif
