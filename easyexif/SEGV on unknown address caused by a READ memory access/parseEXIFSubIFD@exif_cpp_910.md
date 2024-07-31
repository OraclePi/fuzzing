Hello there , 
I've been wrting some fuzzing test scripts and the mutated jpg file generated has lead to a SEGV in 
`easyexif::EXIFInfo::parseEXIFSubIFD(unsigned char const*, unsigned int, unsigned int) exif.cpp:910` , caused by a invalid memory access in `:Rational::operator double (this=0x0) at exif.cpp:60`
, the ASAN details are as follows:
```sh
$ ./demo ./crash14.jpg
AddressSanitizer:DEADLYSIGNAL
=================================================================
==1702445==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000004 (pc 0x55a757cd7835 bp 0x7ffc8f1535c0 sp 0x7ffc8f1535b0 T0)
==1702445==The signal is caused by a READ memory access.
==1702445==Hint: address points to the zero page.
    #0 0x55a757cd7835 in operator double /mnt/c/Users/Orrr/Desktop/caveman_fuzzer/caveman_fuzzer/easyexif-master/easyexif-master/exif.cpp:60
    #1 0x55a757cdbda2 in easyexif::EXIFInfo::parseEXIFSubIFD(unsigned char const*, unsigned int, unsigned int) /mnt/c/Users/Orrr/Desktop/caveman_fuzzer/caveman_fuzzer/easyexif-master/easyexif-master/exif.cpp:910
    #2 0x55a757cd973f in easyexif::EXIFInfo::parseFromEXIFSegment(unsigned char const*, unsigned int) /mnt/c/Users/Orrr/Desktop/caveman_fuzzer/caveman_fuzzer/easyexif-master/easyexif-master/exif.cpp:581
    #3 0x55a757cd93bc in easyexif::EXIFInfo::parseFrom(unsigned char const*, unsigned int) /mnt/c/Users/Orrr/Desktop/caveman_fuzzer/caveman_fuzzer/easyexif-master/easyexif-master/exif.cpp:519
    #4 0x55a757ce9b5c in main /mnt/c/Users/Orrr/Desktop/caveman_fuzzer/caveman_fuzzer/easyexif-master/easyexif-master/demo.cpp:31
    #5 0x7f40c6c93d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #6 0x7f40c6c93e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #7 0x55a757cd7724 in _start (/mnt/c/Users/Orrr/Desktop/caveman_fuzzer/caveman_fuzzer/easyexif-master/easyexif-master/demo+0x3724)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /mnt/c/Users/Orrr/Desktop/caveman_fuzzer/caveman_fuzzer/easyexif-master/easyexif-master/exif.cpp:60 in operator double
==1702445==ABORTING
```

**env**:
```bash
g++ (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
Ubuntu 22.04.4 LTS
5.15.153.1-microsoft-standard-WSL2
```


To reproduce this :
makefile:
```makefile
CXX=g++
CXXFLAGS=-fsanitize=address -ggdb

all: demo

exif.o: exif.cpp
	$(CXX) $(CXXFLAGS) -c exif.cpp

demo: exif.o demo.cpp
	$(CXX) $(CXXFLAGS) -o demo exif.o demo.cpp

clean:
	rm -f *.o demo demo.exe
```

```sh
$ make
$ ./demo crash14.jpg
```

below are some gdb info for this SEGV

```sh
────────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]─────────────────────────────
*RAX  0x0
*RBX  0x7fffffffcc90 —▸ 0x7fffffffcd10 ◂— 0x41b58ab3
*RCX  0x7
*RDX  0x0
*RDI  0x0
*RSI  0x0
*R8   0x7fffffffcc30 —▸ 0x3ac0005a20f ◂— 0x0
 R9   0x0
*R10  0x7ffff44ea900 ◂— 0x0
*R11  0x7fffff7ff000 ◂— 0x7fffff7ff000
*R12  0x7fffffffcc10 ◂— 0x41b58ab3
*R13  0xffffffff982 ◂— 0x0
*R14  0x7fffffffcc10 ◂— 0x41b58ab3
 R15  0x7fffffffce30 ◂— 0x41b58ab3
*RBP  0x7fffffffcbd0 —▸ 0x7fffffffccb0 —▸ 0x7fffffffcdb0 —▸ 0x7fffffffcdf0 —▸ 0x7fffffffd460 ◂— ...
*RSP  0x7fffffffcbc0 ◂— 0x0
*RIP  0x555555557835 ((anonymous namespace)::Rational::operator double() const+75) ◂— mov eax, dword ptr [rax + 4]
─────────────────────────────────────[ DISASM / x86-64 / set emulate on ]──────────────────────────────────────
 ► 0x555555557835 <(anonymous namespace)::Rational::operator double() const+75>     mov    eax, dword ptr [rax + 4]
   0x555555557838 <(anonymous namespace)::Rational::operator double() const+78>     test   eax, eax
   0x55555555783a <(anonymous namespace)::Rational::operator double() const+80>     jne    555555557845h                 <(anonymous namespace)::Rational::operator double() const+91>
    ↓
   0x555555557845 <(anonymous namespace)::Rational::operator double() const+91>     mov    rax, qword ptr [rbp - 8]
   0x555555557849 <(anonymous namespace)::Rational::operator double() const+95>     mov    rdx, rax
   0x55555555784c <(anonymous namespace)::Rational::operator double() const+98>     shr    rdx, 3
   0x555555557850 <(anonymous namespace)::Rational::operator double() const+102>    add    rdx, 7fff8000h
   0x555555557857 <(anonymous namespace)::Rational::operator double() const+109>    movzx  edx, byte ptr [rdx]
   0x55555555785a <(anonymous namespace)::Rational::operator double() const+112>    test   dl, dl
   0x55555555785c <(anonymous namespace)::Rational::operator double() const+114>    setne  sil
   0x555555557860 <(anonymous namespace)::Rational::operator double() const+118>    mov    rcx, rax
───────────────────────────────────────────────[ SOURCE (CODE) ]───────────────────────────────────────────────
In file: /mnt/c/Users/Orrr/Desktop/caveman_fuzzer/caveman_fuzzer/easyexif-master/easyexif-master/exif.cpp
   55 struct Rational {
   56   uint32_t numerator = 0;
   57   uint32_t denominator = 0;
   58 
   59   operator double() const {
 ► 60     if (denominator < 1e-20) {
   61       return 0;
   62     }
```