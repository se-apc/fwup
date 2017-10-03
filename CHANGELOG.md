# Changelog

## v0.16.0

  * New features
    * Added path_write, pipe_write, and execute commands. These are only usable
      if the --unsafe flag is passed. They enable people using raw NAND to
      invoke the ubi tools. People wanting fwup to upgrade other chips can also
      use these commands. These commands are considered experimental, can create
      platform-dependent .fw files, and open up some security issues. However,
      assuming signed firmware updates, they can be very useful. Thanks to
      Michael Schmidt for these additions.
    * Progress bar now includes approximate bytes written.

  * Bug fixes
    * Support overwriting files in FAT partitions. Previously you had to remove
      them first.
    * Fix tests to run on BSD systems again.

## v0.15.4

  * New features
    * Changed signing keys to be base64 encoded so that they'd be easier to bake
      into firmware and pass in environment variables on CI systems. The
      previous raw binary format still works and will remain supported.
    * Added commandline parameters for passing public and private keys via
      commandline arguments. Along with the base64 change, this cleans up CI
      build scripts.

  * Bug fixes/Improvements
    * Fix lseek seek_end issue on Mac when working with SDCards. This fixed an
      issue where upgrade tasks didn't work on Macs. Not a common issue, but
      confusing since you'd hit it while debugging.
    * Make requirement checks report their result with `-v`.
    * Fix verbose prints to use the fwup_warn helper instead of calling fprintf
      directely. (Cleanup)
    * Enlarged trim cache for up to 64 GiB memory devices. Large ones will work,
      but trim caching is ignored after 64 GiB. This should support almost all
      known uses of fwup now. The use of fwup on large SSDs still works, since
      fwup is pretty much only used at lower offsets.

## v0.15.3

  * Bug fixes/Improvements
    * Fix segfault when using large media. This was found on a 1 TB SSD, but
      should have affected much smaller media.
    * Improved error messages for when FAT filesystems get corrupt and start
      returning weird errors.
    * Fixed trimming on media that wasn't a multiple of 128K bytes. This could
      have resulted in loss of data if anything was stored in the final bytes.
    * Fixed memory leaks identified by valgrind (nothing affecting proper
      operation)

## v0.15.2

  * New features
    * Added meta-misc and meta-vcs-identifier metadata fields. This addition is
      backwards compatible assuming you're using libconfuse 3.0 or later. If you
      don't use these metadata fields in your fwup.conf files, there is no
      compatibility issue.

  * Bug fixes/Improvements
    * Order block cache flush logic so that blocks get written in fwup.conf
      order. This fixes an issue where the cache could write the A/B partition
      swap before the new firmware was completely written. Given that the
      cache is pretty small and there was an extra flush before on-final,
      systems without the fix are likely fine.
    * Improve the caching heuristics to reduce the number of writes to FAT
      filesystems.
    * Improve detection of typos in variable names and content. This catches
      accidental writes to offset 0 when creating fwup.config files among other
      annoyances.
    * Improve MBR error messages

## v0.15.1

  * Bug fixes
    * The OSX eject bug was finally found. It was due to the file handle still
      being open on the SDCard at the time of the eject. For whatever reason, it
      turned out to be 100% reproducable on v0.15.0 with a simple .fw file.

## v0.15.0

  * Completely rewritten caching layer. This has the following improvements;
    * OS caching no longer used on Linux. Direct I/O is used now with the
      caching internal to fwup. For large archives, this results in about
      a 10% performance improvement. More importantly, fwup can provide more
      confidence that the final write to swap A/B partitions actually happens
      last.
    * Flash erase block aligned writes - currently hardcoded to 128 KB. This may
      be helpful for some devices even though it doesn't appear to improve
      performance.
    * One cache - previously there were limited ones for FAT operations and raw
      writes. This removed some code and the new implementation is simpler.
    * Support for discarding unused blocks and hardware TRIM
    * Unit tests now validate read and write syscalls for alignment, size and
      final order on Linux using ptrace.
    * The progress indicator now moves more linearly rather than racing to 99%
      and hanging for a while.

  * New features
    * trim() command to discard storage regions. This avoids read/modify/write
      operations from being issued by fwup on smaller than erase-block sized
      writes.
    * `--enable-trim` option to enable the sending of hardware TRIM commands to
      devices that support them.
    * Added uboot_recover command for optionally re-initialize corrupt or
      uninitialized uboot environments. This is useful for adding u-boot
      environment blocks to devices without them as is being done with Nerves.

  * Bug fixes
    * Fix segfault if a bad value is specified in the u-boot environment block.
    * Silence eject failure warning on OSX that appears to be harmless. Fixes
      #29.

