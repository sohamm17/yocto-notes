# Adding a Desktop environment in Yocto Linux (3.0.1 Zeus)

The `macthbox` is a desktop-like environent running on top of XWindow framework. It has a very basic matchbox-desktop package.
It is included in the `sato` image target. We can include it in one of our own image target. Of course, the `sato` recipes should be available.

Let's say our image target name is `kokil`. And we have a `.bb` file at `meta-kokil/recipes-core/images/kokil-image.bb`. In this file, we can add the following
line:

```
IMAGE_INSTALL += "matchbox-desktop"
```
