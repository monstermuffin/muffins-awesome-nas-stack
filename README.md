# ✨ MANS ✨ — Muffin's Awesome NAS Stack

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
  * Disk(s) for parity that is/are larger or as large as your largest data disk, for most setups, see note below.

> [!NOTE]
>As of v0.92 multiple smaller parity disks can be used. This works by having multiple parity files on the smaller disks. So you may use 2x 8TB parity disks when using 16TB data disks, for example. Be aware that this means your parity/'backup' is now dependent on multiple disks being available. This **is not** the same as having 2 parity disks configured.

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

### Disk Config
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

`parity_disks` — Your parity disk(s). Must be at least 1 disk here.

> [!IMPORTANT]
> As of MANS v0.92 parity disk configuration has changed significantly due to https://github.com/monstermuffin/muffins-awesome-nas-stack/issues/24 and https://github.com/monstermuffin/muffins-awesome-nas-stack/issues/25. 
>
> The role will fail if you have an older version of the variables but a newer version of the role. Simply change the format as below.

There are two 'modes' to put a parity disk into, dedicated and split.

* Dedicated parity disks are single parity disk(s). This means the disk(s) are **larger or as large as your largest data disk.**
* Split parity disks are **smaller then your largest data disk but larger or as large when combined together.** This allows you to use multiple smaller parity disks when using larger data disks.

Example parity config:
```yml
parity_disks:
  # Level 1 split across two disks
  - device: /dev/disk/by-id/disk1
    mode: split
    level: 1
  - device: /dev/disk/by-id/disk2
    mode: split
    level: 1
  # Level 2 is a single dedicated disk
  - device: /dev/disk/by-id/disk3
    mode: dedicated
    level: 2
```

`Levels` are defined as an entire parity level. So you can only ever have one level per dedicated disk, as this is a dedicated parity `level`. The only time a `level` should be spread across disks is when using split mode. This is done because of the complexities of deploying such a config accurately. 

A split config can have as many disks as required to be larger or as large as your largest data disk, as long as they are in the same `level`.

Example A: You want one parity level and your parity disk is larger than any of your data disks:
```yml
parity_disks:
  - device: /dev/disk/by-id/disk1
    mode: dedicated
    level: 1
```
> [!TIP]
> In most cases this is the setup you will be using.

Example B: You want two parity levels and your parity disks are larger than any of your data disks:
```yml
parity_disks:
  - device: /dev/disk/by-id/disk1
    mode: dedicated
    level: 1
  - device: /dev/disk/by-id/disk2
    mode: dedicated
    level: 2
```
> [!TIP]
> In most cases this is the setup you will be using if you want multiple parity.

Example C: You want to split a parity across two smaller disks. Your largest data disk is 16TB. Both your parity disks are 8TB:
```yml
parity_disks:
  - device: /dev/disk/by-id/disk1
    mode: split
    level: 1
  - device: /dev/disk/by-id/disk2
    mode: split
    level: 1
```

Example D: You want to mix a dedicated parity disk as well as add a split parity across two other disks:
```yml
parity_disks:
  - device: /dev/disk/by-id/disk1
    mode: split
    level: 1
  - device: /dev/disk/by-id/disk2
    mode: split
    level: 1
  - device: /dev/disk/by-id/disk3
    mode: dedicated
    level: 2
```

Example E: You want two levels of parity, both using split disks:
```yml
parity_disks:
  - device: /dev/disk/by-id/disk1
    mode: split
    level: 1
  - device: /dev/disk/by-id/disk2
    mode: split
    level: 1
  - device: /dev/disk/by-id/disk3
    mode: split
    level: 2
  - device: /dev/disk/by-id/disk4
    mode: split
    level: 2
```

> [!IMPORTANT]  
> MANS will attempt to warn about incorrect parity var config at the start of the run, but this cannot be guaranteed.

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

## Split Parity Migration
MANS now supports split parity files to overcome ext4's 16TB file size limitation. This allows using data disks larger than 16TB with ext4-formatted parity disks by splitting parity data across multiple files.
Migration is only required if:

* You have an existing single-file setup **AND**
* You plan to use data disks larger than 16TB

> [!NOTE]  
> Whilst there is no real need to enable migration if the above in your situation is true, there may come a time when this option is deprecated completely. New deployments are split, so this can technically stay here forever, but I can't see the future. 
>
> It would be best to migrate when you can.

To migrate:

### Set MANS to migrate

In your vars.yml, set:

```yaml
split_parity_migrate: true
```
> [!NOTE]
> You will need to add this variable most likely if you have an existing MANS setup, please see `vars_example.yml` for any new vars you may be missing from updates.

### Run the MANS playbook

```bash
ansible-playbook playbook.yml
```

After playbook completion:

#### Delete existing parity file
```bash
sudo rm /mnt/parity-disks/parity01/snapraid.parity
```

> [!WARNING]  
> This may take significant time and look like it's hung, it's not. Do this in a tmux window and in another session you can see free/used space slowly changing with `df -h`. For my 12Tb parity file the delete action took  14m 40s.

> [!NOTE] 
> You may have more parity files to delete on other disks.

#### Rebuild parity in split files

> [!IMPORTANT] 
> If you have a lot of data this can take a significant amount of time. I highly recommend running the command below in a [tmux](https://www.howtogeek.com/671422/how-to-use-tmux-on-linux-and-why-its-better-than-screen/) window to run unattended. If you spawn sync in a normal SSH window and that connection is broken, it will break the sync.

```bash
sudo snapraid-btrfs sync --force-full
```
> [!NOTE]
>The sync process can take significant time depending on array size. Do not interrupt the process, as above.
>
>Each parity disk will have its own set of split files (e.g., snapraid-1.parity, snapraid-2.parity, snapraid-3.parity)
>
>Files are filled sequentially - when one file is full, SnapRAID moves to the next

> [!NOTE]
> As below, this took me about 26h to do 72Tb.

```bash
100% completed, 72481609 MB accessed in 26:10     0:00 ETA

     d1 20% | ************
     d2 22% | *************
     d3  9% | *****
     d4  5% | ***
     d5 10% | ******
     d6 10% | ******
     d7  0% |
 parity  7% | ****
   raid  3% | **
   hash 11% | ******
  sched  0% |
   misc  0% |
            |______________________________________________________________
                           wait time (total, less is better)
```

## Changelog
See the full changelog [here](./CHANGELOG.md).
___
<a href="https://www.buymeacoffee.com/muffn" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>
