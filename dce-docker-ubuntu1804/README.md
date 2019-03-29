# DCE on Ubuntu 18.04
## Build environment / Docker image
The Dockerfile is based upon https://github.com/direct-code-execution/dce-dockerfiles/blob/master/ubuntu1604/Dockerfile  
and https://github.com/direct-code-execution/ns-3-dce/blob/master/utils/Dockerfile  
DCE is built using the original bake instead of https://github.com/thehajime/bake and with added debug options to produce valuable logs. Some of the missing dependencies could be installed, but they are not necessary to build. The optional dependencies were also not necessary to pass DCE tests for the Ubuntu 16.04 image and have thus been omitted to keep the image smaller.

## Errors
The test script `source/ns-3-dce/test.py` fails multiple times because of core dumps.  The `Fatal error: glibc detected an invalid stdio handle` error message leads to https://github.com/direct-code-execution/ns-3-dce/issues/57, where multiple other users report this error on Ubuntu 18.04. Consultation of the ns-3 mailing list confirmed it is a known issue and unknown whether and when it is going to be fixed (https://groups.google.com/forum/#!topic/ns-3-users/nhYx8ghdERE ). Recommendation was to use older Linux releases.

### Debugging the 'glibc detected an invalid stdio handle' error
First idea was to use another glibc version for DCE. Debugging was not conducted in the docker container, but on my local Ubuntu 18.04 installation. First choice was glibc 2.23 (standard on Ubuntu 16.04).
#### Building glibc
May require to additionally install `gawk` package.
```
git clone git://sourceware.org/git/glibc.git && cd glibc && git checkout glibc-2.23
# Apply patch becaue commit ab30899d for glibc-2.23 without patch gives fails building on Ubuntu 18.04
git apply < glibc2-23_regexp.patch # See below
mkdir ../glibc-build ../glibc-2.23 && cd ../glibc-build
./../glibc/configure --prefix=/home/crude/dev/ns3/glibc-2.23 --enable-kernel=2.6.26 --disable-werror
make -j 4
```
Building without patch on Ubuntu 18.04 fails, see:  
https://patchwork.ozlabs.org/patch/797471/  
https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=869717  
Patchfile (generated with git diff, code from https://sourceware.org/git/gitweb.cgi?p=glibc.git;a=commitdiff;h=388b4f1a02f3a801965028bbfcd48d905638b797)  
```
diff --git a/misc/regexp.c b/misc/regexp.c
index 3b3668272f..b2a2c6e636 100644
--- a/misc/regexp.c
+++ b/misc/regexp.c
@@ -29,14 +29,15 @@
 
 #if SHLIB_COMPAT (libc, GLIBC_2_0, GLIBC_2_23)
 
-/* Define the variables used for the interface.  */
-char *loc1;
-char *loc2;
+/* Define the variables used for the interface.  Avoid .symver on common
+   symbol, which just creates a new common symbol, not an alias.  */
+char *loc1 __attribute__ ((nocommon));
+char *loc2 __attribute__ ((nocommon));
 compat_symbol (libc, loc1, loc1, GLIBC_2_0);
 compat_symbol (libc, loc2, loc2, GLIBC_2_0);
 
 /* Although we do not support the use we define this variable as well.  */
-char *locs;
+char *locs __attribute__ ((nocommon));
 compat_symbol (libc, locs, locs, GLIBC_2_0);
 
 
```

For simple tests with new glibc and to check if the correct glibc is used, the following program (`libcversion.c`) may be useful:
```
 #include <stdio.h>
 #include <gnu/libc-version.h>
 int main (void) { puts (gnu_get_libc_version ()); return 0; }
```
Then compile with new glibc and specify loader:
```
gcc libcversion.c -o libcversion -Wl,--rpath=/home/crude/dev/ns3/glibc-2.23/lib -Wl,--dynamic-linker=/home/crude/dev/ns3/glibc-2.23/lib/ld-2.23.so
./libcversion #should now print '2.23'
```

#### Building DCE with different glibc
First naive approach was to just build DCE with a different system glibc and see what happens. For this, bake needs to be told to use the correct flags when compiling DCE.
One approach for this could be to create a dedicated config in `bake/bakeconf.xml`
```
    <module name="my-dce-linux-dev">
      <source type="git">
      <attribute name="url" value="git://github.com/direct-code-execution/ns-3-dce"/>
        <attribute name="module_directory" value="ns-3-dce"/>
      </source>
      <depends_on name="dce-meta-dev" optional="False"/>
      <depends_on name="net-next-nuse-4.4.0" optional="False"/>
      <depends_on name="iproute2-4.4.0" optional="False"/>
      <depends_on name="lksctp-dev" optional="True"/>
      <!-- Trying to disable optional modules to reduce build time for debugging -->
      <disable name="qt"/>
      <disable name="pyviz-gtk3-prerequisites"/>
      <disable name="netanim"/>
      <disable name="libgoocanvas2"/>
      <build type="waf" objdir="yes" sourcedir="ns-3-dce">
        <attribute name="supported_os" value="linux;linux2"/>
        <attribute name="configure_arguments" value="configure --prefix=$INSTALLDIR --with-ns3=$INSTALLDIR --with-elf-loader=$INSTALLDIR/lib --with-libaspect=$INSTALLDIR --enable-kernel-stack=$SRCDIR/../net-next-nuse-4.4.0/arch"/>
        <!-- try new_variable from tutorial 
        <attribute name="new_variable" value="YO" />-->
      </build>
    </module>
```
Potentially interesting links regarding this:  
https://www.nsnam.org/docs/bake/tutorial/html/bake-tutorial.html#the-configuration-file  
https://www.nsnam.org/docs/release/3.25/tutorial/singlehtml/index.html#compilers-and-flags  
https://waf.io/apidocs/confmap.html  
Information about linking can also be found at the [NSC on Ubuntu 18.04 experiment](../nsc-docker-ubuntu1804/README.md)

After checking the configuration files, it became clear that DCE uses its own libc. Excerpt from `wscript`:
```
# The very small libc used to replace the glibc
    # and forward to the dce_* code
    bld.shlib(source = ['model/libc.cc', 'model/libc-setup.cc', 'model/libc-global-variables.cc'],
              target='lib/c-ns3',
              cxxflags=['-g', '-fno-profile-arcs', '-fno-test-coverage', '-Wno-builtin-declaration-mismatch'],
              defines=['LIBSETUP=libc_setup'],
              linkflags=['-nostdlib', '-fno-profile-arcs',
                         '-Wl,--version-script=' + os.path.join('model', 'libc.version'),
'-Wl,-soname=libc.so.6'])
```

I also found the "Direct Code Execution: Revisiting Library OS Architecture for Reproducible Network Experiments" (2014) paper: https://hal.inria.fr/hal-00880870v2/document  
This paper contains information about the interaction between their libc implementation and the system glibc (see section 2.3) and other architecture design decisions.

This error can probably not be solved by simple means from the outside, but instead requires a depper understanding of the DCE library. As per answer at the mailing list, it is unclear if and when a mainter addresses this.

## Result
DCE build completes without critical errors, but it does only pass 3 out of 47 tests of the `source/ns-3-dce/test.py` script and is therefore non-functional. 
