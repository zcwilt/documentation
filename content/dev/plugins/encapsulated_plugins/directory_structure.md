---
title: Directory Structure
description: 
weight: 1 
layout: docs
tags:
    - v157+
---

## Directory Structure

Plugins written for the new architecture will reside in the `zc_plugins` directory.  Within their directory, plugin authors can add subdirectories that mimic the main Zen Cart directories. Note how each plugin starts with a versioned directory. This allows for automated upgrades and the possibility of downgrading as well.

Note that the `admin` sub-directory *must* be called `admin` here (not renamed) and the `catalog` sub-directory contains the storefront files.

e.g.

- zc_plugins
    - PluginName
        - v1.0.0
            - manifest.php
            - Installer
            - admin
                - includes
                    - auto_loaders
                    - classes
                      - observers
                    - extra_configures
                    - extra_datafiles
                    - functions
                        - extra_functions
                    - init_includes
                    - javascript
                    - languages
                        - english
                            - extra_definitions
            - catalog
                - includes
                  - auto_loaders
                  - classes
                    - ajax
                    - observers
                  - extra_configures
                  - extra_datafiles
                  - functions
                    - extra_functions
                  - init_includes
                  - languages
                    - english
                      - extra_definitions
                  - modules
                    - pages
                    - sideboxes
                  - templates
                    - default
                      - common (note that only *additional* common files will be loaded!)
                      - css
                      - images
                      - jscript
                      - templates
        - v1.0.1
            - Uses a similar structure to v1.0.0, with any files added/modified/removed since that version

***Notes***:

1. Zen Cart v1.5.7 or later is required. Plugins for prior versions must be installed directly into the main Zen Cart directory structure.

2. The `Installer` folder is optional; you can embed the installation logic in your plugin if you prefer.  See [Installer Classes](/dev/plugins/encapsulated_plugins/installer_classes/) and [Plugin SQL Installation](/dev/plugins/encapsulated_plugins/sql_installation/) for details on using the `Installer` folder. 

3. Naming an `/admin/javascript/` file the same name as your PHP file name will make it automatically load.  If you have additional .js files to load, give them names starting with the PHP file name followed by an underscore.  For example, if you create a file called `admin/rewards.php`, the autoloader will load `admin/includes/javascript/rewards.js`, as well as any JavaScript file whose name matches the pattern `admin/includes/javascript/rewards_*.js`, e.g. `admin/includes/javascript/rewards_sha-256.js`.
