## notation
* source code follows [Hungarian notation](https://www.cse.iitk.ac.in/users/dsrkg/cs245/html/Guide.htm)

## how to load the project?
* -- via -- IDE
  * | [Makefile](makefile) > right click > "Load the Makefile"
* -- via -- CL
  * TODO:

Compilers Supported
-------------------
MinGW GCC (v3.4.5)
Microsoft Visual C++ 6.0 SP6

Dependencies
------------
* Libraries / 
  * allows
    * build
  * you need to obtain SEPARATELY

| Library        | Default Path  | Download                                                                  |
|----------------|---------------|---------------------------------------------------------------------------|
| wxWidgets      | \wxWidgets    | http://www.wxwidgets.org/downloads/                                       |
| OpenSSL        | \OpenSSL      | http://www.openssl.org/source/                                            |
| Berkeley DB    | \DB           | http://www.oracle.com/technology/software/products/berkeley-db/index.html |
| Boost  v1.35.0 | \Boost        | http://www.boost.org/users/download/                                      |

OpenSSL
-------
* Bitcoin does NOT use encryption
* if you want to exclude encryption routines -> do a no-everything build of OpenSSL -> steps
  * OpenSSL v0.9.8h
  * add | "crypto\err\err_all.c" & `BEFORE void ERR_load_RSA_strings(void) { }`
    ```c++
    Edit engines\e_gmp.c and put this #ifndef around #include <openssl/rsa.h>
      #ifndef OPENSSL_NO_RSA
      #include <openssl/rsa.h>
      #endif
    ```
  * TODO:  

Edit ms\mingw32.bat and replace the Configure line's parameters with this
no-everything list.  You have to put this in the batch file because batch
files can't handle more than 9 parameters.
  perl Configure mingw threads no-rc2 no-rc4 no-rc5 no-idea no-des no-bf no-cast no-aes no-camellia no-seed no-rsa no-dh

Also REM out the following line in ms\mingw32.bat.  The build fails after it's
already finished building libeay32, which is all we care about, but the
failure aborts the script before it runs dllwrap to generate libeay32.dll.
  REM  if errorlevel 1 goto end

Build
  ms\mingw32.bat

If you want to use it with MSVC, generate the .lib file
  lib /machine:i386 /def:ms\libeay32.def /out:out\libeay32.lib


# Berkeley DB
## how to build?
* | MinGW + MSYS
  * `cd \DB\build_unix`
  * `sh ../dist/configure --enable-mingw --enable-cxx`
  * `make`
