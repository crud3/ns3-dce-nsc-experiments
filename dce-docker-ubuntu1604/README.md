# DCE on Ubuntu 16.04

## Build environment / Docker image
The Dockerfile is stitched together from https://github.com/direct-code-execution/dce-dockerfiles/blob/master/ubuntu1604/Dockerfile and https://github.com/direct-code-execution/ns-3-dce/blob/master/utils/Dockerfile with some additional packages. DCE is built using the original bake instead of https://github.com/thehajime/bake and with added debug options to produce valuable logs.

## Errors
No critical errors.

## Result
DCE build completes without critical errors, and passes all tests of the `source/ns-3-dce/test.py` script. Some of the missing dependencies could be installed, but they are not necessary to build or pass tests and thus have been omitted to keep the image smaller.

## TODO