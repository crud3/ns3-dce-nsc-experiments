# DCE on Ubuntu 18.04
## Build environment / Docker image
The Dockerfile is based upon https://github.com/direct-code-execution/dce-dockerfiles/blob/master/ubuntu1604/Dockerfile  
and  
https://github.com/direct-code-execution/ns-3-dce/blob/master/utils/Dockerfile  
The `bc wget` packages have been added to mitigate errors. DCE is built using the original bake instead of https://github.com/thehajime/bake and with added debug options to produce valuable logs. Some of the missing dependencies could be installed, but they are not necessary to build. The optional dependencies were also not necessary to pass tests for the Ubuntu 16.04 image and have thus been omitted to keep the image smaller.

## Errors
The test script `source/ns-3-dce/test.py` fails multiple times because of core dumps.  The `Fatal error: glibc detected an invalid stdio handle` error message leads to https://github.com/direct-code-execution/ns-3-dce/issues/57, where multiple other users report this error on Ubuntu 18.04. Consultation of the ns-3 mailing list confirmed it is a known issue and unknown whether and when it is going to be fixed (https://groups.google.com/forum/#!topic/ns-3-users/nhYx8ghdERE ). Recommendation was to use older Linux releases.

## Result
DCE build completes without critical errors, but it does only pass 3 out of 47 tests of the `source/ns-3-dce/test.py` script and is therefore non-functional. It is unlikely that this can be solved in the near future.
