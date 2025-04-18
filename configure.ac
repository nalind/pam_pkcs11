# $Id$

# Require autoconf 2.52
AC_PREREQ([2.69])

# Process this file with autoconf to produce a configure script.
AC_INIT([pam_pkcs11],[0.6.13])
AC_CONFIG_SRCDIR([src/pam_pkcs11/pam_pkcs11.c])
AC_CANONICAL_HOST
AM_INIT_AUTOMAKE
AC_CONFIG_HEADERS([config.h])

# silent build by default
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

AM_MAINTAINER_MODE

AM_GNU_GETTEXT([external])

dnl Add the languages which your application supports here.
ALL_LINGUAS="de fr it ka nl pl pt_BR ru tr zh_CN"

# Checks for programs.
AC_PROG_CC
AC_PROG_CXX
AM_PROG_AR
LT_INIT
AC_PROG_INSTALL
AC_PROG_LN_S
AM_PROG_LEX
AC_PROG_MAKE_SET

# check enable_dependency_tracking configure option
AM_CONDITIONAL([AMDEP], [test "x$enable_dependency_tracking" != xno])

# Disable pointer-sign warnings for a while...
CFLAGS="$CFLAGS -Wno-pointer-sign"

# Special check for pthread support
AX_PTHREAD(, [ AC_MSG_ERROR([POSIX thread support required]) ])
CC="$PTHREAD_CC"

# Argument for debugging
AC_ARG_ENABLE(debug, 
  AS_HELP_STRING([--disable-debug],[remove debugging code]))
if test "$enable_debug" = "no" -o "$enable_debug" = "false"
then
  AC_MSG_WARN([Debugging support is completely disabled!])
  with_debug=no
else
  AC_DEFINE(DEBUG, 1, [Define to 1 if you want to include debugging code.])
  with_debug=yes
fi

# check for dlopen() in libdl
AC_SUBST(LIBDL)
AC_CHECK_LIB(dl, dlopen,
	[LIBDL="$LIBDL -ldl"],
	[AC_CHECK_FUNCS(dlopen,, [AC_MSG_ERROR(dlopen() not found)])])

# Add argument for using curl
AC_ARG_WITH(curl,
  AS_HELP_STRING([--with-curl],[use curl (default=no)]))
if test "$with_curl" = "yes" -o "$with_curl" = "true"
then
  AC_CHECK_LIB(curl, curl_easy_init, 
	[ 
	with_curl=yes 
	LIBS="$LIBS -lcurl"
	],[
	AC_MSG_WARN(Could not locate cURL library. cURL disabled)
  	with_curl=no
	]) 
else
  with_curl=no
fi
AC_SUBST(CURL_CFLAGS)
AC_SUBST(CURL_LIBS)
AM_CONDITIONAL(HAVE_CURL, test x$with_curl = xyes)

# Add argument for defining configuration directory
AC_ARG_WITH(confdir,
AS_HELP_STRING([--with-confdir=DIR],[directory containing pam_pkcs11.conf (default /etc/pam_pkcs11)]),
	[confdir="${with_confdir}"], [confdir=false])
if test x${confdir} = xfalse ; then
	confdir="/etc/pam_pkcs11"
fi
AC_DEFINE_UNQUOTED(CONFDIR, "$confdir", [directory containing pam_pkcs11.conf (default /etc/pam_pkcs11)])
AC_SUBST(confdir)

# Add argument for using ldap
AC_ARG_WITH(ldap,
  AS_HELP_STRING([--with-ldap],[use ldap (default=yes)]))
if test "$with_ldap" = "no" -o "$with_ldap" = "false"
then
  with_ldap=no
else
  AC_CHECK_LIB( ldap, ldap_init,
	[
	with_ldap=yes
	LIBS="$LIBS -lldap"
	],[
	AC_MSG_WARN([Cannot find LDAP libraries. LDAP support disabled])
	with_ldap=no
	])
