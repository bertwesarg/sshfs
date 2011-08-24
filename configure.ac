AC_INIT(sshfs, 2.8)
AC_CANONICAL_TARGET
AM_INIT_AUTOMAKE([foreign subdir-objects])
AM_CONFIG_HEADER(config.h)

AC_PROG_CC
AM_PROG_CC_C_O
AC_CANONICAL_HOST
CFLAGS="$CFLAGS -Wall -W"
LIBS=
AC_SEARCH_LIBS(dlsym, [dl])
sshnodelay_libs=$LIBS
AC_SUBST(sshnodelay_libs)
LIBS=

case "$target_os" in
    *linux*) osname=linux;;
    *darwin*) osname=darwin;;
    *) osname=unknown;;
esac

AC_ARG_ENABLE(sshnodelay,
	[  --disable-sshnodelay    Don't compile NODELAY workaround for ssh])

if test -z "$enable_sshnodelay"; then
	AC_MSG_CHECKING([OpenSSH version])
	[eval `ssh -V 2>&1 | sed -n 's/^OpenSSH_\([1-9][0-9]*\)\.\([0-9][0-9]*\).*/ssh_major=\1 ssh_minor=\2/p'`]
	if test "x$ssh_major" != x -a "x$ssh_minor" != x; then
		if test $ssh_major -gt 4 -o \( $ssh_major = 4 -a $ssh_minor -ge 4 \); then
			AC_MSG_RESULT([$ssh_major.$ssh_minor >= 4.4, disabling NODELAY workaround])
			enable_sshnodelay=no
		else
			AC_MSG_RESULT([$ssh_major.$ssh_minor < 4.4, enabling NODELAY workaround])
			enable_sshnodelay=yes
		fi
	else
		AC_MSG_RESULT([not found])
	fi
fi

if test "$enable_sshnodelay" = "yes"; then
	AC_DEFINE(SSH_NODELAY_WORKAROUND, 1, [Compile ssh NODELAY workaround])
fi

AM_CONDITIONAL(SSH_NODELAY_SO, test "$enable_sshnodelay" = "yes")

export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
PKG_CHECK_MODULES([SSHFS], [fuse >= 2.3 glib-2.0 gthread-2.0])
have_fuse_opt_parse=no
oldlibs="$LIBS"
LIBS="$LIBS $SSHFS_LIBS"
AC_CHECK_FUNC([fuse_opt_parse], [have_fuse_opt_parse=yes])
LIBS="$oldlibs"
if test "$have_fuse_opt_parse" = no -o "$osname" = darwin; then
	CFLAGS="$CFLAGS -I${srcdir}/compat"
fi
AM_CONDITIONAL(FUSE_OPT_COMPAT, test "$have_fuse_opt_parse" = no)
AM_CONDITIONAL(DARWIN_COMPAT, test "$osname" = darwin)

AC_CHECK_PROG(UNMOUNT_COMMAND, fusermount, fusermount -u, umount)

# TODO: Figure out why we special-case this in Darwin.  Would be nice if
# the default setting was consistent across platforms so we wouldn't need
# to care about it here.
case "$osname" in
    darwin) IDMAP_DEFAULT=user ;;
    *) IDMAP_DEFAULT=none ;;
esac
AC_SUBST(IDMAP_DEFAULT)

AC_CHECK_HEADERS([poll.h sys/pstat.h sys/un.h vis.h])
AC_CHECK_FUNCS([sendmsg recvmsg setproctitle strnvis vsnprintf])

dnl make sure we're using the real structure members and not defines
AC_CACHE_CHECK([for msg_accrights field in struct msghdr],
		ac_cv_have_accrights_in_msghdr, [
	AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/uio.h>
		]], [[
#ifdef msg_accrights
#error "msg_accrights is a macro"
exit(1);
#endif
struct msghdr m;
m.msg_accrights = 0;
exit(0);
		]])],
		[ ac_cv_have_accrights_in_msghdr="yes" ],
		[ ac_cv_have_accrights_in_msghdr="no" ]
	)
])
if test "x$ac_cv_have_accrights_in_msghdr" = "xyes" ; then
	AC_DEFINE([HAVE_ACCRIGHTS_IN_MSGHDR], [1],
		[Define if your system uses access rights style
		file descriptor passing])
fi

AC_CACHE_CHECK([for msg_control field in struct msghdr],
		ac_cv_have_control_in_msghdr, [
	AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/uio.h>
		]], [[
#ifdef msg_control
#error "msg_control is a macro"
exit(1);
#endif
struct msghdr m;
m.msg_control = 0;
exit(0);
		]])],
		[ ac_cv_have_control_in_msghdr="yes" ],
		[ ac_cv_have_control_in_msghdr="no" ]
	)
])
if test "x$ac_cv_have_control_in_msghdr" = "xyes" ; then
	AC_DEFINE([HAVE_CONTROL_IN_MSGHDR], [1],
		[Define if your system uses ancillary data style
		file descriptor passing])
fi

AC_CACHE_CHECK([if libc defines __progname], ac_cv_libc_defines___progname, [
	AC_LINK_IFELSE([AC_LANG_PROGRAM([[]],
		[[ extern char *__progname; printf("%s", __progname); ]])],
	[ ac_cv_libc_defines___progname="yes" ],
	[ ac_cv_libc_defines___progname="no"
	])
])
if test "x$ac_cv_libc_defines___progname" = "xyes" ; then
	AC_DEFINE([HAVE___PROGNAME], [1], [Define if libc defines __progname])
fi

# Check for some target-specific stuff
case "$host" in
*-*-aix*)
	AC_DEFINE([SPT_TYPE], [SPT_REUSEARGV],
		[Define to a Set Process Title type if your system is
		supported by bsd-setproctitle.c])
	;;
*-*-cygwin*)
	;;
*-*-dgux*)
	;;
*-*-darwin*)
	;;
*-*-dragonfly*)
	;;
*-*-haiku*)
	;;
*-*-hpux*)
	AC_DEFINE([SPT_TYPE], [SPT_PSTAT])
	;;
*-*-irix5*)
	;;
*-*-irix6*)
	;;
*-*-k*bsd*-gnu | *-*-kopensolaris*-gnu)
	AC_DEFINE([SPT_TYPE], [SPT_REUSEARGV])
	;;
*-*-linux*)
	AC_DEFINE([SPT_TYPE], [SPT_REUSEARGV])
	;;
mips-sony-bsd|mips-sony-newsos4)
	;;
*-*-netbsd*)
	;;
*-*-freebsd*)
	;;
*-*-bsdi*)
	;;
*-next-*)
	;;
*-*-openbsd*)
	;;
*-*-solaris*)
	;;
*-*-sunos4*)
	;;
*-ncr-sysv*)
	;;
*-sni-sysv*)
	;;
# UnixWare 1.x, UnixWare 2.x, and others based on code from Univel.
*-*-sysv4.2*)
	;;
# UnixWare 7.x, OpenUNIX 8
*-*-sysv5*)
	;;
*-*-sysv*)
	;;
# SCO UNIX and OEM versions of SCO UNIX
*-*-sco3.2v4*)
	;;
# SCO OpenServer 5.x
*-*-sco3.2v5*)
	;;
*-*-unicosmk*)
	;;
*-*-unicosmp*)
	;;
*-*-unicos*)
	;;
*-dec-osf*)
	;;
*-*-nto-qnx*)
	;;
*-*-ultrix*)
	;;
*-*-lynxos)
	;;
esac

AC_CONFIG_FILES([Makefile])
AC_OUTPUT
