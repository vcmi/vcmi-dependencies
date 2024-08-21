# VCMI Dependencies

This repository contains prebuilt Conan dependencies for vcmi. These dependencies are primary used by Github CI and can be used to avoid building dependencies locally by developers.

Current flow to update dependencies:
- push new version of conanfile and/or CI settings to vcmi/vcmi repository, to branch 'update_prebuilts'
- run CI in this repository to generate new packages
- create new release from generated artifacts
- open PR in vcmi/vcmi with changes to conanfile, CI, new URL to dependencies package and whatever changes need to be done to source code to use newer package (if any)

# TODO List

- Find a way to set up an artifactory on our server or find somebody willing to host it for us and deprecate this repository. Potential options:
  - Artifactory Community Edition (requires more high-spec server than our current one)
  - https://docs.conan.io/2/reference/conan_server.html (not recommended by Conan)
  - https://jfrog.com/

- Switch to conan 2. Incomplete (and potentially outdated) PR can be found here: https://github.com/vcmi/vcmi/pull/1603 Will also require changing how we create final package - instead of archiving `~/.conan/data` we'll need to use `conan cache` command

- Use Conan for msvc builds. Currently blocked by several issues, namely:
  - Conan 1 does not works with latest Visual Studio 2022. We need to either use msvc 2019 or upgrade to conan 2.
  - ffmpeg fails to find its dependencies when building with conan 1 + msvc 2019. Might be fixed in conan 2.
  - Qt fails to build due to broken string escaping in a path (conan 1 + msvc 2019)

- Switch Android CI to use Linux runners instead of macOS runners (both for prebuilts and for vcmi itself)

- Upgrade ubuntu runner to ubuntu-24.04 and rebuild packages using newer mingw

- Rebuild boost and disable boost_url which we don't use

- Rebuild SDL (including SDL_mixer and SDL_image). 
  - Consider updating packages.
  - Enable support for opus and flac.
  - Remove unnecessary image formats such as gif and pcx
  - Ensure that vcmi can load ogg/opus and flac as 'sounds' and not only as music

- Rebuild ffmpeg with libdav1d and av1 support enabled. Needs investigation as to why dav1d fails to build on mingw and on android.

- Rebuild all binaries in prebuilts package to ensure that everything is configured correctly and to replace any locally-built binaries with binaries from CI

- Rebuild entire package from scratch using latest recipes from conan, to test current version of recipes

- Run CI with full package rebuild on schedule (weekly? monthly?) to detect any regressions or breaking changes in CI or in used recipes

- Automatically generate Github release with updated packages as part of CI. Should probably be done only for changes in main branch and/or for manually triggered workflows

# Proposed better flow for updating dependencies

- Move conanfile and conan profiles (or even entire CI directory) to this repository
- Add this repository as a submodule to vcmi/vcmi repository

With this approach we will be able to use following flow for new dependencies:
- change conan/CI settings in this repository as needed
- run CI in this repository to generate new packages and new release
- update URL to dependencies in CI/update_conan_dependencies.sh to point to new release
- open PR in vcmi/vcmi repository that bumps submodule to newest revision of this repository and whatever changes need to be done to source code to use newer package (if any)

After merging PR in vcmi/vcmi repository vcmi will use new dependencies.

If vcmi/vcmi PR is discarded for one reason or another, changes in this repository will have to be discarded as well