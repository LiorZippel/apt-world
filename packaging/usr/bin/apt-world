#!/usr/bin/env python3

"""
package_inspector.py - A tool to display explicitly installed packages on Debian systems

This script parses package management files (/var/lib/dpkg/status and /var/lib/apt/extended_states)
to identify packages that were explicitly installed by the user, similar to Gentoo's "selected set".
"""

import os
import sys
import argparse


def parse_dpkg_status(file_path="/var/lib/dpkg/status"):
    """
    Parse the dpkg status file to extract package information.
    
    Args:
        file_path (str): Path to the dpkg status file
        
    Returns:
        list: List of dictionaries containing package information
    """
    packages = []
    current_package = {}
    last_field = None
    
    try:
        with open(file_path, 'r', encoding='utf-8') as f:
            for line in f:
                line = line.rstrip()
                
                # Empty line indicates end of a package entry
                if not line:
                    if current_package:
                        packages.append(current_package)
                        current_package = {}
                    continue
                
                # Continued field (starts with space)
                if line.startswith(" "):
                    if last_field and last_field in current_package:
                        current_package[last_field] += "\n" + line
                    continue
                
                # New field
                try:
                    field, value = line.split(":", 1)
                    field = field.strip()
                    current_package[field] = value.strip()
                    last_field = field
                except ValueError:
                    # Handle malformed lines
                    pass
            
            # Don't forget the last package
            if current_package:
                packages.append(current_package)
                
        return packages
    except FileNotFoundError:
        print(f"Error: Could not find {file_path}")
        return []
    except Exception as e:
        print(f"Error parsing {file_path}: {e}")
        return []


def parse_extended_states(file_path="/var/lib/apt/extended_states"):
    """
    Parse the APT extended states file to identify automatically installed packages.
    
    Args:
        file_path (str): Path to the extended states file
        
    Returns:
        set: Set of package names that were automatically installed
    """
    auto_installed = set()
    current_package = {}
    
    try:
        with open(file_path, 'r', encoding='utf-8') as f:
            for line in f:
                line = line.rstrip()
                
                if not line:
                    # Process the current package if it's marked as auto-installed
                    if current_package.get("Package") and current_package.get("Auto-Installed") == "1":
                        auto_installed.add(current_package.get("Package"))
                    current_package = {}
                    continue
                
                try:
                    field, value = line.split(":", 1)
                    current_package[field.strip()] = value.strip()
                except ValueError:
                    pass
            
            # Check the last package
            if current_package.get("Package") and current_package.get("Auto-Installed") == "1":
                auto_installed.add(current_package.get("Package"))
                
        return auto_installed
    except FileNotFoundError:
        print(f"Warning: Could not find {file_path}. Assuming no auto-installed packages.")
        return set()
    except Exception as e:
        print(f"Error parsing {file_path}: {e}")
        return set()


def get_essential_packages(packages):
    """
    Identify essential system packages that should not be considered user-selected.
    
    Args:
        packages (list): List of package dictionaries
        
    Returns:
        set: Set of package names that are marked as essential
    """
    essential = set()
    
    for package in packages:
        # Essential packages are marked with "Essential: yes"
        if package.get("Essential") == "yes":
            essential.add(package.get("Package"))
            
        # Also consider required packages part of the base system
        if package.get("Priority") in ["required", "important"]:
            essential.add(package.get("Package"))
            
    return essential


def get_user_installed_packages():
    """
    Identify packages explicitly installed by the user.
    
    Returns:
        list: List of package dictionaries that were explicitly installed
    """
    # Get all installed packages
    all_packages = parse_dpkg_status()
    
    # Get auto-installed packages
    auto_installed = parse_extended_states()
    
    # Get essential packages (part of the base system)
    essential_packages = get_essential_packages(all_packages)
    
    # Filter for installed packages
    installed_packages = [p for p in all_packages if 
                         p.get("Status", "").endswith("installed")]
    
    # Find user-installed packages (installed but not auto-installed or essential)
    user_installed = [p for p in installed_packages if 
                     p.get("Package") not in auto_installed and
                     p.get("Package") not in essential_packages]
    
    return user_installed


def main():
    """
    Main function to handle command line arguments and display results.
    """
    parser = argparse.ArgumentParser(
        description="Display packages explicitly installed by the user on Debian systems."
    )
    parser.add_argument(
        "-v", "--verbose", action="store_true",
        help="Show package descriptions in addition to names"
    )
    args = parser.parse_args()
    
    user_packages = get_user_installed_packages()
    
    if not user_packages:
        print("No user-installed packages found.")
        return
    
    print(f"Found {len(user_packages)} packages explicitly installed by user:")
    
    for package in sorted(user_packages, key=lambda p: p.get("Package", "")):
        if args.verbose:
            description = package.get('Description', '')
            first_line = description.split('\n')[0] if description else ''
            print(f"{package.get('Package'):<30} - {first_line}")
        else:
            print(package.get('Package'))


if __name__ == "__main__":
    main()