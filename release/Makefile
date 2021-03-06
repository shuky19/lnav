
VERSION=0.8.1

SRC_VERSION=master

outbox:
	mkdir -p $@

clean-outbox: outbox
	rm -f outbox/*

PACKAGE_URLS = \
	http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.gz \
	http://ftp.gnu.org/gnu/automake/automake-1.15.tar.gz \
	https://ftp.gnu.org/gnu/make/make-4.2.1.tar.gz \
	ftp://invisible-island.net/ncurses/ncurses-5.9.tar.gz \
	ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz \
	ftp://ftp.cwru.edu/pub/bash/readline-6.3.tar.gz \
	http://zlib.net/zlib-1.2.8.tar.gz \
	http://www.bzip.org/1.0.6/bzip2-1.0.6.tar.gz \
	http://sqlite.org/2016/sqlite-autoconf-3130000.tar.gz \
	http://openssl.skazkaforyou.com/source/openssl-1.0.1t.tar.gz \
	http://www.libssh2.org/download/libssh2-1.7.0.tar.gz \
	http://curl.haxx.se/download/curl-7.48.0.tar.gz

.PHONY: linux freebsd pkger download-pkgs

%-vm: %
	cd vagrant-static && vagrant up $<

download-pkgs:
	mkdir -p vagrant-static/pkgs && cd vagrant-static/pkgs && \
	for pkg in $(PACKAGE_URLS); do \
		wget -N $${pkg}; \
	done

%-build: % %-vm
	cd vagrant-static && vagrant ssh $< -c "/vagrant/build.sh ${SRC_VERSION}"

%-package: % clean-outbox %-build
	mkdir -p $<-pkg/lnav-${VERSION}
	cp ../README ../NEWS $<-pkg/lnav-${VERSION}
	mv vagrant-static/lnav $<-pkg/lnav-${VERSION}
	cd $<-pkg && zip -r ../outbox/lnav-${VERSION}-$<-64bit.zip lnav-${VERSION}
	rm -rf $<-pkg

linux-ospkg: pkger-vm linux-package
	rm -f vagrant-static/lnav\*.deb vagrant-static/lnav\*.rpm
	cd vagrant-static && vagrant ssh pkger -c "/vagrant/build-pkg.sh ${VERSION}"
	mv vagrant-static/lnav*.deb vagrant-static/lnav*.rpm outbox/

osx-build:
	rm -rf osx-build-dir
	mkdir -p osx-build-dir
	cd ../ && ./autogen.sh
	cd osx-build-dir && ../../configure && make -j && make distcheck -j8 && make dist-bzip2

osx-package: clean-outbox osx-build
	mkdir -p osx-pkg/lnav-${VERSION}
	cp ../README ../NEWS osx-pkg/lnav-${VERSION}
	cp osx-build-dir/src/lnav osx-pkg/lnav-${VERSION}
	cp osx-build-dir/lnav-${VERSION}.tar.gz outbox/
	cp osx-build-dir/lnav-${VERSION}.tar.bz2 outbox/
	cd osx-pkg && zip -r ../outbox/lnav-${VERSION}-os-x.zip lnav-${VERSION}
	rm -rf osx-pkg

release: linux-ospkg freebsd-package osx-package

push:
	env LANG=UTF-8 package_cloud push tstack/lnav/ubuntu/lucid outbox/lnav*.deb
	env LANG=UTF-8 package_cloud push tstack/lnav/el/5 outbox/lnav-0.8.1-1.x86_64.rpm

clean:
	cd vagrant-static && vagrant destroy -f
	rm -rf vagrant-static/pkgs
