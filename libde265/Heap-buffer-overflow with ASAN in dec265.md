#### Description:
~~~bash
AddressSanitizer: heap-buffer-overflow (/src/Fuzz/libde265-master/libde265-master/dec265/dec265+0xa8396) (BuildId: 60bba1fc7b10411972a59b7f0baa995877dbfe4b) in __asan_memcpy
~~~

#### Project:
<https://github.com/strukturag/libde265>

#### Version:
~~~bash
fk@localhost:/mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/dec265$ ./dec265 -v
 dec265  v1.0.15
-----------------
usage: dec265 [options] videofile.bin
The video file must be a raw bitstream, or a stream with NAL units (option -n).

options:
  -q, --quiet       do not show decoded image
  -t, --threads N   set number of worker threads (0 - no threading)
  -c, --check-hash  perform hash check
  -n, --nal         input is a stream with 4-byte length prefixed NAL units
  -f, --frames N    set number of frames to process
  -o, --output      write YUV reconstruction
  -d, --dump        dump headers
  -0, --noaccel     do not use any accelerated code (SSE)
  -v, --verbose     increase verbosity level (up to 3 times)
  -L, --no-logging  disable logging
  -B, --write-bytestream FILENAME  write raw bytestream (from NAL input)
  -m, --measure YUV compute PSNRs relative to reference YUV
  -T, --highest-TID select highest temporal sublayer to decode
      --disable-deblocking   disable deblocking filter
      --disable-sao          disable sample-adaptive offset filter
  -h, --help        show help
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
export CC=afl-clang-fast CXX=afl-clang-fast++ AFL_USE_ASAN=1
cmake . && make
./dec265/dec265 ./poc.in
~~~


#### ASAN:
~~~bash
fk@localhost:/mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/dec265$ ./dec265 poc.in
=================================================================
==19720==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x63100003a010 at pc 0x5555555fc397 bp 0x7fffffffd8d0 sp 0x7fffffffd0a0
READ of size 312 at 0x63100003a010 thread T0
    #0 0x5555555fc396 in __asan_memcpy (/mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/dec265/dec265+0xa8396) (BuildId: 60bba1fc7b10411972a59b7f0baa995877dbfe4b)
    #1 0x5555556407de in SDL_YUV_Display::display420(unsigned char const*, unsigned char const*, unsigned char const*, int, int) /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/dec265/sdl.cc:146:9
    #2 0x55555563fd39 in SDL_YUV_Display::display(unsigned char const*, unsigned char const*, unsigned char const*, int, int) /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/dec265/sdl.cc:107:5
    #3 0x55555563c30d in display_sdl(de265_image const*) /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/dec265/dec265.cc:310:10
    #4 0x55555563ca87 in output_image(de265_image const*) /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/dec265/dec265.cc:353:12
    #5 0x55555563eaaf in main /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/dec265/dec265.cc:802:20
    #6 0x7ffff75f9d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #7 0x7ffff75f9e3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #8 0x555555577664 in _start (/mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/dec265/dec265+0x23664) (BuildId: 60bba1fc7b10411972a59b7f0baa995877dbfe4b)

0x63100003a010 is located 0 bytes to the right of 71696-byte region [0x631000028800,0x63100003a010)
allocated by thread T0 here:
    #0 0x5555555fdbf7 in posix_memalign (/mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/dec265/dec265+0xa9bf7) (BuildId: 60bba1fc7b10411972a59b7f0baa995877dbfe4b)
    #1 0x7ffff7dcb599 in ALLOC_ALIGNED(unsigned long, unsigned long) /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/libde265/image.cc:55:9
    #2 0x7ffff7dcb599 in de265_image_get_buffer(void*, de265_image_spec*, de265_image*, void*) /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/libde265/image.cc:129:21
    #3 0x7ffff7dcd616 in de265_image::alloc_image(int, int, de265_chroma, std::shared_ptr<seq_parameter_set const>, bool, decoder_context*, long, void*, bool) /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/libde265/image.cc:393:25
    #4 0x7ffff7d9513d in decoded_picture_buffer::new_image(std::shared_ptr<seq_parameter_set const>, decoder_context*, long, void*, bool) /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/libde265/dpb.cc:266:28
    #5 0x7ffff7d7da2a in decoder_context::process_slice_segment_header(slice_segment_header*, de265_error*, long, nal_header*, void*) /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/libde265/decctx.cc:2039:28
    #6 0x7ffff7d7ad85 in decoder_context::read_slice_NAL(bitreader&, NAL_unit*, nal_header&) /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/libde265/decctx.cc:650:7
    #7 0x7ffff7d85d14 in decoder_context::decode_NAL(NAL_unit*) /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/libde265/decctx.cc:1241:11
    #8 0x7ffff7d86730 in decoder_context::decode(int*) /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/libde265/decctx.cc:1329:16
    #9 0x55555563e8e5 in main /mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/dec265/dec265.cc:784:17
    #10 0x7ffff75f9d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: heap-buffer-overflow (/mnt/c/Users/Orrr/Desktop/Fuzz/libde265-master/libde265-master/dec265/dec265+0xa8396) (BuildId: 60bba1fc7b10411972a59b7f0baa995877dbfe4b) in __asan_memcpy
Shadow bytes around the buggy address:
  0x0c627ffff3b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c627ffff3c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c627ffff3d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c627ffff3e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c627ffff3f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c627ffff400: 00 00[fa]fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c627ffff410: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c627ffff420: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c627ffff430: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c627ffff440: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c627ffff450: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==19720==ABORTING
~~~

#### POC:
<https://github.com/OraclePi/fuzzing/blob/main/libde265/input/poc.in>