fi
if test "$with_ldap" \!= "no"; then
  AC_CHECK_FUNCS(ldap_init ldap_initialize)
  AC_CHECK_FUNCS(ldap_set_option ldap_get_option)  
  AC_CHECK_FUNCS(ldap_start_tls ldap_start_tls_s)
fi	
AC_SUBST(LDAP_CFLAGS)
AC_SUBST(LDAP_LIBS)
AM_CONDITIONAL(HAVE_LDAP, test x$with_ldap = xyes)

# Add argument for using libp11
#AC_ARG_WITH(libp11,
#  AS_HELP_STRING([--with-libp11],[use libp11 (default=yes)]))
#if test "$with_libp11" = "no" -o "$with_libp11" = "false"
#then
#  with_libp11=no
#else
#  AC_CHECK_LIB( p11, PKCS11_CTX_new,
#	[
#	with_libp11=yes
#	LIBS="$LIBS -lp11"
#	],[
#	AC_MSG_WARN([Cannot find LIBP11 library. Using direct Pkcs11 access])
#	with_libp11=no
#	])
#fi
#AC_SUBST(LIBP11_CFLAGS)
#AC_SUBST(LIBP11_LIBS)
#AM_CONDITIONAL(HAVE_LIBP11, test x$with_libp11 = xyes)

PKG_PROG_PKG_CONFIG

# Check and set crypto paths
AC_ARG_WITH(nss,
  AS_HELP_STRING([--with-nss],[use NSS instead of openSSL and raw PKCS 11 (default=no)]))
if test "$with_nss" = "yes" -o "$with_nss" = "true"
then
  PKG_CHECK_MODULES(NSS, nss, [with_nss=yes],
   [ AC_MSG_ERROR(could not locate NSS crypto library) ] )
  OPENSSL_CFLAGS=
  OPENSSL_LIBS=
  AC_SUBST(OPENSSL_CFLAGS)
  AC_SUBST(OPENSSL_LIBS)
  with_ssl=no
else
   PKG_CHECK_MODULES(OPENSSL, openssl, [with_ssl=yes ],
    [ AC_MSG_ERROR(openssl not found) ])
  NSS_CFLAGS=
  NSS_LIBS=
  AC_SUBST(NSS_CFLAGS)
  AC_SUBST(NSS_LIBS)
  with_nss=no
fi
AM_CONDITIONAL(HAVE_NSS, test x$with_nss = xyes)
if test "$with_nss" = "yes"
then
    CRYPTO_CFLAGS="${NSS_CFLAGS} -DHAVE_NSS"
    CRYPTO_LIBS=${NSS_LIBS}
else
    CRYPTO_CFLAGS=${OPENSSL_CFLAGS}
    CRYPTO_LIBS=${OPENSSL_LIBS}
fi
AC_SUBST(CRYPTO_CFLAGS)
AC_SUBST(CRYPTO_LIBS)


# Check for PCSC-Lite
AC_ARG_WITH(pcsclite,
  AS_HELP_STRING([--without-pcsclite],[Do not use pcsc-lite (default=yes)]),
  [], [ with_pcsclite=yes ])

