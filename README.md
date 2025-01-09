# ✨ MANS ✨ — Muffin's Awesome NAS Stack

## Changelog

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

## Intro

An Ansible role for setting up a Debian based NAS using mergerfs, SnapRaid & snapraid-btrfs, utilising caching with automatic cache 'moving' to a backing pool.
___

## Tasks

This role will do the following:

* Check for updates and autoupdate if accepted
* Manage the base OS
  * Apply TZ
  * Enables passwordless-sudo (optional)
  * Create/append users/groups
  * Required apps are installed
    * Optional apps are installed (user defined)
  * Install [fastfetch](https://github.com/fastfetch-cli/fastfetch) (optional)
  * Apply ssh keys (optional) (user defined)
  * Upgrade all packages and clean unused
* Install ZSH (optional)
  * Install [powerlevel10k](https://github.com/romkatv/powerlevel10k) (optional)
  * Runs fastfetch at login (optional)
* Install rclone (stefangweichinger.ansible_rclone) (optional)
* Install Docker (geerlingguy.docker) (optional)
* Install mergerfs
* Install SnapRaid
* Install btrfs
* Wipe and setup disks
  * Wipe all disks that do not have the correct fs
  * Setup `data_disks` with btrfs `data` subvolumes
  * Setup `parity_disks` & `cache_disks` as ext4
  * Configure mounts and fstab
* Configures mergerfs with `systemd` service files dependent on the config in `vars.yml`
* Configures SnapRaid/snapper
  * Snapper configs verified and failsafe config re0-create initiated if issued are detected
* Configures Samba (vladgh.samba.server)
* Configures [mergerfs-cache-mover](https://github.com/MonsterMuffin/mergerfs-cache-mover)
* Deploys & Configures [Scrutiny](https://github.com/AnalogJ/scrutiny)

## More Info

I have written in-depth blog posts about how I got here and detailing how this setup works.

* [Part 3: Designing & Deploying MANS — A Hybrid NAS Approach with SnapRAID, MergerFS, and OpenZFS ](https://blog.muffn.io/posts/part-3-mini-100tb-nas/) attempts to explain my design approach.

* [Part 4: MANS (Muffin's Awesome NAS Stack), An Overview And Guide](https://blog.muffn.io/posts/part-4-100tb-mini-nas/) goes through how this role works and fits together, as well as a much more concise explanation of all the variables.

I would ***highly recommend*** you read, at least, those two blog posts so you are familiar with how this works. I understand the yolo mentality, but please do not run this blindly.

<small>(If that link doesn't work it's not released yet, but it's coming soon!)</small>

## Prerequisites

* You must be using Debian. You can a check in the vars that stops you from using anything else but I will not support any fixes that are required to make this work on another distro.
* Understand how Snapraid works and its drawbacks.
* Understand what this role does and how it configures all the bits of software it uses.
* A device (ideally running MacOS/Linux) with Ansible.
* A machine that you will be using as a NAS with multiple drives (minimum 3).
  * Disk(s) for parity that is/are larger or as large as your largest data disk.

## Clone

Clone the repo somewhere on your machine.

```bash
https://github.com/monstermuffin/muffins-awesome-nas-stack.git
```
## Config

This role will take your settings from a global vars file and apply those to the setup. If/when you need anything changed, including adding/removing disks, you should modify the vars file and rerun the playbook.

Firstly, make a copy of the inventory file, example playbook and example vars file whilst in the root of the project.

```bash
cd muffins-awesome-nas-stack
cp inventories/inventory.example.yml inventories/inventory.yml
cp vars_example.yml vars.yml
cp mansplaybook_example.yml playbook.yml
```

Edit your new `inventory.yml` file to include the IP/hostname of your Debian server/machine, along with the user you want to run Ansible as that has root access.

It should look something like this:

```yml
---
mans_host:
  hosts:
    hht-fs01:
      ansible_host: hht-fs01.internal.muffn.io
      ansible_user: muffin
```

## Vars

Below is an explanation of the core variables needed to make the role function, as well as some things you may wish to change.

This does not cover all the variables just what you should change to your preference at a minimum, and the core variables needed to configure the services.
___

`passwordless_sudo: true` — Enables passwordless sudo. If set to `false` you must always specify your `become` pass on every run.

`configure_zsh: true` — Installs ohmyzsh and sets your default shell to ohmyzsh.

`configure_powerlevel10k: true` — Configures [powerlevel10k](https://github.com/romkatv/powerlevel10k)

`fastfetch_motd: true` — Sets a MOTD to be fastfetch. Useful IMO. Must have zsh enabled to work. Fastfetch will be installed if missing.

`install_rclone: true` — Installs rclone.

`install_docker: true` — Installs and configures docker. Setting this or `configure_scrutiny` to true will run docker.

`configure_scrutiny: true` — Installs and configures [Scrutiny](https://github.com/AnalogJ/scrutiny) in Docker for your disks.

`configure_hdidle: true` — Installs and configures [hdidle.](https://github.com/adelolmo/hd-idle)

`skip_os_check: false` — Skips the OS check for Debian. Unsupported.

`wipe_and_setup: true` — If disks need to be setup, this needs to be set to true else the playbook will immediately fail. You will still be required to accept a prompt before changes are made.

`extra_apps:` — List any extra applications from apt that you would like to install. Follow the example layout and uncomment.

___

`samba users // password` — Use vault to set a password, or just slap some plain text into here, I won't know.
___

`content_files` — If you are not planning on using a cache disk you must remove the third path and possibly replace it. Depending on how many parity disks you have, you may need more content files and these _must_ be on separate disks. You cannot place them on the data disks as it is not supported to have these in a subvolume.
___
`data_directories` — Top level directories that will be created on every `data_disks` and `parity_disks`.
___
You ***must*** have your disks formatted in the format that is pre-filled. You can of course add or remove any entries as necessary, but `/dev/disk/by-id/your-disk` must be how the vars are entered.

Any of the disks can be added/removed at any time, simply make your changes and rerun, that's the point of this.

To get your disks in the correct format copypasta the following into your terminal:

```bash
lsblk -do NAME,SIZE,MODEL | while read -r name size model; do
    echo -e "Disk: /dev/$name\nSize: $size\nModel: $model\nID Links:";
    ls -l /dev/disk/by-id/ | grep "/$name$" | grep -v "wwn-" | awk '{print "  /dev/disk/by-id/"$9}';
    echo "";
done
```

`data_disks` — Your data disks.

`parity_disks` — Your parity disk(s). As previous, these must be larger or as large as the largest data disk. Must be at least 1 disk here.

`cache_disks` — Any fast disk you want to send writes to, ideally this should be an NVME. This variable can be:

* 1 single disk in the form of `/dev/disk/by-id/your-disk`.
* Multiple disks in the form of `/dev/disk/by-id/your-disk`. When multiple disks are used, this ***is not adding redundancy***, just more cache.
* An existing path on your operating system. If you already have space on your OS drive for example, you could use something like `/opt/mergerfs-cache` or something. To ensure this doesn't fill up, adjust the `mergerfs-cache-mover` vars as needed (below.)
* A path ***and*** a disk in the format of `/dev/disk/by-id/your-disk`. I don't know why but it's supported.
___
Cache mover things:
https://github.com/MonsterMuffin/mergerfs-cache-mover
___
If you left `configure_scrutiny` to `true` then you can setup `omnibus` or `collector` mode here, if you don't know then leave the default, `omnibus`.

To get notifications about your disk health, enable one or more of the notification options and enter the relevant variables for the service.

## Install Requirements

To install the requirements, in the proect dir run the following:

```bash
pip install -r requirements.txt
ansible-galaxy install -r requirements.yml
```

## Deploying

To run the playbook, simply run:

```bash
ansible-playbook playbook.yml -Kk
```

If you have opted to install your ssh keys with this role, subsequent runs will not require `k`.

If you have opted to configure `passwordless_sudo`, `K` will not be required on subsequent runs.

The playbook should execute all the required actions to set up & configure MANS. Subsequent runs will of course be much faster.

You can target specific elements of the setup process with tags. For example:

```bash
# Only run mergerfs setup
ansible-playbook playbook.yml --tags mergerfs
# You can also use the shorthand version, `-t`
ansible-playbook playbook.yml -t mergerfs
```

You can use this in reverse, excluding any step with a given tag:

```bash
ansible-playbook playbook.yml --skip-tags mergerfs
```

You can see all available tags:

```bash
ansible-playbook playbook.yml --list-tags
```

You can list all tasks and their tags:

```bash
ansible-playbook playbook.yml --list-tasks
```

You can show all tasks that would be included with a given tag:

```bash
ansible-playbook playbook.yml --tags install_btrfs --list-tasks
```


## Usage

After a successful deployment, you will have the following (dependent on config):

### Mounts

* `/mnt/media` — This is the 'cached' share (if any cache device was specified). This is where writes will go and samba is configured to serve to/from.
* `/mnt/media-cold` — 'Non-cached' share. This is the pool of backing data disks.
* `/mnt/cache-pool` — If multiple cache devices were defined, this is the mount point for the pooled cache devices.

* `/mnt/data-disks/dataxx` — Mount points for `data_disks`.
* `/mnt/parity-disks/parityxx` — Mount points for `parity_disks`.
* `/mnt/cache-disks/cachexx` — Mount points for `cache_disks`.

### Logs

* `/var/log/snapraid-btrfs-runner.log` — Logs for `snapraid-btrfs-runner` runs.
* `/var/log/snapper.log` — Logs for `snapper`.
* `/var/log/cache-mover.log` — Logs for `mergerfs-cache-mover` runs.

### Commands
* `sudo python3 /var/snapraid-btrfs-runner/snapraid-btrfs-runner.py -c /var/snapraid-btrfs-runner/snapraid-btrfs-runner.conf` — Runs `snapraid-btrfs-runner` manually.

* `sudo python3 /opt/mergerfs-cache-mover/cache-mover.py --console-log` — Runs `mergerfs-cache-mover` manually.

* `sudo snapper list-configs` — Lists all valid `snapper` configs.

* `sudo snapraid-btrfs ls` - List snapshots.

* `sudo btrfs subvolume list /mnt/data-disks/data0x` - Show Btrfs subvolumes for given data disk.

### Ports

* `8080` - Scrutiny web-ui (omnibus).

## Issues & Requests

Please report any issues with full logs (`-vvv`). If you have any requests or improvements, please feel free to raise this/submit a PR.

### Snapper error "The config 'root' does not exist"

This is expected and unfortunately just a symptom of the way this is configured vs. how Snapper expects to be used.

The error presents itself like so:

```bash
$ sudo snapper list
The config 'root' does not exist. Likely snapper is not configured.
```

Most things can be done instead with `snapraid-btrfs`, for example:

```bash
$ sudo snapraid-btrfs list
data01 /mnt/data-disks/data01
# │ Type   │ Pre # │ Date                         │ User │ Cleanup │ Description         │ Userdata
──┼────────┼───────┼──────────────────────────────┼──────┼─────────┼─────────────────────┼──────────────────────
0 │ single │       │                              │ root │         │ current             │
2 │ single │       │ Thu 17 Oct 2024 03:52:31 BST │ root │         │ snapraid-btrfs sync │ snapraid-btrfs=synced

data02 /mnt/data-disks/data02
# │ Type   │ Pre # │ Date                         │ User │ Cleanup │ Description         │ Userdata
──┼────────┼───────┼──────────────────────────────┼──────┼─────────┼─────────────────────┼──────────────────────
0 │ single │       │                              │ root │         │ current             │
2 │ single │       │ Thu 17 Oct 2024 03:52:31 BST │ root │         │ snapraid-btrfs sync │ snapraid-btrfs=synced

data03 /mnt/data-disks/data03
# │ Type   │ Pre # │ Date                         │ User │ Cleanup │ Description         │ Userdata
──┼────────┼───────┼──────────────────────────────┼──────┼─────────┼─────────────────────┼──────────────────────
0 │ single │       │                              │ root │         │ current             │
2 │ single │       │ Thu 17 Oct 2024 03:52:32 BST │ root │         │ snapraid-btrfs sync │ snapraid-btrfs=synced
```

If necessary, you can run `snapper` commands via `snapraid-btrfs`, and this seems to work fine: `snapraid-btrfs snapper <command>`

___
<a href="https://www.buymeacoffee.com/muffn" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>
