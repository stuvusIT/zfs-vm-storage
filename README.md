# zfs-storage

This role installs ZFS on Linux and configures filesystems and ZVOLs. `nfs-kernel-server` will be installed if a configured filesystem makes use of the `sharenfs` attribute.

## Requirements

An apt-based package manager with source version 16.04 or later, as ZFS isn't included in older versions.

## Role Variables

| Name                   | Default / Mandatory | Description                                                                                                     |
|:-----------------------|:-------------------:|:----------------------------------------------------------------------------------------------------------------|
| `zfs_parent_fs`        |                     | Existing parent ZFS filesystem for all filesystems and zvols that is optionally prepended to all configurations |
| `zfs_storage_defaults` |        `{}`         | Dict containing ZFS attributes that will be applied to all configured ZFS filesystems/zvols                     |
| `zfs_filesystems`      |        `[]`         | List of zfs_filesystems defined by a `name` and a dict of `attributes` (mandatory for each entry)               |
| `zvols`                |        `[]`         | List of zvols defined by a `name` and a dict of `attributes` (mandatory for each entry)                         |

Note: There are some ZFS attributes that can only be set at creation (see [man zfs](https://linux.die.net/man/8/zfs)). These are `utf8only`, `normalization` and `casesensitivity` for filesystems and `volsize` and `volblocksize` for ZVOLs. The respective task will fail if you try to change those.
Also, `volsize` is a mandatory attribute for ZVOLs.
If an attribute is not defined, the ZFS default will be configured, except for these four attributes:
- `acltype`=`posixacl`
- `compression`=`on`
- `relatime`=`on`
- `xattr`=`sa`

## Example Playbook

```yml
- hosts: zfsstorage
  roles:
    - role: zfs-storage
      zfs_parent_fs: tank
      zfs_storage_defaults:
        acltype: posixacl
        volsize: 50G
        quota: 50G
      zfs_filesystems:
        - name: testing
          attributes:
            quota: 200G
        - name: testing/wiki
          attributes:
            sharenfs: rw=@172.27.10.13
            compression: off
      zvols:
        - name: testing/dns01
          attributes:
            volsize: 100G
        - name: testing/ldap01
          attributes:
```

### Result

| Name                  | Type       | Attributes                                                                              |
|:----------------------|:-----------|:----------------------------------------------------------------------------------------|
| `tank/testing`        | filesystem | `acltype`=`posixacl`, `quota`=`200G`                                                    |
| `tank/testing/wiki`   | filesystem | `acltype`=`posixacl`, `sharenfs`=`rw=@172.27.10.13`, `compression`=`off`, `quota`=`50G` |
| `tank/testing/dns01`  | zvol       | `volsize`=`100G`                                                                        |
| `tank/testing/ldap01` | zvol       | `volsize`=`50G`                                                                         |

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).

## Author Information

 * [Michel Weitbrecht (SlothOfAnarchy)](https://github.com/SlothOfAnarchy) _michel.weitbrecht@stuvus.uni-stuttgart.de_
