# mender modules

Collection of [mender](https://mender.io/) modules to update a linux installation.

## Deployment

This is the main module that updates the deployment that will be run on the next reboot.

For deployment to work there are a few prerequisites:
  - The subvolid=5 of the disk containing rootfs __MUST__ be mounted on */base*
  - */base* directory __MUST__ contain *deployments* directory where deployments are being stored
  - Every deployment __MUST__ contains an executable file named */var/lib/mender/deployment/install*

To create a deployment write the rootfs into a btrfs subvolume (for example /mnt/osname_123dfa) and then use btrfs send:

```sh
export DEPLOYMENT_NAME="osname_123dfa"
btrfs send "/mnt/${DEPLOYMENT_NAME}" | xz -9e --memory=95% -T0 > "${DEPLOYMENT_NAME}.btrfs.xz"
```

__WARNING__: .btrfs extension as well as .xz are important as they will be used to recognise the file type
during the ArtifactInstall mender step.

## How To

Install required update modules placing them in the */usr/share/mender/modules/v3* directory of the rootfs.

Configure mender as described in the documentation and upload artifacts this way (taken from the [custom module documentation](https://docs.mender.io/artifact-creation/create-a-custom-update-module)):

```sh
export DEVICE_TYPE="raspberrypi4" # example
./mender-artifact write module-image \
    -t $DEVICE_TYPE \
    -o "${DEPLOYMENT_NAME}.btrfs.xz.mender" \
    -T deployment \
    -n "${DEPLOYMENT_NAME}.btrfs.xz" \
    -f "${DEPLOYMENT_NAME}.btrfs.xz"
```

The command line options are detailed below:

  - -t - The compatible device type of this Mender Artifact.
  - -o - The path where to place the output Mender Artifact. This should always have a .mender suffix.
  - -T - The payload type. It should be the same as the Update Module name and corresponds to the filename of the script which is present inside the /usr/share/mender/modules/v3 directory.
  - -n - The name of the Mender Artifact.
  - -f - The path to the file(s) to send to the device(s) in the update.

For more details, see mender-artifact write module-image --help