if test "$with_pcsclite" != "no"; then
	OLD_CFLAGS="$CFLAGS"
	OLD_LIBS="$LIBS"
	with_pcsclite=yes

    PKG_CHECK_MODULES(PCSC, [libpcsclite], ,
	[ if test -f /usr/local/lib/pkgconfig/libpcsclite.pc ; then
		AC_MSG_ERROR([use PKG_CONFIG_PATH=/usr/local/lib/pkgconfig ./configure])
	fi
    ])

	CFLAGS="$CFLAGS $PCSC_CFLAGS"
	AC_CHECK_HEADER(winscard.h,,
		[AC_MSG_ERROR([winscard.h not found, install pcsc-lite, or use PCSC_CFLAGS=... ./configure])],
		[ #include <winscard.h> ])

	LIBS="$LDLIBS $PCSC_LIBS"
	AC_CHECK_FUNC(SCardEstablishContext, ,
		[AC_MSG_ERROR([SCardEstablishContext() not found, install pcsc-lite or later,or use PCSC_LIBS=... ./configure])])

	LIBS="$OLD_LIBS"
	CFLAGS="$OLD_CFLAGS"
else
	AC_MSG_WARN([pcsc-lite disabled])
fi
AC_SUBST(PCSC_CFLAGS)
AC_SUBST(PCSC_LIBS)
AM_CONDITIONAL(HAVE_PCSC, test "x$with_pcsclite" = "xyes")

# Check for SGML processor
AC_ARG_WITH(docbook,
  AS_HELP_STRING([--without-docbook],[do not generate html manual (needs docbook)]),
        with_docbook=$withval)
if test "$with_docbook" = "no" -o "$with_docbook" = "false"
then
        with_docbook=no
else
    AC_CHECK_PROG(XSLTPROC, xsltproc, xsltproc)
    if test "x$XSLTPROC" = "x"; then
        if test "x$with_docbook" = "xyes"; then
                AC_MSG_ERROR([Docbook support requested, but cannot find xsltproc])
        fi
        with_docbook=no
    else
	with_docbook=yes
    fi
fi
AM_CONDITIONAL(HAVE_DOCBOOK, test "x$with_docbook" != "xno")

# Checks for libraries.
AC_CHECK_LIB(pam, pam_get_item, , AC_MSG_ERROR(could not locate pam libraries))

# Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS([string.h syslog.h fcntl.h unistd.h error.h])
if test "x$with_ldap" = "xyes"; then
AC_CHECK_HEADERS([ldap.h])
fi
if test "x$with_curl" = "xyes"; then
AC_CHECK_HEADERS([curl/curl.h])
fi
if test "x$with_libp11" = "xyes"; then
AC_CHECK_HEADERS([libp11.h])
fi
AC_CHECK_HEADERS([security/pam_ext.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_TYPE_SIZE_T

# Checks for library functions.
AC_FUNC_MALLOC
AC_FUNC_REALLOC
AC_FUNC_STAT
AC_FUNC_VPRINTF
AC_CHECK_FUNCS([memset strdup strerror strndup daemon])


AC_CONFIG_FILES([ 
Makefile
po/Makefile.in
doc/Makefile
doc/pam_pkcs11.8
doc/doxygen.conf
etc/Makefile
etc/pam.d_login.example
etc/pam_pkcs11.conf.example
src/Makefile
src/scconf/Makefile
src/common/Makefile
src/common/rsaref/Makefile
src/tools/Makefile
src/mappers/Makefile
src/pam_pkcs11/Makefile
tools/Makefile
])
AC_OUTPUT

A=`eval echo ${prefix}` ; A=`eval echo ${A}`
B=`eval echo ${bindir}` ; B=`eval echo ${B}`
C=`eval echo ${sysconfdir}` ; C=`eval echo ${C}`
D=`eval echo ${libdir}` ; D=`eval echo ${D}`

echo ""
echo "PAM-PKCS#11 has been configured with the following options"

echo ""
echo "Version:             ${PACKAGE_VERSION}"
echo "User binaries:       ${B}"
echo "Configuration files: ${C}"
echo "libdir:              ${D}"
echo ""
echo "Host:                ${host}"
echo "Compiler:            ${CC}"
echo "Compiler flags:      ${CFLAGS}"
echo "Preprocessor flags:  ${CPPFLAGS}"
echo "Linker flags:        ${LDFLAGS}"
echo "Libraries:           ${LIBS}"

echo ""
echo "Debugging:           ${with_debug}"
echo "DocBook support:     ${with_docbook}"
echo "PC/SC support:       ${with_pcsclite}"
echo "CURL support:        ${with_curl}"
echo "LDAP support:        ${with_ldap}"
echo "NSS support:         ${with_nss}"
echo "OPENSSL support:     ${with_ssl}"
echo "confdir:             ${confdir}"
#echo "LIBP11 support:      ${with_libp11}"
echo ""
