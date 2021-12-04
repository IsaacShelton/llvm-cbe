# IsaacShelton/llvm-cbe

Contains bugs fixes and improvements upon `JuliaComputingOSS/llvm-cbe`

Takes LLVM SSA intermediate representation and elevates it back up to C code

For example, you can start off with a `.c` file:
```
#include <CoreFoundation/CoreFoundation.h>

void macMessageBox(const char *title, const char *text)
{
    SInt32 nRes = 0;
    CFUserNotificationRef pDlg = NULL;
    const void *keys[] = {kCFUserNotificationAlertHeaderKey, kCFUserNotificationAlertMessageKey};
    CFStringRef cfTitle = CFStringCreateWithCString(NULL, title, kCFStringEncodingWindowsLatin1);
    CFStringRef cfText = CFStringCreateWithCString(NULL, text, kCFStringEncodingWindowsLatin1);
    const void *vals[] = {
        cfTitle, cfText
    };

    CFDictionaryRef dict = CFDictionaryCreate(0, keys, vals, sizeof(keys) / sizeof(*keys), &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
    pDlg = CFUserNotificationCreate(kCFAllocatorDefault, 0, kCFUserNotificationStopAlertLevel, &nRes, dict);
    CFRelease(cfTitle);
    CFRelease(cfText);
    CFRelease(pDlg);
}

int main(int argc, const char** argv)
{
    for(int i = 0; i != 3; i++){
        const char *something = argv[0];
        macMessageBox(something, "Whatever this is");
    }
    return 0;
}
```

