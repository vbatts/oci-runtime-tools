ARCHES ?= amd64

all: $(ARCHES:%=rootfs-%.tar.gz)

rootfs-%.tar.gz: rootfs/%/bin/echo
	tar -czf $@ -C rootfs/$* .

.PRECIOUS: rootfs/%/bin/busybox
rootfs/%/bin/busybox: downloads/stage3-%-current.tar.bz2 rootfs-files
	gpg --verify $<.DIGESTS.asc
	(cd downloads && \
		grep -A1 '^# SHA512 HASH' stage3-$*-current.tar.bz2.DIGESTS.asc | \
		grep -v '^--' | \
		sha512sum -c)
	sudo rm -rf rootfs/$*
	sudo mkdir -p rootfs/$*
	sudo tar -xvf downloads/stage3-$*-current.tar.bz2 -C rootfs/$* \
		--no-recursion --wildcards $$(< rootfs-files)
	sudo touch $@

.PRECIOUS: rootfs/%/bin/echo
rootfs/%/bin/echo: rootfs/%/bin/busybox
	sudo sh -c 'COMMANDS=$$($< --list) || exit 1; for COMMAND in $${COMMANDS}; do \
		test -L "rootfs/$*/bin/$${COMMAND}" || ln -rs $< "rootfs/$*/bin/$${COMMAND}" || exit; \
	done'

downloads/stage3-%-current.tar.bz2: get-stage3.sh
	STAGE3_ARCH=$* ./$<
	touch downloads/stage3-$*-*.tar.bz2

clean:
	rm -f downloads/*
	sudo rm -rf rootfs
