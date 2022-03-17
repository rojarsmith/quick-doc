# Quick Msys2



```bash
# Update the package database and base packages.
pacman -Syu

# Update the rest of the base packages.
pacman -Su

# mingw-w64 GCC
pacman -S --needed base-devel mingw-w64-x86_64-toolchain
```

