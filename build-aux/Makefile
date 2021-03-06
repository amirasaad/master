
abs_top_srcdir = $(shell realpath ../)
abs_srcdir = $(shell realpath .)
SED = $(shell which sed)
PACKAGE_NAME = font-manager

# Project must be configured or some of these targets will fail
configure:
	cd $(abs_top_srcdir) && \
	test -e Makefile || ./autogen.sh && \
	cd $(abs_srcdir)

POTFILES.readme:
	echo "Do not edit POTFILES.* directly. See build-aux directory" > $(abs_top_srcdir)/po/$@

POTFILES.in:
	find $(abs_top_srcdir) -name "*.vala*" -print | $(SED) -e 's|$(abs_top_srcdir)/||g' -e 's/\.in//g' | sort | uniq > $(abs_top_srcdir)/po/$@ ;
	echo "[type: gettext/glade]data/ApplicationMenu.ui" >> $(abs_top_srcdir)/po/$@ ;
	echo "data/org.gnome.FontManager.appdata.xml.in" >> $(abs_top_srcdir)/po/$@ ;
	echo "data/org.gnome.FontManager.desktop.in" >> $(abs_top_srcdir)/po/$@ ;
	echo "" >> $(abs_top_srcdir)/po/$@ ;
	$(SED) -i 's|\.\./||g' $(abs_top_srcdir)/po/$@ ;

POTFILES.skip:
	find $(abs_top_srcdir) -name "*.vala*" -print | $(SED) -e 's|$(abs_top_srcdir)/||g' -e 's/\.in//g' | sort | uniq | $(SED) 's/.vala/.c/g' > $(abs_top_srcdir)/po/$@ ;
	$(SED) -i 's|\.\./||g' $(abs_top_srcdir)/po/$@ ;
	echo "lib/Glue/Enums.c" >> $(abs_top_srcdir)/po/$@ ;

POT: configure
	cd $(abs_top_srcdir) && \
	make $(PACKAGE_NAME).pot && \
	cd $(abs_srcdir)

write_sources = \
	for i in `find . -type f -name '*.vala*' -print | $(SED) 's/\.in//g' | sort | uniq`; \
	do \
		echo "$$i \\" >> $1  ; \
	done && \
	$(SED) -i -e 's/^\.\//	/g' -e '$$ s/\\//' $1

ValaSources.mk:
	cd $(abs_top_srcdir)/lib && \
	echo "# Do not edit directly. See build-aux directory" > $@  && \
	echo "" >> $@ && \
	echo "libfontmanager_la_VALASOURCES = \\" >> $@  && \
	$(call write_sources,$@) && \
	cd $(abs_top_srcdir)/src/$(PACKAGE_NAME) && \
	echo "# Do not edit directly. See build-aux directory" > $@  && \
	echo "" >> $@ && \
	echo "font_manager_VALASOURCES = \\" >> $@  && \
	$(call write_sources,$@) && \
	cd $(abs_srcdir)

License.h:
	license/genheader.py $(abs_top_srcdir)/lib/Glue

Vendor.h:
	vendor/genheader.py $(abs_top_srcdir)/lib/Glue

update-headers: License.h Vendor.h
update-po: POTFILES.readme POTFILES.in POTFILES.skip POT
update-sources: ValaSources.mk
update: update-headers update-po update-sources

# Fedora RPM packaging

rpm-prep: configure
	cd $(abs_top_srcdir) && \
	make dist-bzip2 && \
	rpmdev-setuptree && \
	cp *.bz2 ~/rpmbuild/SOURCES/ && \
	cp fedora/$(PACKAGE_NAME).spec ~/rpmbuild/SPECS/ && \
	rm -f ~/rpmbuild/SRPMS/$(PACKAGE_NAME)*.src.rpm && \
	cd $(abs_srcdir)

rpm: rpm-prep
	pkexec dnf builddep ~/rpmbuild/SPECS/$(PACKAGE_NAME).spec
	rpmbuild -bb ~/rpmbuild/SPECS/$(PACKAGE_NAME).spec

srpm: rpm-prep
	rpmbuild -bs ~/rpmbuild/SPECS/$(PACKAGE_NAME).spec

copr-build: srpm
	copr-cli build --nowait jerrycasiano/FontManager ~/rpmbuild/SRPMS/$(PACKAGE_NAME)*.src.rpm