Then generate the low level SAA intermediate representation for it with clang:
```
; ModuleID = 'alert.c'
source_filename = "alert.c"
target datalayout = "e-m:o-i64:64-i128:128-n32:64-S128"
target triple = "arm64-apple-macosx11.0.0"

%struct.__CFString = type opaque
%struct.CFDictionaryKeyCallBacks = type { i64, i8* (%struct.__CFAllocator*, i8*)*, void (%struct.__CFAllocator*, i8*)*, %struct.__CFString* (i8*)*, i8 (i8*, i8*)*, i64 (i8*)* }
%struct.__CFAllocator = type opaque
%struct.CFDictionaryValueCallBacks = type { i64, i8* (%struct.__CFAllocator*, i8*)*, void (%struct.__CFAllocator*, i8*)*, %struct.__CFString* (i8*)*, i8 (i8*, i8*)* }
%struct.__CFUserNotification = type opaque
%struct.__CFDictionary = type opaque

@kCFUserNotificationAlertHeaderKey = external constant %struct.__CFString*, align 8
@kCFUserNotificationAlertMessageKey = external constant %struct.__CFString*, align 8
@kCFTypeDictionaryKeyCallBacks = external constant %struct.CFDictionaryKeyCallBacks, align 8
@kCFTypeDictionaryValueCallBacks = external constant %struct.CFDictionaryValueCallBacks, align 8
@kCFAllocatorDefault = external constant %struct.__CFAllocator*, align 8
@.str = private unnamed_addr constant [17 x i8] c"Whatever this is\00", align 1

; Function Attrs: noinline nounwind optnone ssp uwtable
define void @macMessageBox(i8* %0, i8* %1) #0 {
  %3 = alloca i8*, align 8
  %4 = alloca i8*, align 8
  %5 = alloca i32, align 4
  %6 = alloca %struct.__CFUserNotification*, align 8
  %7 = alloca [2 x i8*], align 8
  %8 = alloca %struct.__CFString*, align 8
  %9 = alloca %struct.__CFString*, align 8
  %10 = alloca [2 x i8*], align 8
  %11 = alloca %struct.__CFDictionary*, align 8
  store i8* %0, i8** %3, align 8
  store i8* %1, i8** %4, align 8
  store i32 0, i32* %5, align 4
  store %struct.__CFUserNotification* null, %struct.__CFUserNotification** %6, align 8
  %12 = getelementptr inbounds [2 x i8*], [2 x i8*]* %7, i64 0, i64 0
  %13 = load %struct.__CFString*, %struct.__CFString** @kCFUserNotificationAlertHeaderKey, align 8
  %14 = bitcast %struct.__CFString* %13 to i8*
  store i8* %14, i8** %12, align 8
  %15 = getelementptr inbounds i8*, i8** %12, i64 1
  %16 = load %struct.__CFString*, %struct.__CFString** @kCFUserNotificationAlertMessageKey, align 8
  %17 = bitcast %struct.__CFString* %16 to i8*
  store i8* %17, i8** %15, align 8
  %18 = load i8*, i8** %3, align 8
  %19 = call %struct.__CFString* @CFStringCreateWithCString(%struct.__CFAllocator* null, i8* %18, i32 1280)
  store %struct.__CFString* %19, %struct.__CFString** %8, align 8
  %20 = load i8*, i8** %4, align 8
  %21 = call %struct.__CFString* @CFStringCreateWithCString(%struct.__CFAllocator* null, i8* %20, i32 1280)
  store %struct.__CFString* %21, %struct.__CFString** %9, align 8
  %22 = getelementptr inbounds [2 x i8*], [2 x i8*]* %10, i64 0, i64 0
  %23 = load %struct.__CFString*, %struct.__CFString** %8, align 8
  %24 = bitcast %struct.__CFString* %23 to i8*
  store i8* %24, i8** %22, align 8
  %25 = getelementptr inbounds i8*, i8** %22, i64 1
  %26 = load %struct.__CFString*, %struct.__CFString** %9, align 8
  %27 = bitcast %struct.__CFString* %26 to i8*
  store i8* %27, i8** %25, align 8
  %28 = getelementptr inbounds [2 x i8*], [2 x i8*]* %7, i64 0, i64 0
  %29 = getelementptr inbounds [2 x i8*], [2 x i8*]* %10, i64 0, i64 0
  %30 = call %struct.__CFDictionary* @CFDictionaryCreate(%struct.__CFAllocator* null, i8** %28, i8** %29, i64 2, %struct.CFDictionaryKeyCallBacks* @kCFTypeDictionaryKeyCallBacks, %struct.CFDictionaryValueCallBacks* @kCFTypeDictionaryValueCallBacks)
  store %struct.__CFDictionary* %30, %struct.__CFDictionary** %11, align 8
  %31 = load %struct.__CFAllocator*, %struct.__CFAllocator** @kCFAllocatorDefault, align 8
  %32 = load %struct.__CFDictionary*, %struct.__CFDictionary** %11, align 8
  %33 = call %struct.__CFUserNotification* @CFUserNotificationCreate(%struct.__CFAllocator* %31, double 0.000000e+00, i64 0, i32* %5, %struct.__CFDictionary* %32)
  store %struct.__CFUserNotification* %33, %struct.__CFUserNotification** %6, align 8
  %34 = load %struct.__CFString*, %struct.__CFString** %8, align 8
  %35 = bitcast %struct.__CFString* %34 to i8*
  call void @CFRelease(i8* %35)
  %36 = load %struct.__CFString*, %struct.__CFString** %9, align 8
  %37 = bitcast %struct.__CFString* %36 to i8*
  call void @CFRelease(i8* %37)
  %38 = load %struct.__CFUserNotification*, %struct.__CFUserNotification** %6, align 8
  %39 = bitcast %struct.__CFUserNotification* %38 to i8*
  call void @CFRelease(i8* %39)
  ret void
}

declare %struct.__CFString* @CFStringCreateWithCString(%struct.__CFAllocator*, i8*, i32) #1

declare %struct.__CFDictionary* @CFDictionaryCreate(%struct.__CFAllocator*, i8**, i8**, i64, %struct.CFDictionaryKeyCallBacks*, %struct.CFDictionaryValueCallBacks*) #1

declare %struct.__CFUserNotification* @CFUserNotificationCreate(%struct.__CFAllocator*, double, i64, i32*, %struct.__CFDictionary*) #1

declare void @CFRelease(i8*) #1

; Function Attrs: noinline nounwind optnone ssp uwtable
define i32 @main(i32 %0, i8** %1) #0 {
  %3 = alloca i32, align 4
  %4 = alloca i32, align 4
  %5 = alloca i8**, align 8
  %6 = alloca i32, align 4
  %7 = alloca i8*, align 8
  store i32 0, i32* %3, align 4
  store i32 %0, i32* %4, align 4
  store i8** %1, i8*** %5, align 8
  store i32 0, i32* %6, align 4
  br label %8

8:                                                ; preds = %16, %2
  %9 = load i32, i32* %6, align 4
  %10 = icmp ne i32 %9, 3
  br i1 %10, label %11, label %19

11:                                               ; preds = %8
  %12 = load i8**, i8*** %5, align 8
  %13 = getelementptr inbounds i8*, i8** %12, i64 0
  %14 = load i8*, i8** %13, align 8
  store i8* %14, i8** %7, align 8
  %15 = load i8*, i8** %7, align 8
  call void @macMessageBox(i8* %15, i8* getelementptr inbounds ([17 x i8], [17 x i8]* @.str, i64 0, i64 0))
  br label %16

16:                                               ; preds = %11
  %17 = load i32, i32* %6, align 4
  %18 = add nsw i32 %17, 1
  store i32 %18, i32* %6, align 4
  br label %8

19:                                               ; preds = %8
  ret i32 0
}

attributes #0 = { noinline nounwind optnone ssp uwtable "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "frame-pointer"="non-leaf" "less-precise-fpmad"="false" "min-legal-vector-width"="0" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="true" "probe-stack"="__chkstk_darwin" "stack-protector-buffer-size"="8" "target-cpu"="apple-a12" "target-features"="+aes,+crc,+crypto,+fp-armv8,+fullfp16,+lse,+neon,+ras,+rcpc,+rdm,+sha2,+v8.3a,+zcm,+zcz" "unsafe-fp-math"="false" "use-soft-float"="false" }
attributes #1 = { "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "frame-pointer"="non-leaf" "less-precise-fpmad"="false" "no-infs-fp-math"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="true" "probe-stack"="__chkstk_darwin" "stack-protector-buffer-size"="8" "target-cpu"="apple-a12" "target-features"="+aes,+crc,+crypto,+fp-armv8,+fullfp16,+lse,+neon,+ras,+rcpc,+rdm,+sha2,+v8.3a,+zcm,+zcz" "unsafe-fp-math"="false" "use-soft-float"="false" }

!llvm.module.flags = !{!0, !1, !2}
!llvm.ident = !{!3}

!0 = !{i32 2, !"SDK Version", [2 x i32] [i32 11, i32 3]}
!1 = !{i32 1, !"wchar_size", i32 4}
!2 = !{i32 7, !"PIC Level", i32 2}
!3 = !{!"Apple clang version 12.0.5 (clang-1205.0.22.11)"}
```
And then by using `llvm-cbe` you can elevate it back up to non-human-readable C code:
```
/* Provide Declarations */
#include <stdint.h>
#ifndef __cplusplus
typedef unsigned char bool;
#endif

#ifndef _MSC_VER
#define __forceinline __attribute__((always_inline)) inline
#endif

#if defined(__GNUC__)
#define  __ATTRIBUTELIST__(x) __attribute__(x)
#else
#define  __ATTRIBUTELIST__(x)  
#endif

#ifdef _MSC_VER  /* Can only support "linkonce" vars with GCC */
#define __attribute__(X)
#endif



/* Global Declarations */

/* Types Declarations */
struct l_struct_struct_OC_CFDictionaryKeyCallBacks;
struct l_struct_struct_OC_CFDictionaryValueCallBacks;

/* Function definitions */
typedef void l_fptr_2(void*, uint8_t*);

typedef uint64_t l_fptr_5(uint8_t*);

typedef uint8_t* l_fptr_1(void*, uint8_t*);

typedef void* l_fptr_3(uint8_t*);

typedef uint8_t l_fptr_4(uint8_t*, uint8_t*);


/* Types Definitions */
struct l_struct_struct_OC_CFDictionaryKeyCallBacks {
  uint64_t field0;
  l_fptr_1* field1;
  l_fptr_2* field2;
  l_fptr_3* field3;
  l_fptr_4* field4;
  l_fptr_5* field5;
};
struct l_struct_struct_OC_CFDictionaryValueCallBacks {
  uint64_t field0;
  l_fptr_1* field1;
  l_fptr_2* field2;
  l_fptr_3* field3;
  l_fptr_4* field4;
};
struct l_array_17_uint8_t {
  uint8_t array[17];
};
struct l_array_2_uint8_t_KC_ {
  uint8_t* array[2];
};

/* External Global Variable Declarations */
extern void* kCFUserNotificationAlertHeaderKey;
extern void* kCFUserNotificationAlertMessageKey;
extern struct l_struct_struct_OC_CFDictionaryKeyCallBacks kCFTypeDictionaryKeyCallBacks;
extern struct l_struct_struct_OC_CFDictionaryValueCallBacks kCFTypeDictionaryValueCallBacks;
extern void* kCFAllocatorDefault;

/* Function Declarations */
void macMessageBox(uint8_t*, uint8_t*) __ATTRIBUTELIST__((noinline, nothrow, stack_protect));
void* CFStringCreateWithCString(void*, uint8_t*, uint32_t);
void* CFDictionaryCreate(void*, uint8_t**, uint8_t**, uint64_t, struct l_struct_struct_OC_CFDictionaryKeyCallBacks*, struct l_struct_struct_OC_CFDictionaryValueCallBacks*);
void* CFUserNotificationCreate(void*, double, uint64_t, uint32_t*, void*);
void CFRelease(uint8_t*);
int main(int, char **) __ATTRIBUTELIST__((noinline, nothrow, stack_protect));


/* Global Variable Definitions and Initialization */
static struct l_array_17_uint8_t _OC_str = { "Whatever this is" };


/* LLVM Intrinsic Builtin Function Bodies */
static __forceinline uint32_t llvm_add_u32(uint32_t a, uint32_t b) {
  uint32_t r = a + b;
  return r;
}


/* Function Bodies */

void macMessageBox(uint8_t* _1, uint8_t* _2) {
  uint8_t* _3;    /* Address-exposed local */
  uint8_t* _4;    /* Address-exposed local */
  uint32_t _5;    /* Address-exposed local */
  void* _6;    /* Address-exposed local */
  struct l_array_2_uint8_t_KC_ _7;    /* Address-exposed local */
  void* _8;    /* Address-exposed local */
  void* _9;    /* Address-exposed local */
  struct l_array_2_uint8_t_KC_ _10;    /* Address-exposed local */
  void* _11;    /* Address-exposed local */
  uint8_t** _12;
  void* _13;
  void* _14;
  uint8_t* _15;
  void* _16;
  uint8_t* _17;
  void* _18;
  uint8_t** _19;
  void* _20;
  void* _21;
  void* _22;
  void* _23;
  void* _24;
  void* _25;
  void* _26;
  void* _27;
  void* _28;

  _3 = _1;
  _4 = _2;
  _5 = 0;
  _6 = ((void*)/*NULL*/0);
  _12 = (&_7.array[((int64_t)0)]);
  _13 = kCFUserNotificationAlertHeaderKey;
  *_12 = (((uint8_t*)_13));
  _14 = kCFUserNotificationAlertMessageKey;
  *((&_12[((int64_t)1)])) = (((uint8_t*)_14));
  _15 = _3;
  _16 = CFStringCreateWithCString(((void*)/*NULL*/0), _15, 1280);
  _8 = _16;
  _17 = _4;
  _18 = CFStringCreateWithCString(((void*)/*NULL*/0), _17, 1280);
  _9 = _18;
  _19 = (&_10.array[((int64_t)0)]);
  _20 = _8;
  *_19 = (((uint8_t*)_20));
  _21 = _9;
  *((&_19[((int64_t)1)])) = (((uint8_t*)_21));
  _22 = CFDictionaryCreate(((void*)/*NULL*/0), ((&_7.array[((int64_t)0)])), ((&_10.array[((int64_t)0)])), 2, (&kCFTypeDictionaryKeyCallBacks), (&kCFTypeDictionaryValueCallBacks));
  _11 = _22;
  _23 = kCFAllocatorDefault;
  _24 = _11;
  _25 = CFUserNotificationCreate(_23, 0, 0, (&_5), _24);
  _6 = _25;
  _26 = _8;
  CFRelease((((uint8_t*)_26)));
  _27 = _9;
  CFRelease((((uint8_t*)_27)));
  _28 = _6;
  CFRelease((((uint8_t*)_28)));
}


int main(int argc, char ** argv) {
  uint32_t _29 = (uint32_t)argc;
  uint8_t** _30 = (uint8_t**)argv;
  uint32_t _31;    /* Address-exposed local */
  uint32_t _32;    /* Address-exposed local */
  uint8_t** _33;    /* Address-exposed local */
  uint32_t _34;    /* Address-exposed local */
  uint8_t* _35;    /* Address-exposed local */
  uint32_t _36;
  uint8_t** _37;
  uint8_t* _38;
  uint8_t* _39;
  uint32_t _40;

  _31 = 0;
  _32 = _29;
  _33 = _30;
  _34 = 0;
  goto _41;

  do {     /* Syntactic loop '' to make GCC happy */
_41:
  _36 = _34;
  if ((_36 != 3u)) {
    goto _42;
  } else {
    goto _43;
  }

_42:
  _37 = _33;
  _38 = *((&(*_37)));
  _35 = _38;
  _39 = _35;
  macMessageBox(_39, ((&_OC_str.array[((int64_t)0)])));
  goto _44;

_44:
  _40 = _34;
  _34 = (llvm_add_u32(_40, 1));
  goto _41;

  } while (1); /* end of syntactic loop '' */
_43:
  return 0;
}
```
And then compile and link it as if it was the original `.c` file.



