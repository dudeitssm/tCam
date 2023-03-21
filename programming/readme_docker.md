Firmware Flashing Using Docker
=====

#### Description

This guide is targeted towards a Linux audience, as there is currently no USB or serial pass through support on MacOS or Windows.

#### Instructions

1. Clone this repo.

```
git clone https://github.com/danjulio/tCam.git
```

2. Pull the espressif official Docker image.

Do _not_ use the `latest` (default) image, as it will fail to compile the firmware due to numerous changes made in versions >=5.0 of `idf.py`:

```
docker pull espressif/idf:release-v4.4
```

3. (Optional) Compile the firmware from source.

```
FIRMWARE_DIRECTORY="/home/$USER/tCam/tCam/firmware"

docker run --rm \
    --volume $FIRMWARE_DIRECTORY:/project:Z \
    --workdir /project \
    espressif/idf:release-v4.4 \
    idf.py build
```

If compiling was successful, you should see a message similar to the following at the end of the build process.

The output binaries will be placed in the `build` directory:

```
Project build complete. To flash, run this command:
/opt/esp/python_env/idf4.4_py3.8_env/bin/python ../opt/esp/idf/components/esptool_py/esptool/esptool.py -p (PORT) -b 460800 --before default_reset --after hard_reset --chip esp32  write_flash --flash_mode dio --flash_size detect --flash_freq 80m 0x1000 build/bootloader/bootloader.bin 0x8000 build/partition_table/partition-table.bin 0xd000 build/ota_data_initial.bin 0x10000 build/tCam.bin
or run 'idf.py -p (PORT) flash'
```

4. Pre-requisites before flashing the firmware.

Ensure your user is part of the `dialout` group (on Fedora), or you will be unable to flash as a non-root user:

```
id $USER | grep dialout
sudo usermod -a <your_username> -G dialout

# Log out of your session and log back in for it to take effect
```

5. Using the compiled binaries from above, or the pre-compiled binaries provided in the repo (shown below), flash the firmware to the t-cam.

With Docker:

```
FIRMWARE_DIRECTORY="/home/$USER/tCam/tCam/firmware"
TCAM_SERIAL_PORT="/dev/ttyUSB0"

docker run --rm \
    --device $TCAM_SERIAL_PORT:$TCAM_SERIAL_PORT \
    --volume $FIRMWARE_DIRECTORY:/project:Z \
    --workdir /project \
    espressif/idf:release-v4.4 \
    idf.py -p $TCAM_SERIAL_PORT flash
```

With Podman (and SELINUX enabled):

```
FIRMWARE_DIRECTORY="/home/$USER/tCam/tCam/firmware"
TCAM_SERIAL_PORT="/dev/ttyUSB0"

podman run --rm \
    --device $TCAM_SERIAL_PORT:$TCAM_SERIAL_PORT \
    --annotation run.oci.keep_original_groups=1 \
    --security-opt label=disable \
    --volume $FIRMWARE_DIRECTORY:/project:Z \
    --workdir /project \
    espressif/idf:release-v4.4 \
    idf.py -p $TCAM_SERIAL_PORT flash
```

If all goes well, your t-cam's screen will go white and the flashing process should begin.

Within about a minute it should finish, with a message similar to the following:

```
...
...
Wrote 8192 bytes (31 compressed) at 0x0000d000 in 0.1 seconds (effective 845.0 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
Executing action: flash
Running ninja in directory /project/build
Executing "ninja flash"...
Done
```

6. (Optional) Clean up the Docker image.

```
docker image rm espressif/idf:release-v4.4
```

7. Congratulations! Your tcam is now ready for action!

#### Helpful References
https://hub.docker.com/r/espressif/idf
https://github.com/containers/podman/issues/4477#issuecomment-584024773
https://github.com/containers/podman/issues/9706#issuecomment-799251672

