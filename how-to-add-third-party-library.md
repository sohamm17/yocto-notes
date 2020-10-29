# How to add a third-party library in Bitbake Yocto

We assume that the third-party library is based on `cmake`. Yocto as in-built recipes for `autotools`, `make`, `cmake`. 
`bitbake` generally configures, compiles and installs package in the target image in a fixed pattern by calling
the following functions one by one: `do_configure`, `do_compile`, `do_install`. You can override these functions if you want.

I want to install capnproto in my yocto image. The `meta-oe` layer already provides the capnproto tool. So, we should have a different package name
for our `capnproto` package. I name it `capnp`. I will install the package using `devtool` and by modifying the recipe in a `bb` file.

I got help from the following links:
1. https://stackoverflow.com/a/52062599/9605189
2. https://www.wolfssl.com/docs/yocto-openembedded-recipe-guide/

### How to add an online directory to bitbake (`tar` or `tar.gz`)

```
devtool add capnp https://capnproto.org/capnproto-c++-0.6.1.tar.gz
```

### How to edit the recipe

The new recipe should be added in the following location `build/workspace/recipes/capnproto/capnproto_0.6.1.bb`

My recipe looks like the following:

```
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://LICENSE.txt;md5=0a5b5b742baf10cc1c158579eba7fb1d"

SRC_URI = "https://capnproto.org/capnproto-c%2B%2B-${PV}.tar.gz"
SRC_URI[md5sum] = "d48846a72abe327b44e258bd46294d1e"
SRC_URI[sha256sum] = "8082040cd8c3b93c0e4fc72f2799990c72fdcf21c2b5ecdae6611482a14f1a04"

S = "${WORKDIR}/${BPN}roto-c++-${PV}/"

# NOTE: unable to map the following CMake srackage dependencies: CapnProto
inherit cmake

EXTRA_OECMAKE += " -DBUILD_TESTING=OFF"
EXTRA_OECMAKE += " -DCMAKE_CXXFLAGS='-fPIC'"
EXTRA_OECMAKE += " -DBUILD_SHARED_LIBS=ON"

FILES_${PN} = "${includedir}/capnp ${includedir}/kj ${bindir} ${libdir}"

FILES_${PN}-dev = ""
```

Notice that The package name variable `BPN` is intelligently `capnp`.
Also, notice that I have to assign no file to the `-dev` package because by default bitbake expects
that the `.so` file might end up in a `-dev` package. So you have to configure these with `FILES_${PN}-dev` variable.

I do not override the `do_configure` or `do_install` but you may need to. 
I tolied a lot on this because `autotools`, `cmake` may work weirdly in different projects.

### How to compile a recipe and check

```
devtool build capnp
```

### How to add a recipe to a layer

I use the following command to add this new recipe to the `meta-openpilot` layer:

```
devtool finish capnp meta-openpilot -f
```

### How to build against the layer

```
bitbake capnp
```
Now, the `capnp` package can be added in the `.bb` file of the target image in the target layer (meta-openpilot): `IMAGE_INSTALL += "capnp"`.

Finally the image can be built:

```
bitbake <image-name>
```

For a library the files could be checked if it has ended up in the following directory:
`tmp/work/<MACHINE_NAME>-poky-linux/<IMAGE_NAME>/1.0-r0/rootfs/`

The above is the target sytem's root filesystem.
