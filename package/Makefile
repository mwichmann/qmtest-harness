# values passed to the spec file
PACKAGE=lsb-qm
REAL_NAME=qmtest
VERSION=2.2
LSBSpec=5.0
LSBRelease=5
BUILDNO=2

# for pkgchk
DEPS=-d lsb-python

#
# Derive date string for daily snapshots
ISO_DATE=$(shell date +"%Y%m%d")
PWD=$(shell pwd)

FULL_PACKAGE_NAME=$(PACKAGE)-$(VERSION)
TARBALL=qm_$(VERSION).orig.tar.gz
RPM_BINARY_NAME=$(FULL_PACKAGE_NAME)-$(RELEASE).lsb$(LSBRelease).$(RPM_BUILD_ARCH).rpm
RPM_SOURCE_NAME=$(FULL_PACKAGE_NAME)-$(RELEASE).lsb$(LSBRelease).src.rpm
TARBALL_URL=http://old-releases.ubuntu.com/ubuntu/pool/universe/q/qm
TARBALL_ALT_URL=http://ftp.linuxfoundation.org/pub/lsb/impl/packages

# Temporary build directory
TMP_BUILD_DIR=/tmp/$(FULL_PACKAGE_NAME)

# Handle different version generation for snapshots than for official builds
# OFFICIAL_RELEASE should be set to the tag to extract from CVS
ifdef OFFICIAL_RELEASE
RELEASE=$(BUILDNO)
# EXPORT_TAG should be set to the bzr revisionspec to use
EXPORT_TAG=--revision=$(OFFICIAL_RELEASE)
else
RELEASE=0.$(ISO_DATE).$(BUILDNO)
EXPORT_TAG=
endif

# Determine whether to use rpm or rpmbuild to build the packages
ifeq ($(wildcard /usr/bin/rpmbuild),)
	RPM_BUILD_CMD=rpm
else
	RPM_BUILD_CMD=rpmbuild 
endif

# Get RPM configuration information
# NOTE THAT RPM_TMP_BUILD_DIR IS DELETED AFTER THE RPM BUILD IS COMPLETED
# The rpmrc file translates targets where there are multiple choices per
# architecture. On build, the derived RPM_BUILD_ARCH is given as the target
RCFILELIST="/usr/lib/rpm/rpmrc:./rpmrc"
MACROFILELIST="/usr/lib/rpm/macros:/etc/rpm:~/.rpmmacros:./rpmmacros"
RPM_TMP_BUILD_DIR=$(HOME)/tmp/rpm-build
RPM_BUILD_ARCH=$(shell rpm --rcfile ${RCFILELIST} --eval=%{_target_cpu})
RPM_BINARY_DIR=$(RPM_TMP_BUILD_DIR)/RPMS/$(RPM_BUILD_ARCH)
RPM_SRPM_DIR=$(RPM_TMP_BUILD_DIR)/SRPMS

# Override this on the command line to use a different repo
BZRTREES?=http://bzr.freestandards.org/lsb/devel
BZR_MODULE=qmtest-harness

# Default target
ifndef BUILD_NO_DEB
all: rpm_package deb_package
else
all: rpm_package
endif

clean:
	@rm -f *.rpm *.deb $(PACKAGE).spec journal.*

tarball: $(TARBALL) 

# Specfile generation rule
%.spec : %.spec.sed
	sed -e "s#@LSBRELEASE@#`echo $(LSBRelease)`#" -e "s#@VERSION@#`echo $(VERSION)`#" -e "s#@RELEASE@#`echo $(RELEASE)`#" < $< > $@

deb_package: rpm_package
	@fakeroot alien -cdk $(RPM_BINARY_NAME)

rpm_package: $(RPM_BINARY_NAME) $(RPM_SOURCE_NAME) 

list_uploadable:
	@echo $(RPM_BINARY_NAME)
ifndef BUILD_NO_DEB
	@ls *.deb
endif

$(RPM_BINARY_NAME) $(RPM_SOURCE_NAME): $(PACKAGE).spec $(TARBALL) 
	@mkdir -p $(RPM_TMP_BUILD_DIR)/BUILD
	@mkdir -p $(RPM_TMP_BUILD_DIR)/RPMS
	@mkdir -p $(RPM_TMP_BUILD_DIR)/SRPMS
ifdef SIGN_PACKAGES
	@expect -c 'set timeout -1' -c 'spawn $(RPM_BUILD_CMD) --sign --rcfile ${RCFILELIST} --macros=$(MACROFILELIST) --define=_sourcedir\ $(PWD) --define=_topdir\ $(RPM_TMP_BUILD_DIR) --define=_target_cpu\ $(RPM_BUILD_ARCH) -ba $(PACKAGE).spec' -c 'expect -ex "Enter pass phrase:"' -c 'send "\n"' -c 'expect "Executing(%clean)"' -c 'expect "exit 0"' -c 'send "\n"'
else
	@$(RPM_BUILD_CMD) --rcfile ${RCFILELIST} --macros=$(MACROFILELIST) --define="_sourcedir $(PWD)" --define="_topdir $(RPM_TMP_BUILD_DIR)" --define="_target_cpu $(RPM_BUILD_ARCH)" -ba $(PACKAGE).spec
endif
	@mv $(RPM_SRPM_DIR)/$(RPM_SOURCE_NAME) .
	@mv $(RPM_BINARY_DIR)/$(RPM_BINARY_NAME) .
	@rm -rf $(RPM_TMP_BUILD_DIR)

pkgchk: $(RPM_BINARY_NAME)
	/opt/lsb/bin/lsbpkgchk -L lsb $(DEPS) -t $(RPM_BINARY_NAME)

$(TARBALL):
	wget $(TARBALL_URL)/$(TARBALL) || wget $(TARBALL_ALT_URL)/$(TARBALL)

.PHONY : tarball rpm_package
