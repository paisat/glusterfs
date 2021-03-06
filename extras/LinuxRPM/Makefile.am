
GFS_TAR = ../../glusterfs-*git.tar.gz
UFO_TAR = ../../gluster-swift-ufo-*git.tar.gz
GFS_SPEC = ../../glusterfs.spec

.PHONY: all

all:
	@echo "To build RPMS run 'make glusterrpms'"

.PHONY: glusterrpms prep srcrpm testsrpm clean

glusterrpms: prep srcrpm rpms
	-rm -rf rpmbuild

glusterrpmswithoutufo: prep srcrpm rpmswithoutufo
	-rm -rf rpmbuild

prep::
	if [ ! -e $(GFS_TAR) -a ! -e $(UFO_TAR) -a ! -e $(GFS_SPEC) ]; then \
		$(MAKE) -c ../.. dist; \
	fi
	-mkdir -p rpmbuild/SPECS
	-mkdir -p rpmbuild/RPMS
	-mkdir -p rpmbuild/SRPMS
	-rm -rf rpmbuild/SOURCES
	@if [ -d /d/cache/glusterfs -a -e /d/cache/glusterfs/sources ]; then \
		echo "copying glusterfs rpm files from local cache..."  ; \
		mkdir -p ./rpmbuild/SOURCES; \
		cp /d/cache/glusterfs/* ./rpmbuild/SOURCES/ ; \
		touch /d/cache/.gluster ; \
	elif [ -x /usr/bin/git ]; then \
		echo "fetching glusterfs rpm files from fedora git repo..."; \
		cd ./rpmbuild && git clone git://pkgs.fedoraproject.org/glusterfs.git > /dev/null && mv glusterfs SOURCES; \
	else \
		echo "glusterfs rpm files not fetched, you don't have git installed!" ; \
		exit 1 ; \
	fi
	cp ../../*.tar.gz ./rpmbuild/SOURCES
	cp ../../glusterfs.spec ./rpmbuild/SPECS

# Remember to change folsom to grizzly when we rebase to grizzly
SWIFT_TARBALL = $(shell grep -v gluster ./rpmbuild/SOURCES/sources | cut -d ' ' -f 3)
SWIFT_MD5SIG = $(shell grep -v gluster ./rpmbuild/SOURCES/sources | cut -d ' ' -f 1)
SWIFT_VERS = $(shell echo $(SWIFT_TARBALL) | grep -o 1\.[0-9]\.[0-9])
SWIFT_URL = https://launchpad.net/swift/folsom/$(SWIFT_VERS)/+download/$(SWIFT_TARBALL)

prep::
	@if [ -e /d/cache/$(SWIFT_TARBALL) -a -d ./rpmbuild/SOURCES ]; then \
		echo "copying swift source tarball from local cache..." ; \
		cp /d/cache/$(SWIFT_TARBALL) ./rpmbuild/SOURCES/ ; \
		touch /d/cache/.swift ; \
	elif [ -x /usr/bin/curl -a -d ./rpmbuild/SOURCES ]; then \
		echo "fetching swift from launchpad.net..." ; \
		cd ./rpmbuild/SOURCES && /usr/bin/curl -sOL $(SWIFT_URL) ; \
	else \
		echo "swift source not fetched, you don't have curl installed!" ; \
		exit 1 ; \
	fi

srcrpm:
	rpmbuild --define '_topdir $(shell pwd)/rpmbuild' -bs rpmbuild/SPECS/glusterfs.spec
	mv rpmbuild/SRPMS/* .

rpms:
	rpmbuild --define '_topdir $(shell pwd)/rpmbuild' -bb rpmbuild/SPECS/glusterfs.spec
	mv rpmbuild/RPMS/*/* .

rpmswithoutufo:
	rpmbuild --define '_topdir $(shell pwd)/rpmbuild' -bb rpmbuild/SPECS/glusterfs.spec --without ufo
	mv rpmbuild/RPMS/*/* .

# EPEL-5 does not like new versions of rpmbuild and requires some
# _source_* defines

testsrpm: prep
	rpmbuild --define '_topdir $(shell pwd)/rpmbuild' \
		--define '_source_payload w9.gzdio' \
		--define '_source_filedigest_algorithm 1' \
		-bs rpmbuild/SPECS/glusterfs.spec
	mv rpmbuild/SRPMS/* ../..
	-rm -rf rpmbuild

clean:
	-rm -rf rpmbuild
	-rm -f *.rpm
