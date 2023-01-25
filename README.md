# git-submod-lib 
![Current Version](https://img.shields.io/badge/Version-0.0.1a-brightgreen)

---
This repo holds common scripts and files to be included via 
[git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) across 
many projects.


## Make Python venv
```shell
make -f makefile/Makefile venv
```

## Build Image Locally
```shell
make -f makefile/Makefile build-image
```

## Releasing

### Minor releases
```shell
make -f makefile/Makefile promotion-alpha
```

Once the PR is approved and merged:
```shell
make -f makefile/Makefile github-release
```

Once the Release is published:
```shell
make -f makefile/Makefile github-image
```

Now cut a version release branch:
```shell
make -f makefile/Makefile github-branch
```

Now move `main` to the next `alpha` version to capture future development
```shell
make -f makefile/Makefile version-alpha
```

### Patch releases
Start with the version branch to be patched (ie `0.0.x`)
```shell
make -f makefile/Makefile promotion-patch
```

Once the PR is approved and merged:
```shell
make -f makefile/Makefile github-release-patch
```

Once the Patch Release is published:
```shell
make -f makefile/Makefile github-image
```


