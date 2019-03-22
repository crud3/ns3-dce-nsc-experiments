# NSC on Ubuntu 18.04
## Build environment / Docker image
The Dockerfile is based upon https://github.com/direct-code-execution/dce-dockerfiles/blob/master/ubuntu1604/Dockerfile  
and  
https://github.com/direct-code-execution/ns-3-dce/blob/master/utils/Dockerfile  
Additional packages have been added to facilitate building NSC (according to https://www.nsnam.org/wiki/Installation#Ubuntu.2FDebian.2FMint).

### Compiler versions
Because building NSC requires gcc-5 and g++-5 are installed and made default. I have not yet found a reliable way to force scons to use gcc-5 when another gcc version is installed and set as default. Neither `CC=gcc-5 python2 scons.py` nor setting variables in the `custom.py` file which is read in `SConstruct` work. 

## Errors
First build with scons (`python2 scons.py`) and gcc-7 yields the following error:

```
scons: Reading SConscript files ...
Checking target architecure...(cached) amd64, checking userland ...amd64
Checking for C library fl... yes
scons: done reading SConscript files.
scons: Building targets ...
flex -t globaliser/lexer.l > globaliser/lexer.lex.cc
bison --defines=globaliser//parser.tab.hh globaliser/parser.yc -o globaliser/parser.tab.cc
globaliser/parser.yc: warning: 1 shift/reduce conflict [-Wconflicts-sr]
g++ -o globaliser/lexer.lex.o -c -Wall -g -O globaliser/lexer.lex.cc
g++ -o globaliser/parser.tab.o -c -Wall -g -O globaliser/parser.tab.cc
globaliser/parser.yc: In function 'int yyparse()':
globaliser/parser.yc:159:17: error: reference to 'is_function' is ambiguous
             if (is_function($1->s_identifier)) {
                 ^~~~~~~~~~~
In file included from globaliser/parser.yc:51:0:
globaliser/handle_global.h:15:6: note: candidates are: bool is_function(const string&)
 bool is_function(const string &iden);
      ^~~~~~~~~~~
In file included from /usr/include/c++/7/bits/move.h:54:0,
                 from /usr/include/c++/7/bits/stl_pair.h:59,
                 from /usr/include/c++/7/bits/stl_algobase.h:64,
                 from /usr/include/c++/7/bits/char_traits.h:39,
                 from /usr/include/c++/7/string:40,
                 from globaliser/parser.yc:35:
/usr/include/c++/7/type_traits:403:12: note:                 template<class> struct std::is_function
     struct is_function;
            ^~~~~~~~~~~
globaliser/parser.yc:168:17: error: reference to 'is_function' is ambiguous
             if (is_function($1->s_identifier)) {
                 ^~~~~~~~~~~
In file included from globaliser/parser.yc:51:0:
globaliser/handle_global.h:15:6: note: candidates are: bool is_function(const string&)
 bool is_function(const string &iden);
      ^~~~~~~~~~~
In file included from /usr/include/c++/7/bits/move.h:54:0,
                 from /usr/include/c++/7/bits/stl_pair.h:59,
                 from /usr/include/c++/7/bits/stl_algobase.h:64,
                 from /usr/include/c++/7/bits/char_traits.h:39,
                 from /usr/include/c++/7/string:40,
                 from globaliser/parser.yc:35:
/usr/include/c++/7/type_traits:403:12: note:                 template<class> struct std::is_function
     struct is_function;
            ^~~~~~~~~~~
scons: *** [globaliser/parser.tab.o] Error 1
scons: building terminated because of errors.
```

Consultation with @cdoepmann reveals that the following changes to the sources can be made to mitigate this error:
```
diff -r a/globaliser/SConscript b/globaliser/SConscript
22c22
<     CCFLAGS = '-Wall -g -O'
---
>     CCFLAGS = '-Wall -std=c++98 -g -O'
diff -r a/globaliser/sim/num_stacks.h b/globaliser/sim/num_stacks.h
25c25
< #define NUM_STACKS 50
---
> #define NUM_STACKS 2000
diff -r a/sim/num_stacks.h b/sim/num_stacks.h
25c25
< #define NUM_STACKS 50
---
> #define NUM_STACKS 2000

```
This error message can also be mitigated by enforcing usage of gcc version 5, because gcc-5 is recommended (probably necessary) for NSC anyways.

After above changes (either using gcc-5 or changes to sources), building still fails with the following error:
```
scons: Reading SConscript files ...
Checking target architecure...(cached) amd64, checking userland ...(cached) amd64
Checking for C library fl... (cached) yes
scons: done reading SConscript files.
scons: Building targets ...
g++ -o globaliser/lexer.lex.o -c -Wall -std=c++98 -g -O globaliser/lexer.lex.cc
g++ -o globaliser/parser.tab.o -c -Wall -std=c++98 -g -O globaliser/parser.tab.cc
g++ -o globaliser/ilex.o -c -Wall -std=c++98 -g -O globaliser/ilex.cc
g++ -o globaliser/handle_global.o -c -Wall -std=c++98 -g -O globaliser/handle_global.cc
g++ -o globaliser/node.o -c -Wall -std=c++98 -g -O globaliser/node.cc
globaliser/node.cc: In member function 'bool node_t::handle_global(std::__cxx11::string, std::__cxx11::list<std::__cxx11::basic_string<char> >*, std::__cxx11::list<std::__cxx11::basic_string<char> >*)':
globaliser/node.cc:906:36: warning: variable 'str_start' set but not used [-Wunused-but-set-variable]
               string::size_type i, str_start, str_end;
                                    ^~~~~~~~~
globaliser/node.cc:772:40: warning: variable 'declarator' set but not used [-Wunused-but-set-variable]
             *initialiser_node = NULL, *declarator = NULL;
                                        ^~~~~~~~~~
globaliser/node.cc:483:9: warning: variable 'orig_newlines' set but not used [-Wunused-but-set-variable]
     int orig_newlines;
         ^~~~~~~~~~~~~
g++ -o globaliser/globaliser globaliser/lexer.lex.o globaliser/parser.tab.o globaliser/ilex.o globaliser/handle_global.o globaliser/node.o -lfl
/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/libfl.so: undefined reference to `yylex'
collect2: error: ld returned 1 exit status
scons: *** [globaliser/globaliser] Error 1
scons: building terminated because of errors. 
```

Whether one uses bake or scons directly does not matter for those errors.

### Flex error
The beforementioned ```libfl.so: undefined reference to `yylex'``` error may be caused by an issue in the flex library version installed for Ubuntu 18.04 (http://lists.linuxfromscratch.org/pipermail/blfs-support/2015-April/076525.html). The proposed fix (replacing `-lfl` with `/usr/lib/libfl.a`) can't be easily applied for NSC. A search for `-lfl` in the sources yields the following results:
```
grep -r "\-lfl" .
Binary file ./source/nsc-0.5.3/.sconsign.dblite matches
./source/nsc-0.5.3/config.log:  |/usr/bin/gcc-5 -o .sconf_temp/conftest_1 .sconf_temp/conftest_1.o -lfl

```
The `config.log` file has the following content:
```
file /home/crude/nsctest/source/nsc-0.5.3/SConstruct,line 154:
    Configure(confdir = .sconf_temp)
scons: Configure: Checking target architecure...
scons: Configure: (cached) amd64, checking userland ...
scons: Configure: ".sconf_temp/conftest_0.c" is up to date.
scons: Configure: The original builder output was:
  |.sconf_temp/conftest_0.c <-
  |  |int main(void)
  |  |{return 0;}
  |  |
  |
scons: Configure: ".sconf_temp/conftest_0.o" is up to date.
scons: Configure: The original builder output was:
  |/usr/bin/gcc-5 -o .sconf_temp/conftest_0.o -c .sconf_temp/conftest_0.c
  |
scons: Configure: (cached) amd64

scons: Configure: Checking for C library fl... 
scons: Configure: ".sconf_temp/conftest_1.c" is up to date.
scons: Configure: The original builder output was:
  |.sconf_temp/conftest_1.c <-
  |  |
  |  |
  |  |
  |  |int
  |  |main() {
  |  |  
  |  |return 0;
  |  |}
  |  |
  |
scons: Configure: ".sconf_temp/conftest_1.o" is up to date.
scons: Configure: The original builder output was:
  |/usr/bin/gcc-5 -o .sconf_temp/conftest_1.o -c .sconf_temp/conftest_1.c
  |
scons: Configure: ".sconf_temp/conftest_1" is up to date.
scons: Configure: The original builder output was:
  |/usr/bin/gcc-5 -o .sconf_temp/conftest_1 .sconf_temp/conftest_1.o -lfl
  |
scons: Configure: (cached) yes
```


Other search results concerning this error message:  
https://stackoverflow.com/questions/34782625/undefined-reference-to-yylex  
https://github.com/sipcapture/captagent/issues/45

## Result
Failure to build NSC on Ubuntu 18.04.

Further investigation and/or consult mailing list.
