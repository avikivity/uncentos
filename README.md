# uncentos - convert a running CentOS 7 image to an Ubuntu 20.04 LTS image

CentOS has become less attractive as a stable operating system.
There is no upgrade path from CentOS 7 to CentOS 8, and CentOS 8
EOL has been moved to 2021 and CentOS 8 was replaced by CentOS
Stream. While it's possible to switch to other RHEL rebuilds,
we have no way to predict their longevity. Ubuntu on the other hand
has a proven track record and a predictable release schedule, so
it is an interesting migration target, apart from the impossiblity
of upgrading in place.

For many use cases it is not necessary to upgrade in place. One
can start a new instance with the new OS and stop the old instance.
for big data applications such as ScyllaDB require a time consuming
data streaming process which is better avoided. It's also usually not
possible to replace the root volume, since shutting down a cloud
instance with local storage loses the local storage.

The solution is this script - it installs Ubuntu on a running CentOS
image while the instance is running. Once you reboot, you will be
running on Ubuntu 20.04 LTS, with all the data volumes intact.

## Procedure for AWS

 1. Take a snapshot of the root volumn (in case anything goes wrong)
 2. Extend the root volume by 5-10GB. This must be done while the instance is running (not during reboot), otherwise cloud-init will grab this space.
 3. Log in to the instance as root.
 4. Run the script with `curl -s https://raw.githubusercontent.com/avikivity/uncentos/master/uncentos | bash -ex`
 5. The script will create a new partition, install Ubuntu, and migrate users and groups.
 6. Perform additional migrations (install new packages, copy configuration files). The new image is available in `/newroot`. You can execute commands in the new image, for example `chroot /newroot apt-get install some-package`.
 7. Reboot.
 8. Verify the image is running, and that it booted into new new OS. If it failed, you can restore the root volume to its snapshot.

## Developing `uncentos`

 1. Run `./customize` to download a CentOS 7 image and prepare it for testing.
 2. Run `./run` to run the image using `qemu-kvm`.
 3. Run `./connect` to ssh into the image. From there, you can run `./mount; cd mount` to access the host filesystem and run the `./uncentos` script, and reboot.

