# Klipper with Sovol Zero changes

This repo attempts to clean up Sovol's Klipper changes made for the Sovol Zero 3D printer.

The [sovol-img](https://github.com/Gekkio/sovol-zero-klipper/tree/sovol-img) branch contains the git commits on the actual disk images on the printers. However, Sovol basically did a massive initial commit instead of applying changes on top of the upstream history. By comparing files and their history, I think Sovol used the upstream commit `faa89be816064b42bff1ba81478405490e49289c` as their base. The initial commit is basically the state of the repo at this particular upstream commit + a lot of Sovol changes in one large commit.

The [sovol-clean](https://github.com/Gekkio/sovol-zero-klipper/tree/sovol-clean) branch contains the changes cleaned up slightly and applied *on top of the upstream base*. This makes it much easier to study the differences for research purposes, since now all git tools (e.g. git blame) work properly.
