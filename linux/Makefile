# @(#) $Id: Makefile,v 6.20 2023/02/15 11:10:05 ralph Exp $
# -------------------------------------------------------------------------
# vim:ts=8:sw=4:sts=4

# In some dists (e.g. Ubuntu) bash is not the default shell. Statements like
#   cp -a etc/cfg2html/{mappings,templates} ...
# assumes bash. So its better to set SHELL

SHELL=/bin/bash

DESTDIR =
OFFICIAL =

### Get version from Config 2 HTML itself
cfg2htmlbin = ../cfg2html
name = cfg2html
version := $(shell awk '/^CVS=/ { print $$3}' $(cfg2htmlbin))
version := $(shell git describe --long|tr "-" ".")

### Get the branch information from git
ifeq ($(OFFICIAL),)
ifneq ($(shell which git),)
git_date := $(shell git log -n 1 --format="%ai")
git_ref := $(shell git symbolic-ref -q HEAD)
## example: refs/heads/master
git_branch = $(lastword $(subst /, ,$(git_ref)))
endif
else
git_branch = cfg2html-$(version)
endif
git_branch ?= master

date := $(shell date --date="$(git_date)" +%Y%m%d%H%M)
release_date := $(shell date --date="$(git_date)" +%Y-%m-%d)

prefix = /usr
sysconfdir = /etc
sbindir = $(prefix)/sbin
datadir = $(prefix)/share
mandir = $(datadir)/man
localstatedir = /var

specfile = packaging/rpm/$(name).spec
dscfile = packaging/debian/$(name).dsc

distversion = $(version)
debrelease = 0
rpmrelease = %nil
obsproject = home:gdha
obspackage = $(name)-$(version)
ifeq ($(OFFICIAL),)
    distversion = $(version)-git$(date)
    debrelease = 0git$(date)
    rpmrelease = .git$(date)
    obsproject = home:gdha
    obspackage = $(name)
endif

ifneq ($(shell which apt 2>/dev/null),)
T := deb
else
T := rpm
endif

.PHONY: doc

all:
	@echo "Nothing to build. Use \`make help' for more information."

help:
	@echo -e "Config2HTML Linux make targets:\n\
\n\
  validate        - Check source code\n\
  install         - Install cfg2html (may replace files)\n\
  uninstall       - Uninstall cfg2html (may remove files)\n\
  dist            - Create tar file\n\
  deb             - Create DEB package\n\
  Linux           - tries to guess RPM or DEB creation\n\
  rpm             - Create RPM package\n\
  pacman          - Create Pacman package\n\
  obs             - Initiate OBS builds\n\
\n\
Config2HTML make variables (optional):\n\
\n\
  DESTDIR=        - Location to install/uninstall\n\
  OFFICIAL=       - Build an official release\n\
"

clean:
	rm -f ../$(name)-$(distversion).tar.gz
	rm -f build-stamp
	make -C doc clean
	rm -rf ../$(name).cron

### You can call 'make validate' directly from your .git/hooks/pre-commit script
validate:
	@echo -e "\033[1m== Validating scripts and configuration ==\033[0;0m"
	find etc/  -name '*.conf' | xargs bash -n
	find lib/ plugins/ -name '*.sh' | xargs bash -n
	bash -n $(cfg2htmlbin)
	find . -name '*.sh' | xargs bash -n
### Fails to work on RHEL4
#	find -L . -type l

man:
	@echo -e "\033[1m== Prepare manual ==\033[0;0m"
	make -C doc man

doc:
	@echo -e "\033[1m== Prepare documentation ==\033[0;0m"
	make -C doc docs

cron:
	@echo -e "\033[1m== Make a cron file ==\033[0;0m"
	echo "# Please uncomment (remove #) the line below to make it active" > ../$(name).cron
	echo "#$$((RANDOM % 55)) $$((RANDOM % 5)) * * $$((RANDOM % 5)) root /usr/sbin/cfg2html -2\%Y\%m\%d > /dev/null 2>&1" \
	>> ../$(name).cron

