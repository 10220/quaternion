# auto_version

Automatically create python package version info based on git commit date and
hash.  This simple little module is intended to be included as a git submodule
by other python packages, and used by `setup.py` on installation.

# Goals

The `calculate_version` function constructs a version number from the date.
This will be of the form

    2015.02.15.dev82e4326

where `2015.02.15` represents the date of the commit, and `82e4326` is the
shortened hash of the commit.  Additionally, if the code is not in a clean
state (changes have been made since the last commit), then `-dirty` will be
appended to the version.  This form requires the ability to run a few simple
`git` commands from `python`.  If that is not possible, the system will fall
back to using the current date (at the time of installation), preceded by
`0.0.0.dev`, so that version ordering will work safely.  In particular, note
that the calling of shell functions from python is a little delicate in python
2.6 and lower.

The `build_py_copy_version` class wraps the basic `build_py` class used in
the standard setup function, but adds a step at the end to create a file named
`_version.py` that gets copied into the installed module.

# Usage

To use both of these in the enclosing package, the enclosing `setup.py` file
could contain something like this:

```python
import distutils.core
from auto_version import calculate_version, build_py_copy_version

distutils.core.setup(...,
                     version=calculate_version(),
                     cmdclass={'build_py': build_py_copy_version},
                     ...,)
```

And in the package's `__init__.py` file, you could have something like this:

```python
from ._version import __version__
```

Then, in other code you would see the version normally:

```python
import enclosing_package_name
print(enclosing_package_name.__version__)
```

# Git subtrees

To add this repo as a subtree to another repo, make sure everything has been
committed in the super-repo, then do this:

```sh
git remote add -f auto_version https://github.com/moble/auto_version.git
git subtree add --prefix=auto_version auto_version --squash master
```

Now, any time the super-repo is checked out, the subtree will be there
automatically.

If you want to merge new changes in the subtree, run this:

```sh
git fetch auto_version
git subtree pull --prefix=auto_version auto_version master --squash
```

This will be a new commit in your super-repo.

## Pushing the subtree

This probably won't be as common, but if you make changes within the
`auto_version` directory in the super-repo, and you want to commit them back to
the original `auto_version` repo, you need to do something special; something
like this (though you may want to verify the details):

```sh
git subtree push --prefix=auto_version https://github.com/moble/auto_version.git master
```