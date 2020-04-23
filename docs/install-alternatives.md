---
layout: docs
title: "Alternative Install"
permalink: /docs/install-alternatives
---

# Alternative install methods

MicroK8s is spectacularly easy to install and use on Ubuntu or any Linux which
supports snaps. For other platforms or less common scenarios, see the relevant
notes below.

<a id="windows"> </a>
## Windows 10

<div class="p-notification--information">
  <p class="p-notification__response">
    Installing MicroK8s with Multipass requires Windows 10 Professional,
    Windows 10 Education or Windows 10 Enterprise, as it relies on the
    <em>Hyper-V</em> hypervisor. It
    will also require at least 4GB of available RAM and 40GB of storage.
  </p>
</div>

From 1.18, MicroK8s now has an official Windows installer, which is the
recommended way to install MicroK8s

1. **Enable Hyper-V**
    If you have not previously done so, you can enable the Hyper-V
    hypervisor feature in Windows 10.  This will avoid a reboot during
    the install process. To do so, please follow the instructions on
    the [Microsoft website][hyper-v].

1.  **Download the installer**

    The Windows installer is available on the MicroK8s GitHub page.
    [Download it here](https://github.com/ubuntu/microk8s/releases/download/installer-v1.0.0/microk8s-installer.exe)

1.  **Run the installer**

    Once the installer is downloaded, run it to begin installation.

    The installer will ask for administritive privilages.  This is in order to check the status of Hyper-V and enable it if necessary.

    ![](https://assets.ubuntu.com/v1/c7d0a5a7-winmk8s-03.png)

    The installer also requires the Ubuntu VM system, [Multipass][multipass], to be
    installed. This will be done automatically when you click 'Yes' here.

1.  **Setup MicroK8s**

    After MicroK8s has extracted, you'll be asked to start with default settings.  If you'd you'd like to tailor the settings,
    you can untick and define you own values for the VM later on.

    ![](https://assets.ubuntu.com/v1/4473ec87-installer-ask-setup.PNG)

    If you selected default setup the VM will be started immediately.

    ![](https://assets.ubuntu.com/v1/714cfc09-installer-auto-setup.png)

1.  **Open the command line:**

    Use PowerShell or the standard Windows 'cmd' to open a commandline.  If you didn't setup MicroK8s during the install, you'll
    need to open your commandline with Administrative Privileges.

    ![](https://assets.ubuntu.com/v1/a5fe14a5-winmk8s-04.png)

1. **Manually setup MicroK8s**

    If you setup MicroK8s in the installer, you can skip this step.

    The following arguments can be used with `microk8s install` to tailor your needs.

    - `--cpu`: Cores used by MicroK8s
    - `--mem`: RAM in GB used by MicroK8s
    - `--disk`: Maximum volume in GB of the dynamically expandable hard disk to be used
    - `--channel`: Kubernetes version to install

    ![](https://assets.ubuntu.com/v1/f89df5c5-installer-manual-setup.png)

    The setup will check that [Hyper-V][hyper-v] is enabled and enable it if not.  It will do the same for [Multipass][multipass].  After
    which, the MicroK8s VM will be started with the resources specified.

1.  **Check MicroK8s is running**

    Run the command:

    ```
    microk8s status --wait-ready
    ```

1.  **Explore what you can do!**

    Congrats! MicroK8s is now running on your Windows machine and is ready
    for you to explore and use Kubernetes. See the
    [main install](/docs/index#rejoin) page for your next steps!

### Uninstalling

To uninstall MicroK8s on Windows 10, please [follow this guide][uninstall].

<a id="macos"> </a>
## macOS

<div class="p-notification--information">
  <p class="p-notification__response">
    Installing MicroK8s with Multipass requires <em>macOS Yosemite</em>, version
    10.10.3 or later installed on hardware from 2010 onwards. It
    will also require at least 4GB of available RAM and 40GB of storage.
  </p>
</div>

The recommended way to install MicroK8s on MacOS is with Homebrew

1.  **Install Homebrew**

    Open a terminal and run the installer:

    ```
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
    ```

1.  **Install MicroK8s**

    Run the command:

    ```
    brew install ubuntu/microk8s/microk8s
    microk8s install
    ```

    [![asciicast](https://asciinema.org/a/IWhwnidik9xaC2YHfjBUIsLin.svg)](https://asciinema.org/a/IWhwnidik9xaC2YHfjBUIsLin)

1.  **Wait for MicroK8s to start**

    ```
    microk8s status --wait-ready
    ```

1.  **Congrats!**

    MicroK8s is now running! Continue to explore by following the
    _Getting Started_ instructions [here](/docs/index#rejoin)



### Uninstalling

To uninstall MicroK8s on Windows 10, please [follow this guide][uninstall].

<a id="multipass"> </a>
## multipass

With multipass installed, you can now create a VM to run MicroK8s. At least 4
Gigabytes of RAM and 40G of storage is recommended -- we can pass these
requirements when we launch the VM:

```bash
multipass launch --name microk8s-vm --mem 4G --disk 40G
```

We can now find the IP address which has been allocated. Running:

```bash
multipass list
```

... will return something like:

```no-highlight
Name                    State             IPv4             Release
microk8s-vm             RUNNING           10.72.145.216    Ubuntu 18.04 LTS
```

Take a note of this IP as services will become available there when accessed
from the host machine.

To work within the VM environment more easily, you can run a shell:

```bash
multipass shell microk8s-vm
```

Then install the  MicroK8s snap and configure the network:

```bash
sudo snap install microk8s --classic --channel=1.18/stable
sudo iptables -P FORWARD ACCEPT
```

From within the VM shell, you can now follow along the rest of the
[quick start instructions](index#status)

#### Useful multipass commands

-   Get a shell inside the VM:

    ```bash
    multipass shell microk8s-vm
    ```

-   Shutdown the VM:

    ```bash
    multipass stop microk8s-vm
    ```

-   Delete and cleanup the VM:

    ```bash
    multipass delete microk8s-vm
    multipass purge
    ```

<a id="arm"> </a>
## Raspberry Pi/ARM

Running MicroK8s on some ARM hardware may run into difficulties because cgroups
(required!) are not enabled by default. This can be remedied on the Rasberry Pi
by editing the boot parameters:

```bash
sudo vi /boot/firmware/nobtcmd.txt
```

<div class="p-notification--positive"><p markdown="1" class="p-notification__response">
<span class="p-notification__status">Note:</span> In older Raspberry Pi versions
the boot parameters are in `/boot/firmware/cmdline.txt`.</p></div>

And adding the following:

```no-highlight
cgroup_enable=memory cgroup_memory=1
```

## Systems using ZFS

There is currently an issue surrounding using MicroK8s on a ZFS filesystem due
to the way containerd is configured. If you have installed MicroK8s on ZFS
you can fix this:

1.  Stop microk8s:

    ```
    microk8s stop
    ```

1.  Remove old state of containerd:

    ```
    sudo rm -rf /var/snap/microk8s/common/var/lib/containerd
    ```

1.  Configure containerd to use ZFS:
    Edit  the file `/var/snap/microk8s/current/args/containerd-template.toml`
    replacing `snapshotter = "overlayfs"`  with `snapshotter = "zfs"`

1.  Create new zfs dataset for containerd to use:

    ```
    zfs create -o mountpoint=/var/snap/microk8s/common/var/lib/containerd/io.containerd.snapshotter.v1.zfs $POOL/containerd
    ```
1.  Restart microk8s:

    ```
    microk8s start
    ```

## Offline deployment

There are situations where it is necessary or desirable to run MicroK8s on a
machine not connected to the internet. This is possible, but there are a few
extra things to be aware of.

#### Downloading a snap

If the machine you are intending to install MicroK8s to has no connectivity at
all, it is possible to download the snap from a machine which does have
access to the internet.

```bash
snap download microk8s
```

this will retrieve *two* files to the local directory:

-   microk8s_xxx.snap: The snap package with a versioned suffix.
-   microk8s_xxx.assert: The assertion file (effectively a signature validating the package).

When the files are transferred to the offline machine, MicroK8s can then be
installed with the following commands:

```bash
sudo snap ack microk8s_993.assert
sudo snap install microk8s_993.snap
```

In an offline environment, the snap will not be able to contact the store for
any updates.

#### Simulating a network

In some environments, as well as being offline, there is no network capability
at all (e.g. no NIC hardware). In such cases the Kubernetes apiserver will not
be able to work. This can be solved by simulating hardware (e.g. in a VM) or
adding a virtual address.

For an example, see this [answer on askubuntu][askubuntu].

<!-- LINKS -->

[uninstall]: /docs/uninstall-alternatives
[ubuntu-app]: https://www.microsoft.com/en-us/p/ubuntu/9nblggh4msv6
[windows-post]: https://discourse.ubuntu.com/t/using-snapd-in-wsl2/12113
[multipass]: https://multipass.run/
[multipass-install]: https://multipass.run/#install
[askubuntu]: https://askubuntu.com/questions/993139/how-to-create-a-virtual-network-interface-in-ubuntu
[profile]: https://github.com/ubuntu/microk8s/tree/master/tests/lxc
[hyper-v]: https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v
<!-- FEEDBACK -->
