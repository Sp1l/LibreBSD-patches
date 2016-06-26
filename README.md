# LibreBSD
Location for the patch-sets to replace OpenSSL with LibreSSL in FreeBSD

![LibreBSD](https://cloud.githubusercontent.com/assets/7547697/13683368/9a2d31f0-e706-11e5-8c72-4f66273040ac.png)

**Before you ask:** This will not be a fork! I intend to maintain this as a patch-set for the most recent release of [FreeBSD](https://freebsd.org) and will maintain it for [HardenedBSD](https://hardenedbsd.org) as well.

Over the past weekend I managed to get LibreSSL to build, and all binaries to link to it on HardenedBSD. The patches were created on a derivative of the -to be released later this year- FreeBSD 11. See my earlier blog-posts for more details ([Part I](/libressl/2016-03-05/libressl-in-hardenedbsd-base-part-i.html) and [Part II](/libressl/2016-03-06/libressl-in-hardenedbsd-base-part-ii.html)).

I had tried to replace OpenSSL in FreeBSD 10 when I was at OpenBSD's LibreSSL hackathon in Varaždin (Croatia) last year but hadn't managed to complete the project. The release of LibreSSL 2.3 also removed SSLv3 so my attention was on fixing fallout from that removal. 'Evidence' of that work and the patches can be found in [the No-SSLv3](https://wiki.freebsd.org/OpenSSL/No-SSLv3) wiki article. As it turned out this time, it wasn't extremely difficult to do so I thought it wouldn't take too much time to do this for FreeBSD 10 as well. FreeBSD 10.3 is nearing its completion, so where better to start than with the current first Release Candidate!

**Feedback appreciated:** I haven't replayed all the steps here, do let me know where I've hidden my typos and mistakes! (email, Twitter, GitHub, Facebook, avionary)

# The 'recipe'

**You'll need to select the correct branch for your FreeBSD version**

1. Download the [LibreSSL 2.3 tarball](http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-2.3.2.tar.gz)
  * Extract this tarball into /usr/src/crypto and rename the directory from `libressl-2.3.2` to `libressl`
2. Apply the patch-set from [my GitHub repo](https://github.com/Sp1l/LibreBSD/tree/FreeBSD-10.3/patchset)
3. Add WITH_LIBRESSL=yes to /etc/src.conf
4. Rebuild and install your kernel and world (see the [FreeBSD handbook chapter](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/makeworld.html) for detail)
5. Reboot

## Commands

As commands (assuming you already have checked out FreeBSD 10.3 into /usr/src)

	#!sh
	cd ~
	mkdir download && cd download
	fetch http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-2.4.1.tar.gz
	fetch https://github.com/Sp1l/LibreBSD/raw/FreeBSD-10.3/patchset/patchset
	cd /usr/src/crypto
	tar xf ~/download/libressl-2.4.1.tar.gz
	mv libressl-2.4.1 libressl
	cd /usr/src
	patch < ~/download/patchset
	echo 'WITH_LIBRESSL=yes' >> /etc/src.conf
	make buildworld && make buildkernel && make installkernel && make installworld
	reboot

Line 3: You should verify the tarball using `signify` or `gpg`.	
Line 11: This should take quite a lot of time (probably hours) and is NOT the canonical way to do this. See the handbook [chapter on rebuilding your system](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/makeworld.html) for a complete description!	

Now that was easy wasn't it?

## Update your ports

After upgrading the kernel and world you'll need to rebuild all ports. If before you had defined

	:::make
	WITH_OPENSSL_PORT= yes
	OPENSSL_PORT=	security/libressl-devel

you can now remove these bits, but then you should rebuild world and kernel after every update of LibreSSL. Unless the shared library version -and thus the ABI- stay the same. ## Updating LibreSSL

If LibreSSL receives an update that has the same shared library version, you can use my guidance from [the FreeBSD wiki](https://wiki.freebsd.org/BernardSpil/PartialWorldBuilds) after downloading/extracting the latest LibreSSL tarball as discussed in the previous paragraph.

	#!sh
	cd /usr/src/secure/lib/libcrypto
	make obj && make depend && make includes && make
	make install
	cd /usr/src/secure/lib/libssl
	make clean && make depend && make includes && make
	make install
	cd /usr/src/secure/usr.bin/openssl
	make clean && make
	make install


# The detail

Next to the patchset, I've also added all the files that were changed to my GitHub repo. The files are in their original location so you can use these as an overlay for your `/usr/src`.

## LibreSSL patches

FreeBSD 11 changed quite a lot in the build framework, so I had to adapt the patches for libcrypto, libssl and openssl accordingly. This made the build for the `openssl` binary fail, so I had to change

	:::make
	LIBADD+= crypto ssl

into

	:::make
	DPADD=  ${LIBSSL} ${LIBCRYPTO}
	LDADD=  -lssl -lcrypto

The bulk of the patches I created for HardenedBSD just worked just fine on 10.3 

## base software patches

Most of the patches that I created for HardenedBSD applied cleanly.

1. The patches for `libtelnet` and `ppp` worked fine.
2. The `wpa` patches are not required, in 10.3 there's a much older version that doesn't have all the OpenSSL version checks.
3. The `heimdal` patches I've not yet tested but these patches.
