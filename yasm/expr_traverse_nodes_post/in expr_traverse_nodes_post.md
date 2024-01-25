#### Description:
~~~shell
 AddressSanitizer: heap-use-after-free /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/expr.c:1116:20 in expr_traverse_nodes_post
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
fk@localhost:/mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/crashes$ ./yasm id_000000,sig_06,src_005623,time_11506018,execs_3448900,op_havoc,rep_1
yasm: file name already has no extension: output will be in `yasm.out'
=================================================================
==27760==ERROR: AddressSanitizer: heap-use-after-free on address 0x6060000005d0 at pc 0x562d0f641065 bp 0x7ffd4c6ac830 sp 0x7ffd4c6ac828
READ of size 4 at 0x6060000005d0 thread T0
    #0 0x562d0f641064 in expr_traverse_nodes_post /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/expr.c:1116:20
    #1 0x562d0f645809 in yasm_expr_destroy /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/expr.c:1049:5
    #2 0x562d0f645809 in expr_delete_term /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/expr.c:1028:17
    #3 0x562d0f645809 in expr_simplify_identity /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/expr.c:582:17
    #4 0x562d0f63d501 in expr_level_op /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/expr.c:702:17
    #5 0x562d0f63fe04 in expr_level_tree /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/expr.c:881:9
    #6 0x562d0f63ee6f in yasm_expr__level_tree /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/expr.c:907:9
    #7 0x562d0f568569 in yasm_value_finalize /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/value.c:546:18
    #8 0x562d0f636edd in bc_data_finalize /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/bc-data.c:117:21
    #9 0x562d0f63a9a4 in yasm_bc_finalize /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/bytecode.c:176:9
    #10 0x562d0f5594bd in yasm_object_finalize /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/section.c:546:13
    #11 0x562d0f508b2a in do_assemble /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/frontends/yasm/yasm.c:647:5
    #12 0x7f0dd706bd8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #13 0x7f0dd706be3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #14 0x562d0f4454b4 in _start (/mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/crashes/yasm+0xba4b4) (BuildId: 4ca98d63b28f16b0ca05fb7a63c14ceb11a123a9)

0x6060000005d0 is located 16 bytes inside of 56-byte region [0x6060000005c0,0x6060000005f8)
freed by thread T0 here:
    #0 0x562d0f4cac32 in free (/mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/crashes/yasm+0x13fc32) (BuildId: 4ca98d63b28f16b0ca05fb7a63c14ceb11a123a9)
    #1 0x562d0f64103a in expr_destroy_each /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/expr.c:1041:5
    #2 0x562d0f64103a in expr_traverse_nodes_post /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/expr.c:1123:12

previously allocated by thread T0 here:
    #0 0x562d0f4caede in __interceptor_malloc (/mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/crashes/yasm+0x13fede) (BuildId: 4ca98d63b28f16b0ca05fb7a63c14ceb11a123a9)
    #1 0x562d0f56c153 in def_xmalloc /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/xmalloc.c:69:14

SUMMARY: AddressSanitizer: heap-use-after-free /mnt/c/Users/Orrr/Desktop/Fuzz/yasm-master/yasm-master/libyasm/expr.c:1116:20 in expr_traverse_nodes_post
Shadow bytes around the buggy address:
  0x0c0c7fff8060: fa fa fa fa fd fd fd fd fd fd fd fa fa fa fa fa
  0x0c0c7fff8070: fd fd fd fd fd fd fd fa fa fa fa fa fd fd fd fd
  0x0c0c7fff8080: fd fd fd fa fa fa fa fa fd fd fd fd fd fd fd fa
  0x0c0c7fff8090: fa fa fa fa fd fd fd fd fd fd fd fa fa fa fa fa
  0x0c0c7fff80a0: fd fd fd fd fd fd fd fa fa fa fa fa fd fd fd fd
=>0x0c0c7fff80b0: fd fd fd fa fa fa fa fa fd fd[fd]fd fd fd fd fa
  0x0c0c7fff80c0: fa fa fa fa fd fd fd fd fd fd fd fa fa fa fa fa
  0x0c0c7fff80d0: fd fd fd fd fd fd fd fa fa fa fa fa fd fd fd fd
  0x0c0c7fff80e0: fd fd fd fa fa fa fa fa fd fd fd fd fd fd fd fa
  0x0c0c7fff80f0: fa fa fa fa 00 00 00 00 00 00 00 fa fa fa fa fa
  0x0c0c7fff8100: 00 00 00 00 00 00 00 fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==27760==ABORTING
~~~

#### POC:
<https://github.com/OraclePi/fuzzing/blob/main/yasm/expr_traverse_nodes_post/input/id_000000%2Csig_06%2Csrc_005623%2Ctime_11506018%2Cexecs_3448900%2Cop_havoc%2Crep_1>