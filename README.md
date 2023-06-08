Ansible role ableton.ccache
===========================

This role installs the [ccache][ccache] software on the given Ansible host and configures
it for a given user.

Requirements
------------

Ansible >= 2.10, and a target host which is either running a Debian-flavor of Linux or
macOS. Other Unix flavors will probably work, provided that the software is installed from
source (see below).

Role Variables
--------------

The following variables influence how `ccache` is installed on the host:

- `ccache_config_options`: Dictionary of key/value options to write to the `ccache.conf`
  file. For more information, please refer to the ccache manual's
  ["Configuration options" section][ccache-config-options].
- `ccache_install_prefix`: Where to install ccache when building from sources or
  installing precompiled binaries (default: `/usr/local`)
- `ccache_install_type`: How to install ccache. Valid options include:
  - `build`: Build from sources
  - `github`: Use a precompiled binary from the ccache GitHub releases page (note: not all
    ccache versions support all platforms. Linux binaries started appearing at release
    4.6.1, and Mac binaries at release 4.8).
  - `package`: Use the system's package manager (default)
- `ccache_symlink_compilers`: List of compiler names to symlink to `ccache_symlink_path`
  (for example, `cc`, `g++`)
- `ccache_symlink_path`: When defined, create symlinks for common compiler names in this
  directory.
- `ccache_version`: Version of ccache to install when installing from sources.

See the [`defaults/main.yml`](defaults/main.yml) file for full documentation on required
and optional role variables.

Example Playbook
----------------

```yaml
---
- name: Install ccache on hosts
  hosts: "all"
  vars:
    ccache_config_options:
      cache_dir: "/tmp/caches/ccache"
      compression: "true"
    ccache_symlink_path: "/usr/local/lib/ccache"
    ccache_symlink_compilers:
      - cc
      - c++
      - clang
      - clang++
      - gcc
      - g++

  roles:
    - ableton.ccache
```

License
-------

MIT

Maintainers
-----------

This project is maintained by the following GitHub users:

- [@ala-ableton](https://github.com/ala-ableton)
- [@nre-ableton](https://github.com/nre-ableton)


[ccache]: https://ccache.dev
[ccache-config-options]: https://ccache.dev/manual/latest.html#_configuration_options