## v0.14.3

  * Bug fixes
    * Fix regression tests to run on ZFS. Thanks to georgewhewell for finding
      the issue and helping debug it.

## v0.14.2

  * Bug fixes
    * Fixed fat_mkdir to not error if the directory already exists. This
      preserves pre-v0.14.0 expectations in some fwup.conf scripts. Other
      fat_mkdir errors will be reported and not ignored like in pre-v0.14.0
      days.
    * Changed warning code in framing to match specs. ("WA"->"WN"). Warnings
      are so rarely used that this was unnoticed.
    * Clarified framing docs

## v0.14.1

  * Bug fixes
    * Patch libsodium 1.0.12 so that static mingw builds work.

## v0.14.0

  * New features
    * Add support for creating resources inside configuration files. This makes
      it possible to create simple config files that have fwup variable
      references in them without requiring a separate script.
    * Add -1,-2,-3,...-9 to tune the compression. The default is -9, but on
      massive archives, this is really slow, so you can pass a lower number
      to speed up archive creation.
    * on-error has finally been implemented. This allows fwup.conf files to
      specify cleanup code should something go wrong. Cleanup is performed
      best effort and fwup still returns an error on exit.
    * Add support for matching strings in files on a FAT filesystem for whether
      to apply upgrades. See `require-fat-file-match()`. Useful for bootloaders
      that can modify a file, but not create/remove one (e.g., grub's grubenv
      file)
    * Verify that all on-resource handlers are run. This is the normal
      expectation, and this change guarantees it.

  * Bug fixes
    * When streaming, the input would always be read through to the end. This
      meant that errors could take a long time to be reported and that was
      annoying at best.
    * FAT filesystem corruption wouldn't cause many of the fat_* commands to
      fail. They do now and unit tests have been added to verify this going
      forward.
    * Fix segfault when reading against a corrupt FAT filesystem due to a
      signed/unsigned conversion. fwup errors out properly now.

## v0.13.0

  * New features
    * Add `require-path-on-device` as another way of detecting which partition
      is active when running A/B updates. This is only works when updating on
      device (the usual way of updating).
    * Add `info()` function for printing out information messages from updates.
      This is helpful when debugging scripts.

  * Bug fixes
    * Flush messages in framed mode to prevent them from getting queued.
    * Send "OK" success message when done applying in framed mode.
    * Support writing >2GiB files on 32-bit platforms

## v0.12.1

  * Bug fixes
    * Fix really subtle issue with archive_read_data_block returning 0 bytes
      when not at EOF. This issue should have been present since the beginning,
      but it only appeared recently.

## v0.12.0

  * New features
    * Added easy way of invoking fwup for those just wanting to program an
      SDCard. Just run, "fwup myfile.fw" and fwup will automatically look
      for an attached SDCard, ask for confirmation, and apply the complete
      task.
    * Ported to FreeBSD and other BSDs. This allows .fw files to be created
      and applied on these systems. (I'm interested in hearing from embedded
      BSD users for refining this support.)
    * Added commandline options to scale the progress reports. See
      --progress-low and --progress-high
    * Added an error() function to let config files produce friendlier error
      messages when none of the upgrade (or any other) tasks match.

  * Bug fixes
    * Sparse files that end with holes will be the right length when written.