ifneq ($(git_date),)
rewrite:
	@echo -e "\033[1m== Rewriting $(specfile), $(dscfile) and $(cfg2htmlbin) ==\033[0;0m"
	cp -f $(specfile) $(specfile).orig
	cp -f $(dscfile) $(dscfile).orig
	cp -f $(cfg2htmlbin) $(cfg2htmlbin).orig
	sed -i.orig \
		-e 's#^Source:.*#Source: $(name)-$(distversion).tar.gz#' \
		-e 's#^Version:.*#Version: $(version)#' \
		-e 's#^%define rpmrelease.*#%define rpmrelease $(rpmrelease)#' \
		-e 's#^%setup.*#%setup -q -n $(name)-$(distversion)#' \
		$(specfile)
	sed -i.orig \
		-e 's#^Version:.*#Version: $(version)-$(debrelease)#' \
		$(dscfile)
	sed -i.orig \
		-e 's#^VERSION=.*#VERSION=$(distversion)#' \
		-e 's#^RELEASE_DATE=.*#RELEASE_DATE="$(release_date)"#' \
		$(cfg2htmlbin)

restore:
	@echo -e "\033[1m== Restoring $(specfile), $(dscfile) and $(cfg2htmlbin) ==\033[0;0m"
	mv -f $(specfile).orig $(specfile)
	mv -f $(dscfile).orig $(dscfile)
	mv -f $(cfg2htmlbin).orig $(cfg2htmlbin)
else
rewrite:
	@echo "Nothing to do."

restore:
	@echo "Nothing to do."
endif

### copy the local.conf file to the same directory as default.conf,
### then set /etc/cfg2html/local.conf from it.
install-config:
	@echo -e "\033[1m== Installing configuration ==\033[0;0m"
	install -d -m0700 $(DESTDIR)$(sysconfdir)/cfg2html/
	-[[ ! -e $(DESTDIR)$(datadir)/cfg2html/etc/default.conf ]] && \
		install -Dp -m0600 etc/default.conf $(DESTDIR)$(datadir)/cfg2html/etc/default.conf
	-[[ ! -e $(DESTDIR)$(datadir)/cfg2html/etc/local.conf ]] && \
		install -Dp -m0600 etc/local.conf $(DESTDIR)$(datadir)/cfg2html/etc/local.conf
	-[[ ! -e $(DESTDIR)$(sysconfdir)/cfg2html/plugins ]] && \
		install -Dp -m0600 plugins $(DESTDIR)$(sysconfdir)/cfg2html/plugins
	-[[ ! -e $(DESTDIR)$(sysconfdir)/cfg2html/systeminfo ]] && \
		install -Dp -m0600 systeminfo $(DESTDIR)$(sysconfdir)/cfg2html/systeminfo
	-[[ ! -e $(DESTDIR)$(sysconfdir)/cfg2html/files ]] && \
		install -Dp -m0600 files $(DESTDIR)$(sysconfdir)/cfg2html/files
	-find $(DESTDIR)$(sysconfdir)/cfg2html/ -name '.gitignore' -exec rm -rf {} \; &>/dev/null

install-bin:
	@echo -e "\033[1m== Installing binary ==\033[0;0m"
	install -Dp -m0755 $(cfg2htmlbin) $(DESTDIR)$(sbindir)/cfg2html
	sed -i -e 's,^CONFIG_DIR=.*,CONFIG_DIR="$(sysconfdir)/cfg2html",' \
		-e 's,^SHARE_DIR=.*,SHARE_DIR="$(datadir)/cfg2html",' \
		-e 's,^VAR_DIR=.*,VAR_DIR="$(localstatedir)/log/cfg2html",' \
		$(DESTDIR)$(sbindir)/cfg2html

