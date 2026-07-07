# ILIAS Skin Development Guide — EUniWell

This document explains how this **private style repository for ILIAS** is set up, based on the public [ILIAS Delos repository](https://github.com/ILIAS-eLearning/delos), and how to maintain the EUniWell skin.

This repository is fully standalone and dedicated to the EUniWell learning instance, with its own template id, its own upstream tracking, and its own repository.

---

## Branch Overview

Each branch of this repository corresponds to one ILIAS major version:

| Branch | ILIAS version |
|---|---|
| `release_9-euniwell` | ILIAS 9 |

**You are currently on `release_9-euniwell` (ILIAS 9).**

---

## A. Code Versioning

We maintain a **private repository** that tracks the public Delos repo as an **upstream remote**.
Our changes (EUniWell design, skin settings) are applied in our own branches.

### Initial Setup (already done for this checkout)

```bash
# 1) Create/clone your private, empty repository
cd C:\Users\####\git
git init EUniWell-Skin
cd EUniWell-Skin

# 2) Add the public Delos repository as upstream (HTTPS, no auth required)
git remote add upstream https://github.com/ILIAS-eLearning/delos.git
git fetch upstream

# 3) Create a clean base branch (no local changes here!)
git switch -c release_9-euniwell --track upstream/release_9

# 4) Once you have an empty private remote repository, add it as origin and push
git remote add origin https://github.com/cce-uzk/EUniWell-Skin.git
git push -u origin release_9-euniwell
```

### Updating with changes from Delos

Use `git merge` (not `git rebase`) — merge commits make rebase error-prone once history diverges.

```bash
git fetch upstream
git switch release_9-euniwell
git merge upstream/release_9
```

**Resolving conflicts:**

Template files we intentionally removed from our skin may cause `modify/delete` conflicts if upstream updated them. Keep our deletion:

```bash
git rm <conflicted-file>
```

Template files added by upstream that we do not want to override should also be removed before committing:

```bash
git rm <unwanted-upstream-template>
```

After resolving all conflicts, complete the merge:

```bash
git commit
```

Then recompile the skin (see Section B) and push:

```bash
git push
```

---

## B. ILIAS Skins

### Setup

The skin is defined in `template.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template xmlns="http://www.w3.org" version="$Id$" id="euniwell" name="EUniWell">
    <style name="EUniWell Style"
           id="skin_euniwell"
           image_directory="images"
           font_directory="fonts"
           css_file="skin_euniwell" />
</template>
```

`skin_euniwell` is currently the **only** style. Should EUniWell need additional style variants in the future, add further `<style>` entries here, each with its own `skin_<id>/` directory, following the same pattern.

### SCSS Structure

* The **main skin** (`euniwell`) is defined in `euniwell.scss` at the repository root, based on `delos.scss`.
* The **style** `skin_euniwell` has its own directory `skin_euniwell/` with its corresponding SCSS entry point `skin_euniwell/skin_euniwell.scss`.
* `skin_euniwell.scss` imports its own settings (`skin_euniwell/010-settings/`) and configures `../euniwell` (the root skin) via `@use ... with (...)`.
* Style-specific assets (`images/`, `fonts/`, login/mail templates under `Services/`) live self-contained inside `skin_euniwell/`.

### Compilation

Compile SCSS to CSS using [Sass](https://sass-lang.com/).
All commands are run from the **repository root**. Compile the main skin first, then the style:

```bash
sass euniwell.scss euniwell.css --no-source-map
sass skin_euniwell/skin_euniwell.scss skin_euniwell/skin_euniwell.css --no-source-map
```

### Migrating to a new ILIAS major version

Each ILIAS major version gets its own branch, tracking the corresponding upstream branch:

```bash
git fetch upstream
git switch -c release_10-euniwell --track upstream/release_10
git push -u origin release_10-euniwell
```

**Note:** The Delos SCSS structure may change between major versions and require manual adaptation before the skin compiles correctly. As of ILIAS 10, the delos SCSS files moved into a `delos/` subdirectory. This means import paths in `euniwell.scss` and `skin_euniwell/skin_euniwell.scss` (as well as the relative paths inside `skin_euniwell/010-settings/`) must be reviewed and updated when setting up a new version branch.

Once adapted, the regular update workflow (fetch → merge → recompile → push) applies identically to the new branch.

---

# Deploying `euniwell` to ILIAS

## First deployment (clone once)

```bash
# 0) Optional: use SSH + a read-only deploy key for private repos

cd <ILIAS_ROOT>/Customizing/global/skin/

# 1) Clone into expected skin id folder name
git clone https://github.com/cce-uzk/EUniWell-Skin.git euniwell
cd euniwell

# 2) Check out the branch matching your ILIAS version (see Branch Overview above)
git checkout release_9-euniwell

# 3) (Optional) make the webserver own the files
#    adjust user:group as needed
chown -R www-data:www-data .
```

## Maintenance (pull latest changes)

```bash
cd <ILIAS_ROOT>/Customizing/global/skin/euniwell
git pull --ff-only
```

## Switching ILIAS versions later

```bash
cd <ILIAS_ROOT>/Customizing/global/skin/euniwell
git fetch --all
git checkout release_10-euniwell   # replace with the branch for your new ILIAS version
git pull --ff-only
```