## v0.11.0

  * New features
    * Added sparse file support. This can significantly reduce the amount of
      time required to write a large EXT2 partition to Flash. It queries the
      OS for gaps so that it will write to the same places that mke2fs wrote
      and skip the places it didn't. See README.md for more info.
    * Updated FatFs from R0.11a to R0.12b. See
      http://elm-chan.org/fsw/ff/updates.txt.

  * Bug fixes
    * Several issues were found and fixed in the write caching code by
      constructing a special .fw file manually. To my knowledge, none of the
      issues could be triggered w/o sparse file support and a filesystem
      that supported <512 byte sparse blocks.

  * Backwards incompatible changes
    * Sparse file support - if used, the created archives will not work on old
      versions of fwup.
    * Remove support for fw_create and fw_add_local_file. Neither of these were
      used much and there were better ways of accomplishing their functions.
      They became a pain to support, since they create .fw files that are
      missing sections, so they don't validate. It is much easier to run fwup
      to create multiple .fw files that are included in the master one for doing
      things like apply/commit or apply/revert style updates that stay within
      fwup. If you're using the functions, either don't upgrade or create new
      .fw configurings that embed pre-generated .fw files.

## v0.10.0

  * New features
    * Add U-Boot environment support. This allows firmware updates to modify
      the U-Boot environment to indicate things like which partition is active.
      fwup can also look at the U-Boot environment to decide which partition to
      update.
    * Add raw_memset to write a fixed value over a range of blocks. This is good
      for invalidating SDCard regions in manufacturing so that they're
      guaranteed to be reinitialized on first boot. SDCard TRIM support would be
      better, but fwup doesn't work on the bulk programmers.
    * Add define_eval() to support running simple math expressions when building
      firmware update packages. This makes entering offset/size pairs less
      tedious.

  * Bug fixes
    * Re-enable max SDCard size check on Linux to reduce risk of writing to an
      hard drive partition by accident.

## v0.9.2

  * Bug fixes
    * Fix SDCard corruption issue on Windows. Disk volumes are now locked for
      the duration of the write process. Thanks to @michaelkschmidt for the fix.
    * Allow /dev/sda to be auto-detected as an SDCard on Linux. It turned out
      that for some systems, this was a legit location. Most people will not
      see it, since it doesn't pass other tests.
    * Set compression parameters on libarchive's zip compressor. This wasn't a
      actually a bug, but there seemed to be some variability in how .fw files
      were compressed.

## v0.9.1

  * New features
    * Build Chocolatey package for Windows - lets Windows user run `choco
      install fwup` once the package is accepted into the Chocolatey repo.
    * Build a `.deb` package for Raspbian. This makes it easier to install
      `fwup` on Raspberry Pis. CI now builds and tests 32-bit armhf versions.

## v0.9.0

  * New features
    * Windows port - It's now possible to natively write to SDCards on
      Windows.

  * Bug fixes
    * Don't create files in `/dev`. This fixes a TOCTOU bug where an SDCard
      exists during enumeration time and disappears before the write. When this
      happened, a regular file was created in `/dev` which just confused
      everyone.
    * Support writing 0-byte files to FAT partitions. They were being skipped
      before. This is different than `touching` a file since it can be used to
      truncate an existing file.

## v0.8.2

This release has only one fix in it to address a corruption issue when updating
an existing FAT filesystem. The corruption manifests itself as zero'ing out 512 bytes
at some place in the filesystem that wasn't being upgraded. It appears that the
location stays the same with the configuration that reproduced the issue. Due to some luck,
the condition that causes this is relatively rare and appears to only have
caused crashes for BeagleBone Green users. BeagleBone Black users still had the
corruption, but it did not affect operation.

It is highly recommended to upgrade the version of `fwup` on your target to this
version.

  * Bug fixes
    * Fix FAT filesystem corruption when running a software update.

## v0.8.1

This release is a bug fix release on v0.8.0. The combination of significantly
improved code coverage on the regression tests (see Coveralls status) and the
Windows port uncovered several bugs. People submitting fwup to distributions are
highly encouraged to run the regression tests (`make check`), since some issues
only appear when running against old or misconfigured versions of libconfuse and
libarchive.

  * Bug fixes
    * Use pkg-config in autoconf scripts to properly discover transitive
      dependencies. Thanks to the Buildroot project for discovering a broken
      configuration.
    * Fix lack of compression support in libarchive in static builds (regression
      test added to catch this in the future)
    * Fix MBR partition handling on 32-bit systems (offsets between 2^31 and
      2^32 would fail)
    * Fix uninitialized variable when framing on stdin/stdout is enabled
    * Various error message improvements thanks to Greg Mefford.

