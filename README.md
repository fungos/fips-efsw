fips-efsw
=========

[![Build Status](https://travis-ci.org/fungos/fips-efsw.svg?branch=travis-ci)](https://travis-ci.org/fungos/fips-efsw)

fipsified efsw (https://bitbucket.org/SpartanJ/efsw)

fips build system: https://github.com/floooh/fips

To use efsw you need just to add it to your `fips.yml`:

```cmake
imports:
     fips-efsw:
         git: https://github.com/fungos/fips-efsw.git
```

And them add a dependency to your project using `fips_deps(efsw)`.