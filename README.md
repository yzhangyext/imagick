# Fork

This is a fork of the gographics/imagick package to work with Bazel. It includes
compiled object files for MacOS and Linux (CentOS/7 and Ubuntu 18.04) for
ImageMagick 6.8.8-7, by downloading it and its dependencies and compiling it all
from source.

These instructions assume a MacOS host.

Note that Linux systems that ultimately run the binaries linked to these object
files require glibc 2.14 or later. The stable versions of glibc for Ubuntu 18.04
and CentOS/7 are 2.27 and 2.17, respectively. The installed version of glibc on
a Linux system can be checked by running `ldd --version`.

## Rebuilding ImageMagick and dependencies

This is only necessary if you want to rebuild the native libraries, for example
to add new delegates (support for more image formats) or upgrading the library.

Linux (Ubuntu 18.04):

```
vagrant up build
rm -rf libs/include/*
rm -rf libs/linux/*
vagrant scp 'build:/home/vagrant/imagemagick-build/include/*.h' libs/include/
vagrant scp 'build:/home/vagrant/imagemagick-build/include/ImageMagick-6/*' libs/include/
vagrant scp 'build:/home/vagrant/imagemagick-build/lib/*.a' libs/linux/
sed -i '' -e 's|home/vagrant/imagemagick-build|usr/local|g' libs/include/magick/*.h
patch -p0 < pixel-wand.patch
```

MacOS (High Sierra):

```
bash download-build-imagemagick.sh
cp ~/imagemagick-build/lib/*.a libs/darwin
```

This is a good overview of the settings if you need to make changes to the
script:

http://imagemagick.sourceforge.net/http/www/install.html

## Testing that it works in development

Linux (Ubuntu 18.04)

```
vagrant up dev
vagrant ssh dev -c 'cd /vagrant && bazel test imagick:go_default_test'
```

If you make changes to the library after provisioning your vagrant VM,
you will need to rsync those changes for them to be visible.

```
vagrant rsync dev
```

You will also want to run `bazel clean`, as inappropriate caching of C
artifacts has been observed.

MacOS

```
bazel test imagick:go_default_test
```

## Testing that it works in production

Linux (CentOS/7)

```
vagrant up dev prod
vagrant ssh dev -c 'cd /vagrant && bazel build imagick:go_default_test'
vagrant scp dev:/vagrant/bazel-bin/imagick/go_default_test_/go_default_test .
chmod a+rwX go_default_test
vagrant scp go_default_test prod:/home/vagrant/
vagrant ssh prod -c ./go_default_test
```


# Go Imagick

Go Imagick is a Go bind to ImageMagick's MagickWand C API.

It was originally developed and tested with ImageMagick 6.8.5-4, however most official Unix or Linux distributions use older
versions (6.7.7, 6.8.0, etc) so some features in Go Imagick's go1 branch are being commented out and will see the light when
these ImageMagick distributions could easily be updated (from the devops PoV).

# Install

## Mac OS X

### MacPorts

```
sudo port install ImageMagick
```

## Ubuntu / Debian

```
sudo apt-get install libmagickwand-dev
```

## Common

Check if pkg-config is able to find the right ImageMagick include and libs:

```
pkg-config --cflags --libs MagickWand
```

Then go get it:

```
go get github.com/gographics/imagick/imagick
```

# API Doc

http://godoc.org/github.com/gographics/imagick/imagick

# Examples

The examples folder is full with usage examples ported from C ones found in here: http://members.shaw.ca/el.supremo/MagickWand/

# Quick and partial example

Since this is a CGO binding, Go GC does not manage memory allocated by the C API then is necessary to use Terminate() and Destroy() methods.

```
package main

import "github.com/gographics/imagick/imagick"

func main() {
	imagick.Initialize()
	defer imagick.Terminate()

	mw := imagick.NewMagickWand()
	defer mw.Destroy()

	...
}
```

# License

Copyright (c) 2013, Herbert G. Fischer
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

 * Redistributions of source code must retain the above copyright
   notice, this list of conditions and the following disclaimer.
 * Redistributions in binary form must reproduce the above copyright
   notice, this list of conditions and the following disclaimer in the
   documentation and/or other materials provided with the distribution.
 * Neither the name of the organization nor the
   names of its contributors may be used to endorse or promote products
   derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL HERBERT G. FISCHER BE LIABLE FOR ANY
DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