## v0.8.0

  * New features
    * Added assertions for verifying that inputs don't excede their expected
      sizes. The assertions are only checked at .fw creation time.
    * Add support for concatenating files together to create one resource. This
      was always possible before using a preprocessing script, but is more
      convenient now.
    * Add "framed" input and output when using stdin/stdout to simplify
      integration with Erlang and possibly other languages.
    * Detecting attached SDCards no longer requires superuser permissions on
      Linux.
    * Listing detected SDCards (`--detect` option) prints the SDCard size as
      well. ***This is a breaking change if you're using this in a script***

  * Bug fixes
    * Fixed va_args bug that could cause a crash with long fwup.conf inputs
    * fwup compiles against uclibc now
    * autoreconf can be run without creating the m4 directory beforehand

## v0.7.0

This release introduces a change to specifying requirements on upgrade sections.
Previously, the only supported option was `require-partition1-offset`.
Requirements can now be specified using function syntax. If you have older versions of
`fwup` in the field, using this new feature will create .fw files that won't
apply. The change makes requirement support less of a hack. If you don't change
to the new syntax, `fwup` will continue to create `.fw` files that are
compatible with old versions.

  * New features
    * Task requirement code now uses functions for checks
    * Add fat_touch to create 0 length files on FAT filesystems
    * Add require-fat-file-exists to check for the existance of a file when
      determining which task to run
    * libconfuse 3.0's unknown attribute support is now used to make fwup
      more robust against changes to meta.conf contents
    * open_memstream (or its 3rd party implementation) is no longer needed. This
      helps portability.
    * The meta.conf file is stripped of empty sections, lists, and attributes
      set to their defaults. If you have an old fwup version in the field, or
      you're using libconfuse < 3.0, it is now much harder to generate
      incompatible fwup files assuming you don't use features newer than your
      deployed fwup versions.

  * Bug fixes
    * Autodetection will work for SDCards up to 64 GB now
    * Fixed off-by-month bug when creating files in FAT partitions

## v0.6.1

  * New features
    * Add bash completion
    * Prebuilt .deb and .rpm archives for Linux

## v0.6.0

  * Bug fixes
    * Fix FAT filesystem creation for AM335x processors (Beaglebone Black)

  * New features
    * Added manpages

## v0.5.2

  * Bug fixes
    * Clean up progress printout that came before SDCard confirmation
    * Improve error message when the read-only tab of an SDCard is on

## v0.5.1

  * New features
    * sudo is no longer needed on OSX to write to SDCards
    * --unmount and --no-unmount commandline options to unmount (or not) all partitions on a device first
    * --eject and --no-eject commandline options to eject devices when complete (OSX)
    * --detect commandline option to list detected SDCards and removable media

  * Bug fixes
    * Various installation fixes and clarifications in the README.md
    * Fix rpath issue with libsodium not being in a system library directory
    * If unmounting is needed and fails, fwup fails. This is different from before. Use --no-unmount to skip this step.

## v0.5.0

  * New features
    * Add `--version` commandline option
    * Document and add more long options
    * Update fatfs library to R0.11a

  * Bug fixes
    * Improve disk full error message (thanks Tony Arkles)

## v0.4.2

  * New features
    * Write caching for FAT and large binaries (helps on OSX)

## v0.4.1

  * Bug fixes
    * Fix streaming upgrades and add unit tests to catch regressions

## v0.4.0

  * New features
    * Builds and programs SDCards on OSX

## v0.3.1

  * Fixes
    * Fix segfault on some unknown arguments

## v0.3.0

This release uses libsodium for cryptographic routines. This makes it backwards
incompatible with v0.2.0 and earlier.

  * New features
    * Firmware signing using Ed25519
    * Switch hash from SHA to BLAKE2b-256
    * Add support for OSIP headers (required for Intel Edison boards)

## v0.2.0

  * New features
    * fat_mkdir and fat_setlabel support
