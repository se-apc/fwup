ACLOCAL_AMFLAGS=-I m4

bin_PROGRAMS=fwup
fwup_SOURCES=\
	archive_open.c \
	block_cache.c \
	cfgfile.c \
	cfgprint.c \
	crc32.c \
	eval_math.c \
	fatfs.c \
	functions.c \
	fwfile.c \
	fwup_apply.c \
	fwup.c \
	fwup_create.c \
	fwup_list.c \
	fwup_sign.c \
	fwup_verify.c \
	fwup_metadata.c \
	fwup_genkeys.c \
	mbr.c \
	mmc_bsd.c \
	mmc_linux.c \
	mmc_osx.c \
	mmc_windows.c \
	pad_to_block_writer.c \
	progress.c \
	requirement.c \
	resources.c \
	simple_string.c \
	sparse_file.c \
	uboot_env.c \
	util.c \
	archive_open.h \
	block_cache.h \
	cfgfile.h \
	cfgprint.h \
	crc32.h \
	eval_math.h \
	fatfs.h \
	functions.h \
	fwfile.h \
	fwup_apply.h \
	fwup_create.h \
	fwup_list.h \
	fwup_metadata.h \
	fwup_genkeys.h \
	fwup_sign.h \
	fwup_verify.h \
	mbr.h \
	mmc.h \
	pad_to_block_writer.h \
	progress.h \
	requirement.h \
	resources.h \
	simple_string.h \
	sparse_file.h \
	uboot_env.h \
	util.h \
	../3rdparty/base64.c \
	../3rdparty/base64.h \
	../3rdparty/semver.c/semver.c \
	../3rdparty/semver.c/semver.h \
	../3rdparty/fatfs/source/diskio.h \
	../3rdparty/fatfs/source/ff.c \
	../3rdparty/fatfs/source/ff.h \
	../3rdparty/fatfs/source/ffconf.h \
	../3rdparty/fatfs/source/ffunicode.c

EXTRA_DIST= \
	../3rdparty/fatfs/source/00history.txt \
	../3rdparty/fatfs/source/00readme.txt \
	../3rdparty/fatfs/source/diskio.c \
	../3rdparty/fatfs/source/ffsystem.c \
	../3rdparty/fatfs/LICENSE.txt \
	../3rdparty/semver.c/README.md \
	../3rdparty/semver.c/LICENSE \
	../3rdparty/semver.c/Makefile \
	../3rdparty/semver.c/semver_unit.c \
	../3rdparty/semver.c/semver_test.c \
	../3rdparty/strptime.c \
	fwup.h2m

if !HAS_STRPTIME
fwup_SOURCES+=../3rdparty/strptime.c
endif

fwup_CFLAGS = -Wall -D_FILE_OFFSET_BITS=64 \
	      $(CONFUSE_CFLAGS) \
	      $(ARCHIVE_CFLAGS) \
	      $(SODIUM_CFLAGS) \
	      $(PTHREAD_CFLAGS)

fwup_LDADD = $(CONFUSE_LIBS) \
	     $(ARCHIVE_LIBS) \
	     $(SODIUM_LIBS) \
	     $(PTHREAD_LIBS)

if ENABLE_GCOV
fwup_CFLAGS += -fprofile-arcs -ftest-coverage
endif

if HAVE_HELP2MAN
man_MANS = fwup.1
CLEANFILES = $(man_MANS)

fwup.1: fwup$(EXEEXT) fwup.h2m
	$(HELP2MAN) --no-discard-stderr --output=$@ --no-info --include fwup.h2m \
	    -n "configurable embedded firmware update creator and runner" \
	    ./fwup$(EXEEXT)
else
	@echo  "Warning: help2man not available, no man page created."
endif

if ENABLE_BASH_COMPLETION
bashcompletiondir = $(BASH_COMPLETION_DIR)
dist_bashcompletion_DATA = bash-completion/fwup
endif

