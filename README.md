# archlinux-packaging

A VSCode devcontainer setup for archlinux packaging.

# Example steps for updating the Haxe package

## Check out the PKGBUILD file

https://wiki.archlinux.org/title/Arch_Build_System#Retrieve_PKGBUILD_source_using_SVN

1. `cd repos`

2. `svn checkout --depth=empty svn://svn.archlinux.org/community`

3. `cd community`

4. `svn update haxe`

## Update PKGBUILD

1. Update `pkgver`, reset `pkgrel` to `1`.

2. Update `source`

3. Update the other build deps/rules if needed.

## Build locally

1. Install dependencies by `makepkg -s`

2. `makepkg`
