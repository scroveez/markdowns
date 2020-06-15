RHEL: 2 Volume Groups with the same name 
===================

[![Build Status][build-button]][build] [![Coverage Status][codecov-button]][codecov] [![Latest Version][mdversion-button]][md-pypi] [![Python Versions][pyversion-button]][md-pypi] [![BSD License][bsdlicense-button]][bsdlicense] [![Code of Conduct][codeofconduct-button]][Code of Conduct]

[build-button]: https://github.com/Python-Markdown/markdown/workflows/CI/badge.svg?event=push
[build]: https://github.com/Python-Markdown/markdown/actions?query=workflow%3ACI+event%3Apush
[codecov-button]: https://codecov.io/gh/Python-Markdown/markdown/branch/master/graph/badge.svg
[codecov]: https://codecov.io/gh/Python-Markdown/markdown
[mdversion-button]: https://img.shields.io/pypi/v/Markdown.svg
[md-pypi]: https://pypi.org/project/Markdown/
[pyversion-button]: https://img.shields.io/pypi/pyversions/Markdown.svg
[bsdlicense-button]: https://img.shields.io/badge/license-BSD-yellow.svg
[bsdlicense]: https://opensource.org/licenses/BSD-3-Clause
[codeofconduct-button]: https://img.shields.io/badge/code%20of%20conduct-contributor%20covenant-green.svg?style=flat-square
[Code of Conduct]: https://github.com/Python-Markdown/markdown/blob/master/CODE_OF_CONDUCT.md

1. OS has 2 VGs with same name vgroot. One of them is active and imported.
```bash
# vgdisplay | egrep -i "uuid|name"
  VG Name               vg_os
  VG UUID               YCz3rr-h7XO-rXJm-u91J-R52S-4PX7-69Ikql
  VG Name               vgroot    
  VG UUID               IL0a0r-A5cp-LkVZ-106X-bEQX-a9Me-7Nk19z
  VG Name               vgroot    
  VG UUID               QiOPy3-WhF8-RYns-44dY-FKKN-8orV-bEi3m3
```

```bash
# lvs
  LV  VG     Attr   LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_root  vg_os  -wi-ao---- 300.00g
  lv_swap  vg_os  -wi-ao---- 256.00g
  lvroot  vgroot -wi-ao---- 200.00g
```

2. Rename the deported volume group (vgroot) as vgroot_new.

```bash
# vgrename -v QiOPy3-WhF8-RYns-44dY-FKKN-8orV-bEi3m3 vgroot_new
Checking for existing volume group "QiOPy3-WhF8-RYns-44dY-FKKN-8orV-bEi3m3"
Checking for new volume group "vgroot_new"
Wiping cache of LVM-capable devices
Archiving volume group "vg_data" metadata (seqno 2).
Renaming "/dev/vgroot" to "/dev/vgroot_new"
Writing out updated volume group
Creating volume group backup "/etc/lvm/backup/vgroot_new" (seqno 3).
Volume group "vg_data" successfully renamed to "vgroot_new"
Wiping cache of LVM-capable devices
Wiping internal VG cache
```

3. Perform a rescan of VGs and LVs to verify that the new VGs and underlying LVs are identified with new names.

```bash
# vgscan
Reading all physical volumes. This may take a while...
Found volume group "vgroot" using metadata type lvm2
Found volume group "vgroot_new" using metadata type lvm2
Found volume group "vg_os" using metadata type lvm2
```
```bash
# lvscan
ACTIVE '/dev/data_vg/lv_data' [200.00 MB] inherit
inactive '/dev/data_vg_new/lv_data' [2.00 GB] inherit
ACTIVE            '/dev/vg_os/lv_swap' [256.00 GiB] inherit
ACTIVE            '/dev/vg_os/lv_root' [300.00 GiB] inherit
```

4. Activate the LV(s) under the new VG – vgroot_new.
```bash
# lvchange -ay /dev/vgrootg_new/lvroot
Verify that the volume(s) with new VG name is active.

# lvscan
ACTIVE '/dev/vgroot/lvroot' [200.00 MB] inherit
ACTIVE '/dev/vgroot_new/lvroot' [2.00 GB] inherit
ACTIVE            '/dev/vg_os/lv_swap' [256.00 GiB] inherit
ACTIVE            '/dev/vg_os/lv_root' [300.00 GiB] inherit
```

5. Mount the volume and check for data availability.
```bash
# mkdir /data_new
# mount /dev/vgroot_new/lvroot /root_new
# ls -lrt /root_new
```
