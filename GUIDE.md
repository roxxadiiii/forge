# Forge — Manual Package Management Guide

This document describes the complete manual workflow for maintaining the Forge Arch Linux binary repository.

---

# Repository Layout

```
forge/
├── pkgs/
│   ├── package1/
│   ├── package2/
│   └── ...
│
├── repo/
│   └── x86_64/
│       ├── forge.db
│       ├── forge.db.tar.gz
│       ├── forge.files
│       ├── forge.files.tar.gz
│       └── *.pkg.tar.zst
│
├── scripts/
│
├── index.html
├── README.md
└── GUIDE.md
```

---

# Adding a New AUR Package

Move into the package directory.

```bash
cd ~/dev/forge/pkgs
```

Clone the PKGBUILD.

```bash
paru -G package-name
```

Example

```bash
paru -G python-cloup
```

---

## Remove Git Metadata

AUR repositories are Git repositories.

Forge should store packages as normal directories.

Always remove the nested Git repository.

```bash
rm -rf package-name/.git
rm -f package-name/.gitmodules
```

Example

```bash
rm -rf python-cloup/.git
```

---

## Review PKGBUILD

Always inspect the PKGBUILD before building.

```bash
less PKGBUILD
```

Verify

* package name
* dependencies
* source URLs
* install commands
* build commands

Never blindly build packages.

---

# Build Package

Enter the package directory.

```bash
cd package-name
```

Build.

```bash
makepkg -sfc
```

Meaning

* `-s`
  Install missing dependencies

* `-f`
  Force rebuild

* `-c`
  Clean build directory after build

---

# Verify Build

Package should appear.

```
package-name.pkg.tar.zst
```

---

# Publish Package

Copy package into repository.

```bash
cp *.pkg.tar.zst ../../repo/x86_64/
```

---

# Update Repository Database

Move into repository.

```bash
cd ../../repo/x86_64
```

Update database.

```bash
repo-add forge.db.tar.gz *.pkg.tar.zst
```

Refresh symlinks.

```bash
ln -sf forge.db.tar.gz forge.db
ln -sf forge.files.tar.gz forge.files
```

---

# Clean Build Artifacts

Return to repository root.

```bash
cd ~/dev/forge
```

Run cleanup.

```bash
./scripts/clean
```

This removes

* src/
* pkg/
* downloaded archives
* makepkg metadata
* temporary files

---

# Verify Git Status

```bash
git status
```

Review every file before committing.

---

# Commit

```bash
git add .
git commit -m "Add package-name"
```

---

# Push

```bash
git push
```

GitHub Actions automatically publishes the repository.

---

# Verify Deployment

Open

```
https://<username>.github.io/forge/
```

Repository database

```
https://<username>.github.io/forge/repo/x86_64/forge.db
```

Package

```
https://<username>.github.io/forge/repo/x86_64/package-name.pkg.tar.zst
```

---

# Install From Forge

Refresh package databases.

```bash
sudo pacman -Syy
```

Install.

```bash
sudo pacman -S package-name
```

---

# Updating an Existing Package

Go into the package directory.

```bash
cd pkgs/package-name
```

Restore the upstream AUR repository if needed.

```bash
git init
git remote add origin https://aur.archlinux.org/package-name.git
git fetch origin
git reset --hard origin/master
```

After updating, remove the nested repository again.

```bash
rm -rf .git
```

Rebuild.

```bash
makepkg -sfc
```

Publish.

```bash
cp *.pkg.tar.zst ../../repo/x86_64
```

Update repository.

```bash
cd ../../repo/x86_64
repo-add forge.db.tar.gz *.pkg.tar.zst
```

Clean.

```bash
cd ~/dev/forge
./scripts/clean
```

Commit.

```bash
git add .
git commit -m "Update package-name"
git push
```

---

# Never Commit

Never commit

```
src/
pkg/
*.tar.gz
*.zip
.BUILDINFO
.MTREE
.PKGINFO
*.pkg.tar.zst
```

inside `pkgs/`.

Only publish binary packages inside

```
repo/x86_64/
```

---

# Golden Rule

```
Clone
↓
Inspect PKGBUILD
↓
Build
↓
Test
↓
Copy package
↓
repo-add
↓
Clean
↓
Commit
↓
Push
↓
Install using pacman
```

This is the complete manual Forge workflow before automation.

