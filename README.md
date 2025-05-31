# BearSSL C++ wrapper
Europa Software's C++ wrapper for BearSSL provides access to this fantastic library in a simpler way, more suited to modern C++ code.</br> 
At the moment the abstractions are primitive, and usability can be further improved (meaning that suggestions and PRs are welcome).</br>
- The wrapper is fully functional, and is used in the [Europa Software Webserver](https://github.com/Europasoft/europasoft-network) to provide HTTPS-TLS capabilities.
- BearSSL headers are only included in TLSInterface.cpp, meaning that other C++ source files will not be contaminated with C declarations from the underlying library. TLSInterface.h provides the wrapper's public interface.
- Currently only serverside is supported.
- Importing keys from a file is not yet supported.

## Usage

### Initialization
Create an object of the TLSContext class. This class handles the state for one connection. </br>
In multithreaded and multi-client environments, multiple independent contexts can be created without the need for synchronization.</br> 
The TLSContext class and related data types reside in the "Encryption" namespace.
```C++
using namespace Encryption;
auto tlscontext = TLSContext(TLSKeyType::RSA, CipherSuiteMode::FULL)
```
Its constructor requires an `Encryption::TLSKeyType`, and an `Encryption::CipherSuiteMode`. </br>
The supported key types are RSA and EC (Elliptic Curve), setting this to EC_MIXED may also be required if your key and the issuing Certificate Authority's key are of different types. </br>
The *`mode`* argument can be changed to load a more lightweight set of encryption algorithms if desired. </br>

### Data flow
The wrapper mainly consist of a set of functions which are used to push data into - and pull data out of - the library.</br>
Below is a list of these public functions

- #### getEncryptedOutgoing()
Gets encrypted data, to be sent to a client
```C++
std::string encryptedToSend;
if (tlscontext->getEncryptedOutgoing(encryptedToSend))
{
    // send data to client
}
```

- #### pushEncryptedIncoming() and canPushIncoming()
Pushes encrypted data received from a client, to be decrypted by BearSSL
```C++
// receive data from client
std::string encryptedReceived;
const size_t pushed = tlscontext->pushEncryptedIncoming(encryptedReceived.data(), encryptedReceived.size());
// check how much data was pushed, handle remaining bytes
```

- #### pushOutgoing() and canPushOutgoing()
Push outgoing data to be encrypted, instead of sending it directly to the client
```C++ 
std::string dataToEncrypt;
if (tlscontext->canPushOutgoing())
{
    const size_t sizeToPush = std::min(tlscontext->getPushMaxSizeOutgoing(), dataToEncrypt.size());
    const size_t sizePushed = tlscontext->pushOutgoing(dataToEncrypt.data(), sizeToPush);
    // check how much data was pushed, handle remaining bytes
}
```

- #### getDecryptedIncoming()
Get decrypted data received from a client
```C++
std::string incomingDecrypted;
if (tlscontext->getDecryptedIncoming(incomingDecrypted))
{
    // use data from client
}
```

- #### isClosed()
Indicates en error or closure of the session
```C++
if (tlscontext->isClosed())
{
    // handle connection closure and destroy the context object
}
```

# Original documentation
#### Below is the original documentation for BearSSL, not including the changes made to this fork.</br>The following information might not be fully applicable to this version.
The most up-to-date documentation is supposed to be available on the
[BearSSL Website](https://www.bearssl.org/).

## Disclaimer

BearSSL is considered beta-level software. Most planned functionalities
are implemented; new evolution may still break both source and binary
compatibility.

Using BearSSL for production purposes would be a relatively bold but not
utterly crazy move. BearSSL is free, open-source software, provided
without any guarantee of fitness or reliability. That being said, it
appears to behave properly, and only minor issues have been found (and
fixed) so far. You are encourage to inspect its API and code for
learning, testing and possibly contributing.

The usage license is explicited in the `LICENSE.txt` file. This is the
"MIT license". It can be summarised in the following way:

 - You can use and reuse the library as you wish, and modify it, and
   integrate it in your own code, and distribute it as is or in any
   modified form, and so on.

 - The only obligation that the license terms put upon you is that you
   acknowledge and make it clear that if anything breaks, it is not my
   fault, and I am not liable for anything, regardless of the type and
   amount of collateral damage. The license terms say that the copyright
   notice "shall be included in all copies or substantial portions of
   the Software": this is how the disclaimer is "made explicit".
   Basically, I have put it in every source file, so just keep it there.

## Installation

Right now, BearSSL is a simple library, along with a few test and debug
command-line tools. There is no installer yet. The library _can_ be
compiled as a shared library on some systems, but since the binary API
is not fully stabilised, this is not a very good idea to do that right
now.

To compile the code, just type `make`. This will try to use sane
"default" values. On a Windows system with Visual Studio, run a console
with the environment initialised for a specific version of the C compiler,
and type `nmake`.

To override the default settings, create a custom configuration file in
the `conf` directory, and invoke `make` (or `nmake`) with an explicit
`CONF=` parameter. For instance, to use the provided `samd20.mk`
configuration file (that targets cross-compilation for an Atmel board
that features a Cortex-M0+ CPU), type:

    make CONF=samd20

The `conf/samd20.mk` file includes the `Unix.mk` file and then overrides
some of the parameters, including the destination directory. Any custom
configuration can be made the same way.

Some compile-time options can be set through macros, either on the
compiler command-line, or in the `src/config.h` file. See the comments
in that file. Some settings are autodetected but they can still be
explicitly overridden.

When compilation is done, the library (static and DLL, when appropriate)
and the command-line tools can be found in the designated build
directory (by default named `build`). The public headers (to be used
by applications linked against BearSSL) are in the `inc/` directory.

To run the tests:

  - `testcrypto all` runs the cryptographic tests (test vectors on all
    implemented cryptogaphic functions). It can be slow. You can also
    run a selection of the tests by providing their names (run
    `testcrypto` without any parameter to see the available names).

  - `testspeed all` runs a number of performance benchmarks, there again
    on cryptographic functions. It gives a taste of how things go on the
    current platform. As for `testcrypto`, specific named benchmarks can
    be executed.

  - `testx509` runs X.509 validation tests. The test certificates are
    all in `test/x509/`.

The `brssl` command-line tool produced in the build directory is a
stand-alone binary. It can exercise some of the functionalities of
BearSSL, in particular running a test SSL client or server. It is not
meant for production purposes (e.g. the SSL client has a mode where it
disregards the inability to validate the server's certificate, which is
inherently unsafe, but convenient for debug).

**Using the library** means writing some application code that invokes
it, and linking with the static library. The header files are all in the
`inc` directory; copy them wherever makes sense (e.g. in the
`/usr/local/include` directory). The library itself (`libbearssl.a`) is
what you link against.

Alternatively, you may want to copy the source files directly into your
own application code. This will make integrating ulterior versions of
BearSSL more difficult. If you still want to go down that road, then
simply copy all the `*.h` and `*.c` files from the `src` and `inc`
directories into your application source code. In the BearSSL source
archive, the source files are segregated into various sub-directories,
but this is for my convenience only. There is no technical requirement
for that, and all files can be dumped together in a simple directory.

Dependencies are simple and systematic:

  - Each `*.c` file includes `inner.h`
  - `inner.h` includes `config.h` and `bearssl.h`
  - `bearssl.h` includes the other `bearssl_*.h`

## Versioning

I follow this simple version numbering scheme:

 - Version numbers are `x.y` or `x.y.z` where `x`, `y` and `z` are
   decimal integers (possibly greater than 10). When the `.z` part is
   missing, it is equivalent to `.0`.

 - Backward compatibility is maintained, at both source and binary levels,
   for each major version: this means that if some application code was
   designed for version `x.y`, then it should compile, link and run
   properly with any version `x.y'` for any `y'` greater than `y`.

   The major version `0` is an exception. You shall not expect that any
   version that starts with `0.` offers any kind of compatibility,
   either source or binary, with any other `0.` version. (Of course I
   will try to maintain some decent level of source compatibility, but I
   make no promise in that respect. Since the API uses caller-allocated
   context structures, I already know that binary compatibility _will_
   be broken.)

 - Sub-versions (the `y` part) are about added functionality. That is,
   it can be expected that `1.3` will contain some extra functions when
   compared to `1.2`. The next version level (the `z` part) is for
   bugfixes that do not add any functionality.
