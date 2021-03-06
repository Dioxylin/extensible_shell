dnl Process this file with autoconf to produce a configure script.
AC_INIT(access.c)
AC_CONFIG_HEADER(config.h)

AC_DEFINE([ABUSED_GETENV], [0], [Description])
AC_DEFINE([HAVE_DEV_FD], [0], [Description])
AC_DEFINE([KERNEL_POUNDBANG], [0], [Description])
AC_DEFINE([LIMIT_T], [0], [Description])
AC_DEFINE([NO_VA_LIST_ASSIGN], [0], [Description])
AC_DEFINE([SOLARIS], [0], [Description])
AC_DEFINE([VOID_SIGNALS], [0], [description])

dnl AC_CMDSTDOUT_CPP(variable, command, headers)
AC_DEFUN([AC_CMDSTDOUT_CPP],
[cat > conftest.$ac_ext <<EOF
[#]line __oline__ "configure"
#include "confdefs.h"
[$3]
EOF
$1=`(eval "$ac_cpp conftest.$ac_ext") 2>&AC_FD_CC | $2`
rm -f conftest*
])


use_readline=yes
use_editline=no

AC_ARG_WITH(readline,
--with-readline		Use GNU Readline, use_readline=yes)
AC_ARG_WITH(editline,
--with-editline		Use the Editline library, use_editline=yes)

#AC_CANONICAL_HOST

case "$host" in
	*sun5* | *solaris2*)
		AC_DEFINE(SOLARIS)
	;;
esac

dnl saved_CFLAGS="$CFLAGS"

dnl Checks for programs.
AC_PROG_CC
AC_PROG_INSTALL
AC_PROG_YACC

dnl CFLAGS="$CFLAGS $saved_CFLAGS"

dnl ----------------------------
dnl CHECK FOR /dev/fd FILESYSTEM
dnl ----------------------------
AC_CACHE_CHECK(for /dev/fd filesystem, es_cv_sys_dev_fd,
[test -d /dev/fd && es_cv_sys_dev_fd=yes || es_cv_sys_dev_fd=no])
if test $es_cv_sys_dev_fd = yes; then
  AC_DEFINE(HAVE_DEV_FD)
fi

AC_SYS_INTERPRETER
if test "$ac_cv_sys_interpreter" = yes
then
	AC_DEFINE(KERNEL_POUNDBANG)
fi


dnl Checks for libraries.

AC_CHECK_LIB(sun, getpwuid)

if test "$use_readline" = yes || test "$use_editline" = yes
then
	AC_CHECK_LIB(terminfo, main)
	AC_CHECK_LIB(termcap, main)
	if test "$use_readline" = yes
	then
		AC_CHECK_LIB(readline, readline)
	elif test "$use_editline" = yes
	then
		AC_CHECK_LIB(edit, readline)
	fi

fi


dnl Checks for header files.
AC_HEADER_DIRENT
AC_HEADER_STDC
AC_HEADER_SYS_WAIT
AC_CHECK_HEADERS(fcntl.h limits.h sys/ioctl.h sys/time.h unistd.h memory.h stdarg.h sys/cdefs.h)


dnl Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_TYPE_UID_T
AC_TYPE_SIZE_T

dnl Checks for library functions.
AC_TYPE_GETGROUPS
AC_PROG_GCC_TRADITIONAL
AC_FUNC_MMAP
AC_TYPE_SIGNAL

if test "$ac_cv_type_signal" = void
then
	AC_DEFINE(VOID_SIGNALS)
fi

AC_FUNC_WAIT3
AC_CHECK_FUNCS(strerror strtol lstat setrlimit sigrelse sighold sigaction \
sysconf setsid sigsetjmp)

AC_CACHE_CHECK(for an abused getenv, es_cv_abused_getenv,
AC_TRY_RUN(
[
changequote(,)
char *foo = 0;
char *foov[] = { "-a", "-bfoo", "bar" };
int fooc = (sizeof(foov) / sizeof(foov[0]));

int getenv() { return *foo; }
int main() { while (-1 != getopt(fooc, foov, "ab:")); exit(0); }
changequote([,])
], 
es_cv_abused_getenv=no,
[es_cv_abused_getenv=yes
AC_DEFINE(ABUSED_GETENV)]))

if test "$es_cv_abused_getenv" = yes
then
	rm -f core conftest.core
fi

dnl Check to see if you can assign to a va_list
AC_CACHE_CHECK(whether assignment to va_list ok?, es_cv_assign_va_list,
AC_TRY_COMPILE(
[#ifndef HAVE_STDARG_H
choke me
#else
#include <stdarg.h>
#endif],

[va_list first, second; first = second; return 0;],
es_cv_assign_va_list=yes, es_cv_assign_va_list=no))
if test "$es_cv_assign_va_list" = no
then
	AC_DEFINE(NO_VA_LIST_ASSIGN)
fi

dnl check for a u_quad_t or something like that
AC_CACHE_CHECK(for rlimit type ..., es_cv_rlimit_t,
AC_CMDSTDOUT_CPP(es_cv_rlimit_t, 
grep rlim_cur | sed -e 's/rlim_cur.*//' -e 's/^ //g' -e 's/^	//g' -e q,
[#ifdef HAVE_SETRLIMIT
# include <sys/resource.h>
#endif],long))

AC_CACHE_CHECK(for files to extract signal information from,
SIGFILES_cv_,
AC_CMDSTDOUT_CPP(SIGFILES_cv_,
[changequote(,)
egrep '^#[ 	]+1[	 ]+' | sed 's/.*"\(.*\)".*/\1/' |sort -u |  
grep '^/' |tr '\012' ' ' 
changequote([,])],
[#include <signal.h>], /usr/include/signal.h))

AC_SUBST(SIGFILES_cv_)

AC_DEFINE_UNQUOTED(LIMIT_T, $es_cv_rlimit_t)

AC_OUTPUT(Makefile)
