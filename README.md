## Overview

Trying out the [Mosquitto](https://github.com/eclipse/mosquitto) MQTT broker.



## Building from source

### Prerequisites

* MacOS developer tools - either the command line or XCode
* [CMake](https://cmake.org/install/)


### OpenSSL

* Clone code from [github](https://github.com/openssl/openssl)
* Configure with the following command

```
./config --prefix=/usr/local
```

* Build using make

```
make
```

* Change to root and install

```
make install
```

### Configure Mosquitto

* I used the CMake GUI
* Selected the mosquitto source directory as source
* Created a `build` subdirectory and selected as target
* Added an entry 
 * Of type filepath
 * Name OPENSSL_INCLUDE_PATH
 * Value `/usr/local/include/openssl`
* Clicked the `configure` button
* Clicked the `generate` button
* In a terminal, navigate to the build directory
* Make the code by simply running 

```
make all
```

* Install everything as root by running the following

```
root# make install
```




## Troubleshooting

### CMake

I had some linker issues with OpenSSL/Mosquitto, for example this

```
[ 33%] Linking C shared library libmosquitto.dylib
Undefined symbols for architecture x86_64:
  "_ASN1_STRING_get0_data", referenced from:
      _mosquitto__verify_certificate_hostname in tls_mosq.c.o
  "_OPENSSL_init_crypto", referenced from:
      _net__init in net_mosq.c.o
  "_OPENSSL_sk_num", referenced from:
      _sk_X509_num in net_mosq_ocsp.c.o
      _sk_GENERAL_NAME_num in tls_mosq.c.o
  "_OPENSSL_sk_pop_free", referenced from:
      _sk_GENERAL_NAME_pop_free in tls_mosq.c.o
  "_OPENSSL_sk_value", referenced from:
      _sk_GENERAL_NAME_value in tls_mosq.c.o
  "_SSL_CTX_set_options", referenced from:
      _net__init_ssl_ctx in net_mosq.c.o
  "_SSL_CTX_set_psk_client_callback", referenced from:
      _net__init_ssl_ctx in net_mosq.c.o
  "_SSL_CTX_up_ref", referenced from:
      _mosquitto_opts_set in options.c.o
      _mosquitto_void_option in options.c.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make[2]: *** [lib/libmosquitto.1.6.7.dylib] Error 1
make[1]: *** [lib/CMakeFiles/libmosquitto.dir/all] Error 2
make: *** [all] Error 2
```

This appears to be because CMake found the `/usr/lib/libcrypto.dylib` file and used that rather than the copy under `/usr/local/lib` which matches the OpenSSL headers.

To fix the issue I used File -> Delete Cache and started the process again bu pressing `Configure` then `Generate`


### Man pages

This needed downloaded copies of XSL files, also the use of [XML Catalogs](http://xmlsoft.org/catalog.html)

The man pages didn't seem to get build by default, when I tried to force them to build I got the following

```
cd man
make 
xsltproc --nonet libmosquitto.3.xml
warning: failed to load external entity "http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl"
compilation error: file manpage.xsl line 3 element import
xsl:import : unable to load http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl
compilation error: file libmosquitto.3.xml line 4 element refentry
xsltParseStylesheetProcess : document is not a stylesheet
make: *** [libmosquitto.3] Error 5
```

It seems that the `--nonet` is a common option to avoid files being downloaded randomly.

The solution is to download copies of the needed files and use a [catalog](http://xmlsoft.org/catalog.html) file to reference these.  The latest DocBook releases can be downloaded from [SourceForge](https://sourceforge.net/projects/docbook/files/) specifically the [XSL release](https://sourceforge.net/projects/docbook/files/docbook-xsl/)

An example catalog file is provided under `xmlcatalog`.  This should be referenced by the environment variable as follows

```
export XML_CATALOG_FILES=/Users/sheila/wd/mosquitto_client_examples/xmlcatalog/catalog.xml
```




## References


