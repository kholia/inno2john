# Usage

Update: This was last updated and tested on 15-Jun-2023.

On Ubuntu, install the following packages,

```
sudo aptitude install libswitch-perl libdatetime-perl \
   libtext-glob-perl libdata-hexdumper-perl \
   libdata-printer-perl libcompress-raw-lzma-perl \
   libio-stringy-perl libdigest-crc-perl -y \
   libcrypt-rc4-perl
```

On Fedora, install the following packages,

```
sudo dnf install perl-Data-Printer perl-Switch \
   perl-DateTime perl-Compress-Raw-Lzma \
   perl-Digest-CRC
```

Get the "hash" to crack,

```
./inno2john.pl samples/Output/setup.exe > hash
```

Give ``hash`` to JtR.

It seems that, ``samples/hot-hash`` is the hottest thing on the internet today!

# uninno

## Introduction

uninno is a portable unpacking tool for Inno Setup (IS) installers.

It was originally conceived as a means to extract the data files from
[GOG.com (Good Old Games)](https://www.gog.com/) installer packages.
Back in 2008, when GOG was launched, only Microsoft Windows versions
of older PC video games were offered. For many of these games, there exist
emulators or engines rewrites that allow them to be played on other
operating systems or CPU architectures.

To allow GOG customers to play these games without requiring a Windows
system or Windows emulator, uninno was created. It is written in Perl
and should run on any platform that has a moderately recent Perl
interpreter available.

After initial success, uninno also became a bit of a research project
for studying computer language parsing and automatic code generation.

When new InnoSetup versions are released, support for them will
occasionally be added to uninno.

## Usage

To just use uninno as a Inno Setup extractor, run the uninno.pl utility.

    $ ./uninno.pl setup.exe

It will extract all files from the application part of the installer archvive
and put them into ./app

Various Perl modules are required. You can either install them from CPAN or
using your package manager (if available).

These are:
* Switch
* DateTime
* Digest::CRC
* IO::Uncompress::AnyInflate
* IO::Uncompress::Bunzip2
* Compress::Raw::Lzma
* Crypt::RC4

On Debian/Ubuntu Linux, use the following command to install all dependencies:

    $ sudo apt install libswitch-perl libdatetime-perl libdigest-crc-perl libcompress-raw-lzma-perl libtext-glob-perl libio-stringy-perl libcrypt-rc4-perl

## Code

uninno consists of a bunch of Perl packages that handle different stages of the
analysis and extractiong process. Dissection of the installer executable is
provided by Win::Exe, which has some submodules for each of the PE's parts.
Setup::Inno provides a frontend for parsing the setup.0 (metadata) and setup.1
(compressed files) parts of the Inno Setup installer.

To improve code reuse and facilitate handling of all the installer versions,
all version-specific code is represented by a class hierarchy, starting with
a base class containing mostly stubs, continuing with support for the first
open source Inno Setup release (2.0.8), and going up to the most recent version.
Each class in the hierarchy has override points that implement headers,
decompression and custom handling required for this version or its descendants.

Secondary to that is a set of data structure parsers for each exact Inno Setup
version. These can be generated from the Inno source code, as outlined below.

Version numbers are coded in 4 digits: x.y.z -> xyzz. The last version part has
two digits and is zero padded. Example: 2.0.8 -> 2008

Inno Setup has used many different compression algorithms in the course of its
history, with varying degrees of compatibility with standard software.
zlib, bzip2, LZMA and LZMA2 compression are supported by uninno, using
corresponding Perl modules.
LZMA compression uses a lot of memory currently and performance is poor, as
the API of Compress::Raw::Lzma is very low-level and hard to use in a
straight-forward way. A better implementation may be written in the future.

Starting with support for Inno Setup 5.5.0, a new approach is used to create the
various structure parsers. Using a custom Delphi grammar based on an old edition
of the Delphi Language Guide, Projects/Struct.pas from the Inno source code
is processed and dissected. Then, a Perl module is generated that can parse
binary data represented by the data structures in that file. The grammar can be
found in DelphiGrammar.pm, the code for the generator is in ParserGenerator.pm.

To generate a new parser, you need the Perl module Marpa::R2. You also need the
specific Inno Setup Struct.pas you would like to implement support for.
The makeissrc.pl utility can clone the official git repository for you and
optionally download and patch in older versions from their respective source
files. It will put the repository into ./innosetup by default.
The tool needs the tools wget, unzip and git to do its work.

makestruct.pl can then be used to access this repository and to generate a
new parser for a specific Inno Setup version.

For example:

    $ ./makestruct.pl --src ./innosetup --version 5.5.0u

Versions with a u at the end are Unicode versions, which means that all strings
are interpreted as UTF-16. Non-Unicode installers used to have their strings
stored in Windows code page 1252, but more recent installer may also specify
the used code page in the data structures. Support for this has not been
implemented yet.
It is recommended to always generate Unicode and non-Unicode versions
when creating a parser for a new version.
Output will go to Struct5500u.pm in this case, which needs to be put into
Setup/Inno/ to make Inno.pm find it.

## Links

* Inno Setup: http://www.jrsoftware.org/isinfo.php
* innounp: http://innounp.sourceforge.net/
* Good Old Games: http://www.gog.com
* Object Pascal Language Guide for Delphi: http://tinyurl.com/nowcn6c

## Copyright

uninno and all its components are

    Copyright © 2012-2019 by Gregor Riepl <onitake@gmail.com>
    All rights reserved.
    
    Redistribution and use in source and binary forms, with or without modification,
    are permitted provided that the following conditions are met:
    
        Redistributions of source code must retain the above copyright notice,
        this list of conditions and the following disclaimer.
        
        Redistributions in binary form must reproduce the above copyright notice,
        this list of conditions and the following disclaimer in the documentation
        and/or other materials provided with the distribution.
    
    THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
    ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
    WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
    DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR
    ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
    (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
    LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
    ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
    (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
    SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

The data extraction routines are generated based on the Inno Setup source code,
while the rest of the software was developed independently.
See http://www.jrsoftware.org/files/is/license.txt for the Inno Setup license.

No part of the project is affiliated with Inno Setup or its authors.
