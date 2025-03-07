* [v0.93](https://github.com/monstermuffin/muffins-awesome-nas-stack/releases/tag/v0.93) - Samba Improvements

  * Added network buffer tuning optimizations for high-speed networks (10Gb+)
  * Configured SMB socket options for improved latency and throughput
  * Set optimal read/write raw parameters for better data transfer
  * Increased maximum transmission unit size to 65KB
  * Added session timeout management (15 minutes dead time)
  * Enabled path caching for faster directory operations
  * Implemented strict allocation with 4KB roundup for better storage efficiency
  * Optimized asynchronous I/O with 64KB read/write sizes
  * Enabled sendfile for faster file transfers
  * Added multi-channel support for SMB3 to utilize multiple network connections
  * Disabled strict locking for better compatibility
  * Enabled opportunistic locks (oplocks) and level2 oplocks for improved caching
  * Added fileid VFS object for consistent file identification

* [v0.92](https://github.com/monstermuffin/muffins-awesome-nas-stack/releases/tag/v0.92) - SnapRAID Improvement/Fix

  * This focuses on [16TB+ issues.](https://github.com/monstermuffin/muffins-awesome-nas-stack/issues/24) More about this can be found below in this README.
    * The 'old' single file parity can still be used as-is but a warning will show at the end of each run prompting for migration. Migration is automatic on the config side of things by setting a flag in vars, however a full sync is required afterwards. This is detailed below in this doc.
    * This change also significantly reworks the parity system in how it is defined in MANS. Documentation has been updated below. Using MANS v0.92 or later with the old parity vars format will throw an error. The syntax just needs to be changed, this is documented.
    * As above, this change allows the use of multiple parity files spread across smaller disks for people that want this, this is implemented into a 'level' system which I thought was the easiest/most straight forward way to do this, but I am probably wrong.
  * Added cleanup task for orphaned mount points when disks are shuffled around. This is unlikely to happen with prod usage but was getting annoying in my testing. 

* [v0.91](https://github.com/monstermuffin/muffins-awesome-nas-stack/releases/tag/v0.91) - Feature and Improvement Release

  * TL;DR - Improved/streamlined a bunch. Thanks @tigattack ❤️ 
  * Replaced cmd with POSIX mount (in most cases).
  * Implemented SnapRAID configuration and improvements.
  * Enhanced disk management.
  * Added tags to playbook.
  * Improved `ansible_managed` strings.
  * Improved output formatting and debugging information.
  * Streamlined role usage in playbooks.
  * Improved MergerFS configuration and service management.
  * Enhanced SnapRAID installation and configuration process.
  * Optimized Linux management tasks.
  * Simplified conditionals and eliminated silent failures in various roles.
  * Fixed HD-idle role.
  * Corrected default group name usage.
  * Resolved issues with Ansible check mode compatibility.
  * Fixed mount path templating in MergerFS configuration.
  * Addressed various minor bugs across different roles.
  * Updated documentation with new commands and error information.
  * Removed unused files and configurations.
  * Updated .gitignore and ansible-lint exclude rules.
  * Installed Ansible dependencies to .dependencies directory.
  * Implemented Ansible linting in CI pipeline.

* [v0.90](https://github.com/monstermuffin/muffins-awesome-nas-stack/releases/tag/v0.90)
  * Add 0.89.1 changes into release.
  * Fix Scrutiny mappings - Scrutiny is now running as a privileged container which is the easiest way to achieve that is required. Not doing so requires some annoying Ansible voodoo to get the mappings correct, especially with SSDs/NVMEs. If this is an issue for you, simply turn off auto-deployment of Scrutiny in vars and deploy it yourself as required. 

* v0.89.1
  * Modified mergerfs `minfreespace` from 1G to 10G.
  * Added `cache_pool_policy` var to allow users to select a different mergerfs policy if required for multiple cache devices.

* [v0.89 - Beta](https://github.com/monstermuffin/muffins-awesome-nas-stack/releases/tag/v0.89)
  * Released role in a state that is ready to be deployed/tested by others.