------------------------------

llvm-cbe
========

This LLVM C backend has been resurrected by Julia Computing with various improvements.

Installation instructions
=========================

This version of the LLVM C backend works with LLVM 10.0, and has preliminary support for LLVM 11.0.

Step 1: Installing LLVM
=======================

Either install the LLVM packages on your system:
--------------------------------------------

On macOS, use [pkgsrc](http://pkgsrc.joyent.com/install-on-osx/) and run the following commands:
```
    pkgin in llvm clang
```

On CentOS, install the llvm-devel package:
```
    dnf install llvm-devel clang
```

On Debian and derivatives, install the llvm-dev package via:
```
    apt install llvm-dev clang
```

Or compile LLVM yourself:
-----------------------------
Note: to convert C to LLVM IR to run the tests, you will also need a C compiler using the LLVM infrastructure, such as clang.

The first step is to compile LLVM on your machine
(this assumes an in-tree build, but out-of-tree will also work):


     git clone https://github.com/llvm/llvm-project.git
     cd llvm-project
     git checkout release/8.x
     mkdir llvm/build
     cd llvm/build
     cmake ..
     make

To run tests, you need to build `lli`.


Step 2: Compiling LLVM-CBE
==========================

Now you can download and compile llvm-cbe.

If you built LLVM yourself, put it in the same folder you built LLVM in:

    cd $HOME/llvm-project/llvm/projects
    git clone https://github.com/JuliaComputing/llvm-cbe
    cd ../build
    cmake -S ..
    make llvm-cbe

If you used your distribution's package, put it wherever you feel like:

    git clone https://github.com/JuliaComputing/llvm-cbe
    cd llvm-cbe && mkdir build && cd build
    cmake -S ..
    make llvm-cbe

Step 3: Usage Examples
======================

If llvm-cbe compiles, you should be able to run it with the following commands.
```
$ cd $HOME/llvm-project/llvm/projects/llvm-cbe/test/selectionsort
$ ls
main.c
$ clang -S -emit-llvm -g main.c
$ ls
main.c main.ll
$ $(HOME)/llvm/build/bin/llvm-cbe main.ll
```

You can find options to configure the C backend's output with `llvm-cbe --help`.
Look for options beginning with `--cbe-`.

Compile Generated C Code and Run
================================

```
$ gcc -o main.cbe main.cbe.c
$ ls
main.c  main.cbe  main.cbe.c  main.ll
$ ./main.cbe
```

Running tests
==================

Unit tests:

```sh
    $ cd $HOME/llvm-project/llvm/build
    $ make CBEUnitTests && projects/llvm-cbe/unittests/CWriterTest
```

Note that you need to have passed `-DLLVM_INCLUDE_TESTS=1` to cmake if you used
your distribution's LLVM package. You also will need to install gtest (on Debian
derivatives: `apt install libgtest-dev`).

Other tests:

First, compile llvm-cbe, and install pytest (e.g. `pip install pytest`). Then:

```sh
    $ cd $HOME/llvm-project/llvm/projects/llvm-cbe
    $ pytest
```

You might have to adjust the llvm-cbe and lli paths in that configuration.

If you want the tests to run faster, installing `pytest-xdist` will allow you to run the test suite in parallel, e.g. `pytest -n 4` if you want to use 4 cores.
