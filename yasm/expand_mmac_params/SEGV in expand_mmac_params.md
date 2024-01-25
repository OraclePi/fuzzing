#### Description:
~~~shell
AddressSanitizer: SEGV /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/modules/preprocs/nasm/nasm-pp.c:3862:20 in expand_mmac_params
==27727==ABORTING
~~~

#### Project:
<https://github.com/yasm/yasm>

#### Version:
~~~bash
fk@localhost:/mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master$ ./yasm --version
yasm 1.3.0
Copyright (c) 2001-2014 Peter Johnson and other Yasm developers.
Run yasm --license for licensing overview and summary.
~~~


#### Env:
~~~bash
Ubuntu 22.04.3 LTS
Ubuntu clang version 14.0.0-1ubuntu1.1
gcc version 11.4.0 (Ubuntu 11.4.0-1ubuntu1~22.04)
~~~

#### Reproduce:
~~~shell
./autogen.sh
export CC=afl-clang-fast CXX=afl-clang-fast++ AFL_USE_ASAN=1 ASAN_OPTIONS=detect_leaks=0
./configure
make
./yasm poc.in
~~~


#### ASAN:
~~~bash
fk@localhost:/mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master$ ./yasm id\:000000\,sig\:11\,src\:004450\,time\:17949411\,execs\:1756333\,op\:havoc\,rep\:14
yasm: file name already has no extension: output will be in `yasm.out'
AddressSanitizer:DEADLYSIGNAL
=================================================================
==27727==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000001 (pc 0x55b6497d1f50 bp 0x7ffca37d6e70 sp 0x7ffca37d6d60 T0)
==27727==The signal is caused by a READ memory access.
==27727==Hint: address points to the zero page.
    #0 0x55b6497d1f50 in expand_mmac_params /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/modules/preprocs/nasm/nasm-pp.c:3862:20
    #1 0x55b6497c9417 in pp_getline /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/modules/preprocs/nasm/nasm-pp.c:5079:21
    #2 0x55b6497c2880 in nasm_preproc_get_line /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/modules/preprocs/nasm/nasm-preproc.c:203:12
    #3 0x55b6497b83a6 in nasm_parser_parse /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/modules/parsers/nasm/nasm-parse.c:219:13
    #4 0x55b6497b6032 in nasm_do_parse /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/modules/parsers/nasm/nasm-parser.c:66:5
    #5 0x55b6497b6032 in nasm_parser_do_parse /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/modules/parsers/nasm/nasm-parser.c:83:5
    #6 0x55b649722b09 in do_assemble /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/frontends/yasm/yasm.c:641:5
    #7 0x7f3afadc3d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #8 0x7f3afadc3e3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #9 0x55b64965f4b4 in _start (/mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/yasm+0xba4b4) (BuildId: 4ca98d63b28f16b0ca05fb7a63c14ceb11a123a9)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/modules/preprocs/nasm/nasm-pp.c:3862:20 in expand_mmac_params
==27727==ABORTING
~~~

#### POC:
<https://github.com/OraclePi/fuzzing/blob/main/yasm/expand_mmac_params/input/id%EF%80%BA000000%2Csig%EF%80%BA11%2Csrc%EF%80%BA004450%2Ctime%EF%80%BA17949411%2Cexecs%EF%80%BA1756333%2Cop%EF%80%BAhavoc%2Crep%EF%80%BA14>