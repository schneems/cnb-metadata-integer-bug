# CNB Metadata integer bug

## Expected

If you run this buildpack twice you should see this output on the second run:

```
Existing layer TOML contents:
[metadata]
  version = 1
```

## Actual

It currently instead outputs this:

```
Existing layer TOML contents:
[metadata]
  version = 1.0
```

Note that this is a float instead of an integer

## Reproduce

Clone the code:

```
git clone https://github.com/schneems/cnb-metadata-integer-bug
cd cnb-metadata-integer-bug
```

Build once to write metadta:

```
$ pack build sample --buildpack ./ --path ./
20: Pulling from heroku/buildpacks
Digest: sha256:c49ebf7c333d099ee765a1841acea6d5e5aa0a81a3375e96d7af41c4abc53344
Status: Image is up to date for heroku/buildpacks:20
20-cnb: Pulling from heroku/heroku
Digest: sha256:cb6e1902c1bd7c10e0e0d81d1a6c67a070e7820326b3cd22b11d0847b155ca5a
Status: Image is up to date for heroku/heroku:20-cnb
Previous image with name "sample" not found
===> DETECTING
sample-counter 0.1.0
===> RESTORING
===> BUILDING
===> EXPORTING
Adding layer 'sample-counter:test'
Adding 1/1 app layer(s)
Adding layer 'launcher'
Adding layer 'config'
Adding label 'io.buildpacks.lifecycle.metadata'
Adding label 'io.buildpacks.build.metadata'
Adding label 'io.buildpacks.project.metadata'
no default process type
Saving sample...
*** Images (12e0340bd3e0):
      sample
Adding cache layer 'sample-counter:test'
Successfully built image sample
```

Build it a gain to see what is in the prior metadat:


```
$ pack build sample --buildpack ./ --path ./
20: Pulling from heroku/buildpacks
Digest: sha256:c49ebf7c333d099ee765a1841acea6d5e5aa0a81a3375e96d7af41c4abc53344
Status: Image is up to date for heroku/buildpacks:20
20-cnb: Pulling from heroku/heroku
Digest: sha256:cb6e1902c1bd7c10e0e0d81d1a6c67a070e7820326b3cd22b11d0847b155ca5a
Status: Image is up to date for heroku/heroku:20-cnb
===> DETECTING
sample-counter 0.1.0
===> RESTORING
Restoring metadata for "sample-counter:test" from app image
Restoring data for "sample-counter:test" from cache
===> BUILDING
Existing layer TOML contents:
[metadata]
  version = 1.0
===> EXPORTING
Reusing layer 'sample-counter:test'
Reusing 1/1 app layer(s)
Reusing layer 'launcher'
Reusing layer 'config'
Adding label 'io.buildpacks.lifecycle.metadata'
Adding label 'io.buildpacks.build.metadata'
Adding label 'io.buildpacks.project.metadata'
no default process type
Saving sample...
*** Images (12e0340bd3e0):
      sample
Reusing cache layer 'sample-counter:test'
Successfully built image sample
```

Note the lines:

```
Existing layer TOML contents:
[metadata]
  version = 1.0
```

You can see in `bin/build` we are writing an integer, but reading a float.

## Problem with strongly typed languages

For dynamic languages there might be little differnce between `1` and `1.0` but in projects like Rust with [libcnb.rs](https://github.com/heroku/libcnb.rs/issues/473) it causes bigger problems as deserializing a float and an integer are two distinctly different things so the deserialization library believes the type signature has changed and it will throw out the old cache contents.

While this was originally discovered while working with buildpacks written in Rust with libcnb.rs we reproduced the issue using bash to get a minimum possible reproduction.
