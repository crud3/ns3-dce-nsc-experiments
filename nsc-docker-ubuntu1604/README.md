# NSC on Ubuntu 16.04
## Build environment / Docker image
The Dockerfile is based upon https://github.com/direct-code-execution/dce-dockerfiles/blob/master/ubuntu1604/Dockerfile  
and  
https://github.com/direct-code-execution/ns-3-dce/blob/master/utils/Dockerfile  
Additional packages have been added to facilitate building NSC (according to https://www.nsnam.org/wiki/Installation#Ubuntu.2FDebian.2FMint).  
NSC is built using the SCONS build system, either directly or through bake. Ubuntu 16.04 already uses gcc-5, so no modifications to the gcc versions were necessary.

## Errors (using scons directly)
Building directly with with scons (`python2 scons.py`) yields the following error:

```
...

Install file: "linux-2.6.26/liblinux2.6.26.so" as "lib/liblinux2.6.26.so"
g++ -o test/app.o -c -Wall -g -Isim -Itest test/app.cc
g++ -o test/sim.o -c -Wall -g -Isim -Itest test/sim.cc
g++ -o test/net.o -c -Wall -g -Isim -Itest test/net.cc
g++ -o test/nsc_support.o -c -Wall -g -Isim -Itest test/nsc_support.cc
g++ -o test/simple_test.o -c -Wall -g -Isim -Itest test/simple_test.cc
ar rc test/libsimulator.a test/sim.o test/net.o test/nsc_support.o test/app.o test/simple_test.o
ranlib test/libsimulator.a
g++ -o test/test_interface.o -c -Wall -g -Isim -Itest test/test_interface.cc
g++ -o test/test_interface test/test_interface.o -Ltest -lfl -lsimulator -ldl
LD_LIBRARY_PATH=/home/ns3nsc/ns3-nsc/nsc-dev/lib test/test_interface
TEST: TestNewLinuxInterface ... /home/ns3nsc/ns3-nsc/nsc-dev/lib/liblinux2.6.18.so: undefined symbol: __memory_barrier
FAIL
test/test_interface.cc:15: assertion 'ops->Load(stack_name) == 0' failed.
scons: *** [test/test_interface] Error 1
scons: building terminated because of errors.
```


## Result
Failure to build NSC on Ubuntu 16.04 with `python2 scons.py`.

Success when building NSC on Ubuntu 16.04 with bake. 
`./test.py -s ns3-tcp-interoperability` succeeds.

### nstor with NSC on Ubuntu 16.04
Building nstor fails because of https://github.com/crud3/nstor/issues/1  
This is temporarily fixed by just deleting the `scratch/01-single-star-web/` folder, after which building nstor succeeds. The `tor-example` script can be run with NSC without errors on all tags (nstor-ns-3.27, nstor-ns-3.28, nstor-ns-3.29) so I assume NSC itself and NSC in combination with nstor works correctly. The build failure may be due to compiler issues in the docker container. If it's really a problem with the `sim.cc` in nstor, the build process can be changed back to normal after a bugfix.
