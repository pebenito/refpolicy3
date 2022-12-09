# Changes from Reference Policy 2

Reference Policy 3 (refpolicy3) is a rewrite using the Cascade policy language.
This will mark a shift in the approach to achieving least privilege.  With the
new ability to delete rules as a feature of the language, the approach in
refpolicy3 is to a slightly coarser granularity than in v2, aimed at making
common case usage the main focus.  This includes a focus on creating higher
level abstractions while keeping low level functions for cases where tight
least privilege access is required.

## Terminolgy

* Interfaces are now called functions, per Cascade.
* Templates are replaced by inheriting virtuals.

## Module Organization Changes

* Some files contain multiple modules, for example selinuxutil.cas.
* The files, corecommands, and miscfiles modules, plus ld_so_t, lib_t, and textrel_shlib_t
  merged into the new files module.
* Port types are moved to the relevant modules.
* unlabeled_t is split from the kernel module into a new unlabeled module.
* init split into sysvinit and systemd modules.
* systemd domains were split into individual modules.
* authlogin was split into auth and login modules.
* logging was split into individual modules.
* sysnetwork was split into individual modules.
* unconfined moved to roles.
* dbus moved to the system directory.
* xdg moved to the apps directory.

## API changes

* Since Cascade supports rule deletion, functions in general allow or dontaudit more
  than interfaces in refpolicy2.
* Search directory member functions are no longer provided.  This access is now
  achieved by deleting the search permission from a list function or writing
  allow() rules.
* Domtrans and run functions are merged into a single domtrans function.  The
  new domtrans function is equivalent to the refpolicy2 run interfaces.
* Domains transitions using setexec() in refpolicy3 use "setexec" in the name
  instead of the confusing "spec" in refpolicy2.
* Create functions on files now includes write and append permissions.
* Create functions on all file-like objects now includes setattr.
* Create, rename, and delete functions on high-level file-like objects (common_file,
  common_pipe, etc.) also include the same permission on symlinks and directories.

## Type Changes

* Generally types are renamed with backwards compatibility, based on the
  behavior of Cascade's associated types.  For example, foo_runtime_t ->
  foo_t.runtime.
* Having directory names in the type name is only allowed by exception.
* Renamed/merged types, with backwards compatibility:
  * initrc_t -> init_script_t
  * qemu_device_t -> kvm_device_t
  * modules_object_t -> kernel_module_t
  * nfsd_ro_t -> public_content_t
  * nfsd_rw_t -> public_content_rw_t
  * port_t -> unreserved_port_t
  * readable_t -> public_content_t
  * smartcard_device_t -> cmx_device_t
  * systemd_activate_exec_t -> bin_t
  * systemd_analyze_exec_t -> bin_t
  * systemd_cgtop_exec_t -> bin_t
  * systemd_conf_t -> etc_t
  * systemd_detect_virt_exec_t -> bin_t
  * systemd_run_exec_t -> bin_t
  * systemd_stdio_bridge_exec_t -> bin_t
  * systemd_unit_command_t -> init_script_t
  * var_run_t -> runtime_t
* Split types:
  * nis_initrc_exec_t
    * yppasswdd_t.initrc_exec
    * ypserv_t.initrc_exec
    * ypxfr_t.initrc_exec
  * nis_unit_t
    * yppasswdd_t.unit
    * ypserv_t.unit
    * ypxfr_t.unit
  * security_t:
    * selinuxfs_t (selinuxfs only)
    * securityfs_t (securityfs only)
    * security object class remains security_t
* Removed types, without backwards compatibility:
  * cardmgr_dev_t
  * misc_device_t
  * test_file_t

## Rule Changes

* Character and block device permissions sets no longer include ioctl.
  The new ioctl function has an optional parameter to specify the individual
  ioctls, with the default being all ioctls.
* The manage permission set for character and block device permissions sets
  no longer include read, append, write, and ioctl permissions.
* distro_suse and distro_ubuntu blocks were dropped.

## Labeling Changes

* All textrel_shlib_t file contexts are dropped so they can be revalidated.
