# $FreeBSD: releng/11.0/secure/usr.bin/openssl/Makefile 290460 2015-11-06 17:39:17Z jkim $

.include <src.opts.mk>

.if ${MK_LIBRESSL} != "no"
PROG=	openssl

LIBADD=	ssl crypto

.if exists(Makefile.man)
.include "Makefile.man"
.endif
.include "../../lib/libcrypto/Makefile.inc"

CFLAGS+=	-DMONOLITH

SRCS+=	app_rand.c apps.c asn1pars.c ca.c ciphers.c cms.c crl.c crl2p7.c \
	dgst.c dh.c dhparam.c dsa.c dsaparam.c ec.c ecparam.c enc.c engine.c \
	errstr.c gendh.c gendsa.c genpkey.c genrsa.c nseq.c ocsp.c openssl.c \
	passwd.c pkcs12.c pkcs7.c pkcs8.c pkey.c pkeyparam.c pkeyutl.c \
	prime.c rand.c req.c rsa.c rsautl.c s_cb.c s_client.c s_server.c \
	s_socket.c s_time.c sess_id.c smime.c speed.c spkac.c srp.c ts.c \
	verify.c version.c x509.c

.include <bsd.prog.mk>

.PATH:	${LCRYPTO_SRC}/apps \
	${.CURDIR}/man

.else
.include "Makefile.libressl"
.endif # ${MK_LIBRESSL} != "no"
