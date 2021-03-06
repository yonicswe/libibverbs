dnl Process this file with autoconf to produce a configure script.

AC_PREREQ([2.67])
AC_INIT([libibverbs],[1.1.8],[linux-rdma@vger.kernel.org])
AC_CONFIG_SRCDIR([src/ibverbs.h])
AC_CONFIG_AUX_DIR(config)
AC_CONFIG_MACRO_DIR(config)
AC_CONFIG_HEADER(config.h)
AM_INIT_AUTOMAKE([foreign subdir-objects])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

dnl Checks for programs
AC_PROG_CC
AC_USE_SYSTEM_EXTENSIONS
AC_PROG_LN_S
LT_INIT

AC_ARG_WITH([valgrind],
    AS_HELP_STRING([--with-valgrind],
        [Enable Valgrind annotations (small runtime overhead, default NO)]))
if test x$with_valgrind = x || test x$with_valgrind = xno; then
    want_valgrind=no
    AC_DEFINE([NVALGRIND], 1, [Define to 1 to disable Valgrind annotations.])
else
    want_valgrind=yes
    if test -d $with_valgrind; then
        CPPFLAGS="$CPPFLAGS -I$with_valgrind/include"
    fi
fi

dnl Checks for libraries
AC_CHECK_LIB(dl, dlsym, [],
    AC_MSG_ERROR([dlsym() not found.  libibverbs requires libdl.]))
AC_CHECK_LIB(pthread, pthread_mutex_init, [],
    AC_MSG_ERROR([pthread_mutex_init() not found.  libibverbs requires libpthread.]))

dnl Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADER(valgrind/memcheck.h,
    [AC_DEFINE(HAVE_VALGRIND_MEMCHECK_H, 1,
        [Define to 1 if you have the <valgrind/memcheck.h> header file.])],
    [if test $want_valgrind = yes; then
        AC_MSG_ERROR([Valgrind memcheck support requested, but <valgrind/memcheck.h> not found.])
    fi])
AC_CHECK_HEADERS([fcntl.h sys/socket.h])

dnl Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_CHECK_DECLS([O_CLOEXEC],,[AC_DEFINE([O_CLOEXEC],[0], [Defined to 0 if not provided])],
[[
#ifdef HAVE_FCNTL_H
# include <fcntl.h>
#endif
]])
AC_CHECK_DECLS([SOCK_CLOEXEC],,[AC_DEFINE([SOCK_CLOEXEC],[0],[Defined to 0 if not provided])],
[[
#ifdef HAVE_SYS_SOCKET_H
# include <sys/socket.h>
#endif
]])

AC_CACHE_CHECK(for close on exec modifier for fopen(), ac_cv_feature_stream_cloexec_flag,
   [if test $ac_cv_have_decl_O_CLOEXEC = yes ; then
        if test $ac_cv_have_decl_SOCK_CLOEXEC = yes ; then
            ac_cv_feature_stream_cloexec_flag="e"
        fi
    fi])
AC_DEFINE_UNQUOTED([STREAM_CLOEXEC], "$ac_cv_feature_stream_cloexec_flag", [fopen() modifier for setting close on exec flag])

AC_CACHE_CHECK(whether ld accepts --version-script, ac_cv_version_script,
    [if test -n "`$LD --help < /dev/null 2>/dev/null | grep version-script`"; then
	ac_cv_version_script=yes
    else
	ac_cv_version_script=no
    fi])

if test $ac_cv_version_script = yes; then
    LIBIBVERBS_VERSION_SCRIPT='-Wl,--version-script=$(srcdir)/src/libibverbs.map'
else
    LIBIBVERBS_VERSION_SCRIPT=
fi
AC_SUBST(LIBIBVERBS_VERSION_SCRIPT)

AC_CACHE_CHECK(for .symver assembler support, ac_cv_asm_symver_support,
    [AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[]],
        [[asm("symbol:\n.symver symbol, api@ABI\n");]])],
        [ac_cv_asm_symver_support=yes],
        [ac_cv_asm_symver_support=no])])
if test $ac_cv_asm_symver_support = yes; then
    AC_DEFINE([HAVE_SYMVER_SUPPORT], 1, [assembler has .symver support])
fi

AC_CONFIG_FILES([Makefile libibverbs.spec])
AC_OUTPUT
