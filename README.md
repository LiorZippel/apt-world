# Apt World

A Python-based utility for Debian systems that displays a list of packages explicitly installed by the user, similar to Gentoo's "selected set".

## Overview

In Debian package management, packages can be installed in two ways:
- Explicitly by the user (manually installed)
- Automatically as a dependency of another package

This tool parses the system package management files directly to identify which packages were explicitly installed by the user, creating functionality similar to Gentoo's "selected set" concept.

## Features

- Identifies packages explicitly installed by the user
- Excludes automatically installed dependencies
- Excludes essential system packages
- Provides verbose mode to show package descriptions
- Packaged as a Debian package for easy installation

## Installation

### From .deb package

```bash
sudo dpkg -i apt-world_1.0-1_all.deb
```

### Building from source

1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/apt-world.git
   cd apt-world
   ```

2. Build the Debian package:
   ```bash
   # Create package structure
   mkdir -p apt-world-build/DEBIAN
   mkdir -p apt-world-build/usr/bin
   
   # Create control file
   cat > apt-world-build/DEBIAN/control << 'EOF'
   Package: apt-world
   Version: 1.0-1
   Section: admin
   Priority: optional
   Architecture: all
   Depends: python3
   Maintainer: Lior Zippel <liorzippel@gmail.com>
   Description: Display packages explicitly installed by the user
    A tool to parse Debian package management files and display a list
    of packages explicitly installed by the user, similar to Gentoo's
    selected set.
   EOF
   
   # Copy the script
   cp package_inspector.py apt-world-build/usr/bin/apt-world
   chmod +x apt-world-build/usr/bin/apt-world
   
   # Build the package
   dpkg-deb --build apt-world-build
   mv apt-world-build.deb apt-world_1.0-1_all.deb
   ```

3. Install the package:
   ```bash
   sudo dpkg -i apt-world_1.0-1_all.deb
   ```

## Usage

Basic usage:
```bash
apt-world
```

Display package descriptions:
```bash
apt-world --verbose
```

## How It Works

The tool works by:

1. Parsing `/var/lib/dpkg/status` to get information about all installed packages
2. Parsing `/var/lib/apt/extended_states` to identify packages that were automatically installed
3. Filtering out essential system packages
4. Displaying the remaining packages that were explicitly installed by the user

## Technical Approach

This solution directly parses the package management files rather than using the output of apt or aptitude commands. This approach provides:

- Direct access to package metadata
- No dependency on other package management tools
- Consistent behavior across different Debian versions

The implementation is in Python 3 and follows these principles:
- Clean, well-commented code
- Proper error handling
- Efficient file parsing

## Requirements

- Debian 12 or compatible system
- Python 3

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Author

Lior Zippel <liorzippel@gmail.com>