install-data:
	@echo -e "\033[1m== Installing scripts ==\033[0;0m"
	install -d -m0755 $(DESTDIR)$(datadir)/cfg2html/
	cp -a cfg2html-linux.sh $(DESTDIR)$(datadir)/cfg2html/
	cp -aLr lib $(DESTDIR)$(datadir)/cfg2html/
	cp -a plugins $(DESTDIR)$(datadir)/cfg2html/
	-find $(DESTDIR)$(datadir)/cfg2html/ -name '.gitignore' -exec rm -rf {} \; &>/dev/null

install-var:
	@echo -e "\033[1m== Installing working directory ==\033[0;0m"
	install -d -m0755 $(DESTDIR)$(localstatedir)/log/cfg2html/

install-doc:
	@echo -e "\033[1m== Installing documentation ==\033[0;0m"
	make -C doc install
	sed -i -e 's,/etc,$(sysconfdir),' \
		-e 's,/usr/sbin,$(sbindir),' \
		-e 's,/usr/share,$(datadir),' \
		-e 's,/usr/share/doc/packages,$(datadir)/doc,' \
		$(DESTDIR)$(mandir)/man8/cfg2html.8

install-cron: cron
	echo -e "\033[1m== Installing cron ==\033[0;0m"
	install -Dp -m0644 ../$(name).cron $(DESTDIR)$(sysconfdir)/cron.d/$(name)
	rm -f ../$(name).cron

install: validate man install-config rewrite install-bin restore install-data install-var install-doc install-cron

uninstall:
	@echo -e "\033[1m== Uninstalling Config2HTML ==\033[0;0m"
	-rm -v $(DESTDIR)$(sbindir)/cfg2html
	-rm -v $(DESTDIR)$(mandir)/man8/cfg2html.8
	-rm -rv $(DESTDIR)$(datadir)/cfg2html/
#	rm -rv $(DESTDIR)$(sysconfdir)/cfg2html/
#	rm -rv $(DESTDIR)$(localstatedir)/lib/cfg2html/

dist: clean validate man rewrite $(name)-$(distversion).tar.gz restore

$(name)-$(distversion).tar.gz:
	@echo -e "\033[1m== Building archive $(name)-$(distversion) ==\033[0;0m"
	cd .. ; \
	git checkout $(git_branch) ; \
	git ls-tree -r --name-only --full-tree $(git_branch) | grep -v -E '(^bin|^hpux|^doc|^aix|^bsd|^sunos)' | \
		tar -czf $(name)-$(distversion).tar.gz --transform='s,^,$(name)-$(distversion)/,S' --files-from=-

Linux:
	case $T in deb) make deb;; rpm) make rpm;; *) make help;; esac

rpm: dist
	@echo -e "\033[1m== Building RPM package $(name)-$(distversion) ==\033[0;0m"
	cd .. ; \
	rpmbuild -tb  \
		--define "_rpmfilename %%{NAME}-%%{VERSION}-%%{RELEASE}.%%{ARCH}.rpm" \
		--define "_sourcedir %(pwd)" \
		--define "debug_package %{nil}" \
		--define "_rpmdir %(pwd)" $(name)-$(distversion).tar.gz

deb: dist cron doc
	@echo -e "\033[1m== Building DEB package $(name)-$(distversion) ==\033[0;0m"
	cp -r packaging/debian/ .
	chmod 755 debian/rules
	fakeroot debian/rules clean
	fakeroot dh_install
	fakeroot debian/rules binary
	-rm -rf debian/

pacman: BUILD_DIR = /tmp/cfg2html-$(distversion)
pacman: dist
	@echo -e "\033[1m== Building Pacman package $(name)-$(distversion) ==\033[0;0m"
	rm -rf $(BUILD_DIR)
	mkdir -p $(BUILD_DIR)
	cp packaging/arch/PKGBUILD.local $(BUILD_DIR)/PKGBUILD
	cp $(name)-$(distversion).tar.gz $(BUILD_DIR)/
	cd $(BUILD_DIR) ; sed -i -e 's/VERSION/$(date)/' \
		-e 's/SOURCE/$(name)-$(distversion).tar.gz/' \
		-e 's/MD5SUM/$(shell md5sum $(name)-$(distversion).tar.gz | cut -d' ' -f1)/' \
		PKGBUILD ; makepkg -c
	cp $(BUILD_DIR)/*.pkg.* .
	rm -rf $(BUILD_DIR)

obs: BUILD_DIR = /tmp/cfg2html-$(distversion)
obs: obsname = $(shell osc ls $(obsproject) $(obspackage) | awk '/.tar.gz$$/ { gsub(".tar.gz$$","",$$1); print }')
obs: dist doc cron
	@echo -e "\033[1m== Updating OBS from $(obsname) to $(name)-$(distversion)== \033[0;0m"
ifneq ($(obsname),$(name)-$(distversion))
	-rm -rf $(BUILD_DIR)
	mkdir -p $(BUILD_DIR)
ifneq ($(OFFICIAL),)
#	osc rdelete -m 'Recreating branch $(obspackage)' $(obsproject) $(obspackage)
	-osc branch home:gdha cfg2html $(obsproject) $(obspackage)
	-osc detachbranch $(obsproject) $(obspackage)
endif
	(cd $(BUILD_DIR) ; osc co -c $(obsproject) $(obspackage) )
	-(cd $(BUILD_DIR)/$(obspackage) ; osc del *.tar.gz )
	cp ../$(name)-$(distversion).tar.gz $(BUILD_DIR)/$(obspackage)
	cp ../$(name).cron $(BUILD_DIR)/$(obspackage)
	tar -xOzf ../$(name)-$(distversion).tar.gz -C $(BUILD_DIR)/$(obspackage) $(name)-$(distversion)/linux/$(specfile) >$(BUILD_DIR)/$(obspackage)/$(name).spec
	tar -xOzf ../$(name)-$(distversion).tar.gz -C $(BUILD_DIR)/$(obspackage) $(name)-$(distversion)/linux/$(dscfile) >$(BUILD_DIR)/$(obspackage)/$(name).dsc
	tar -xOzf ../$(name)-$(distversion).tar.gz -C $(BUILD_DIR)/$(obspackage) $(name)-$(distversion)/linux/packaging/debian/control >$(BUILD_DIR)/$(obspackage)/debian.control
	tar -xOzf ../$(name)-$(distversion).tar.gz -C $(BUILD_DIR)/$(obspackage) $(name)-$(distversion)/linux/packaging/debian/compat >$(BUILD_DIR)/$(obspackage)/debian.compat
	tar -xOzf ../$(name)-$(distversion).tar.gz -C $(BUILD_DIR)/$(obspackage) $(name)-$(distversion)/linux/packaging/debian/rules >$(BUILD_DIR)/$(obspackage)/debian.rules
	echo -e "cfg2html ($(version)-$(debrelease)) stable; urgency=low\n\n  * new snapshot build\n\n -- OpenSUSE Build System <obs@cfg2html> $$(date -R)" >$(BUILD_DIR)/$(obspackage)/debian.changelog
	tar -xOzf ../$(name)-$(distversion).tar.gz -C $(BUILD_DIR)/$(obspackage) $(name)-$(distversion)/linux/packaging/debian/changelog >>$(BUILD_DIR)/$(obspackage)/debian.changelog
	cd $(BUILD_DIR)/$(obspackage); osc addremove
	cd $(BUILD_DIR)/$(obspackage); osc ci -m "Update to $(name)-$(distversion)" $(BUILD_DIR)/$(obspackage)
	rm -rf $(BUILD_DIR)
	@echo -e "\033[1mNow visit https://build.opensuse.org/package/show?package=cfg2html&project=$(obsproject)"
	@echo -e "or inspect the queue at: https://build.opensuse.org/monitor\033[0;0m"
else
	@echo -e "OBS already updated to this release."
endif
