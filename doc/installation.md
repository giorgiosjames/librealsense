# Linux Ubuntu Installation

**Note:** Due to the USB 3.0 translation layer between native hardware and virtual machine, the *librealsense* team does not support installation in a VM. If you do choose to try it, we recommend using VMware Workstation Player, and not Oracle VirtualBox for proper emulation of the USB3 controller.
<br><br> Please ensure to work with the supported Kernel versions listed here and verify that the kernel is updated properly according to the instructions.

## Ubuntu Build Dependencies
The scripts and commands below invoke `wget, git, add-apt-repository` which may be blocked by router settings or a firewall. Infrequently, apt-get mirrors or repositories may also timeout. For *librealsense* users behind an enterprise firewall, configuring the system-wide Ubuntu proxy generally resolves most timeout issues.

## Prerequisites
**Important:** Running RealSense Depth Cameras on Linux requires patching and inserting modified kernel drivers. Some OEM/Vendors choose to lock the kernel for modifications. Unlocking this capability may requires to modify BIOS settings

  **Make Ubuntu Up-to-date:**  
  1. Update Ubuntu distribution, including getting the latest stable kernel
    * `sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade`  <br />  
  **Note:** On stock Ubuntu 14 LTS systems and Kernel prior to 4.4.0-04 the standard *apt-get upgrade* command is not sufficient to bring the distribution to the latest recommended baseline.  
  On those systems the following should be used:   
  `sudo apt-get install --install-recommends linux-generic-lts-xenial xserver-xorg-core-lts-xenial xserver-xorg-lts-xenial xserver-xorg-video-all-lts-xenial xserver-xorg-input-all-lts-xenial libwayland-egl1-mesa-lts-xenial `<br />    


  * Update OS Boot and reboot to enforce the correct kernel selection with <br />`sudo update-grub && sudo reboot`<br />

  * Interrupt the boot process at Grub2 Boot Menu -> "Advanced Options for Ubuntu" and select the kernel version installed in the previous step. Press and hold SHIFT if the Boot menu is not presented.
  * Complete the boot, login and verify that a supported kernel version (4.4.0-.., 4.8.0-.., 4.10.0-.. , 4.13.0-..or 4.15.0-.. as of May 2018) is in place with `uname -r`  


**Prepare Linux Backend and the Dev. Environment:**  
  1. Navigate to *librealsense* root directory to run the following scripts.<br />
     Unplug any connected Intel RealSense camera.<br />  

  2. Install the core packages required to build *librealsense* binaries and the affected kernel modules:  
    `sudo apt-get install git libssl-dev libusb-1.0-0-dev pkg-config libgtk-3-dev`  <br /><br />
    Distribution-specific packages:  <br />
     * Ubuntu 14 or when running of Ubuntu 16.04 live-disk:<br />
      `sudo apt-get install cmake3`<br />
      `./scripts/install_glfw3.sh`  <br />

     * Ubuntu 16:<br />
      `sudo apt-get install cmake3 libglfw3-dev`<br />

     * Ubuntu 18:<br />
      `sudo apt-get install cmake libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev`  <br /><br />

     **Note** on graphic sub-system utilization:<br />
     *glfw3*, *mesa* and *gtk* packages are required if you plan to build the SDK's OpenGl-enabled examples. The *librealsense* core library and a range of demos/tools are designed for headless environment deployment.

  3. Install Intel Realsense permission scripts located in librealsense source directory:<br />
    `sudo cp config/99-realsense-libusb.rules /etc/udev/rules.d/`  <br />
    `sudo udevadm control --reload-rules && udevadm trigger`
    <br />

  4. Build and apply patched kernel modules for: <br />
    * **Ubuntu 14/16/18 LTS**
      script will download, patch and build realsense-affected kernel modules (drivers).<br />
      Then it will attempt to insert the patched module instead of the active one. If failed
      the original uvc modules will be restored.  

      `./scripts/patch-realsense-ubuntu-lts.sh`<br />
    * **Intel® Joule™ with Ubuntu**
      Based on the custom kernel provided by Canonical Ltd.  

      `./scripts/patch-realsense-ubuntu-xenial-joule.sh`<br />
    * **Arch-based distributions**
      * You need to install the [base-devel](https://www.archlinux.org/groups/x86_64/base-devel/) package group.
      * You also need to install the matching linux-headers as well (i.e.: linux-lts-headers for the linux-lts kernel).<br />
        * Navigate to the scripts folder  `cd ./scripts/`<br />
        * Then run the following script to patch the uvc module: `./patch-arch.sh`<br /><br />

    * Check the patched modules installation by examining the generated log as well as inspecting the latest entries in kernel log:<br />
      `sudo dmesg | tail -n 50`<br />
    The log should indicate that a new uvcvideo driver has been registered.  
       Refer to [Troubleshooting](#Troubleshooting) in case of errors/warning reports.

  5. TM1-specific:
     * Tracking Module requires *hid_sensor_custom* kernel module to operate properly.
      Due to TM1's power-up sequence constrains, this driver is required to be loaded during boot for the HW to be properly initialized.

      In order to accomplish this add the driver's name *hid_sensor_custom* to `/etc/modules` file, eg:
      ```sh
      echo 'hid_sensor_custom' | sudo tee -a /etc/modules`
      ```

## Building librealsense2 SDK
  * On Ubuntu 14.04, update your build toolchain to *gcc-5*:
    * `sudo add-apt-repository ppa:ubuntu-toolchain-r/test`
    * `sudo apt-get update`
    * `sudo apt-get install gcc-5 g++-5`
    * `sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 60 --slave /usr/bin/g++ g++ /usr/bin/g++-5`
    * `sudo update-alternatives --set gcc "/usr/bin/gcc-5"`

    > You can check the gcc version by typing: `gcc -v`
    > If everything went fine you should see gcc 5.0.0.


  * Navigate to *librealsense* root directory and run `mkdir build && cd build`<br />
  * Run CMake:
    * `cmake ../` - The default build is set to produce the core shared object and unit-tests binaries in Debug mode. Use `-D CMAKE_BUILD_TYPE=release` to build with optimizations.<br />
    * `cmake ../ -DBUILD_EXAMPLES=true` - Builds *librealsense* along with the demos and tutorials<br />
    * `cmake ../ -DBUILD_EXAMPLES=true -DBUILD_GRAPHICAL_EXAMPLES=false` - For systems without OpenGL or X11 build only textual examples<br /><br />

  * Recompile and install *librealsense* binaries:<br />  
  `sudo make uninstall && make clean && make && sudo make install`<br />  
  The shared object will be installed in `/usr/local/lib`, header files in `/usr/local/include`.<br />
  The binary demos, tutorials and test files will be copied into `/usr/local/bin`<br />
  **Tip:** Use *`make -jX`* for parallel compilation, where *`X`* stands for the number of CPU cores available:<br />
  `sudo make uninstall && make clean && make **-j8** && sudo make install`<br />
  This enhancement may significantly improve the build time. The side-effect, however, is that it may cause a low-end platform to hang randomly.<br />
  **Note:** Linux build configuration is presently configured to use the V4L2 backend by default.<br />
  **Note:** If you encounter the following error during compilation `gcc: internal compiler error` it might indicate that you do not have enough memory or swap space on your machine. Try closing memory consuming applications, and if you are running inside a VM increase available RAM to at least 2 GB.

  2. Install IDE (Optional):
    We use QtCreator as an IDE for Linux development on Ubuntu
    * Follow the  [link](https://wiki.qt.io/Install_Qt_5_on_Ubuntu) for QtCreator5 installation


## <a name="Troubleshooting"></a>Troubleshooting Installation and Patch-related Issues

Error    |      Cause   | Correction Steps |
-------- | ------------ | ---------------- |
`git.launchpad... access timeout` | Behind Firewall | Configure Proxy Server |
`dmesg:... uvcvideo: module verification failed: signature and/or required key missing - tainting kernel` | A standard warning issued since Kernel 4.4-30+ | Notification only - does not affect module's functionality |
`sudo modprobe uvcvideo` produces `dmesg: uvc kernel module is not loaded` | The patched module kernel version is incompatible with the resident kernel | Verify the actual kernel version with `uname -r`.<br />Revert and proceed from [Make Ubuntu Up-to-date](#make-ubuntu-up-to-date) step |
Execution of `./scripts/patch-video-formats-ubuntu-xenial.sh`  fails with `fatal error: openssl/opensslv.h` | Missing Dependency | Install *openssl* package from [Video4Linux backend preparation](#video4linux-backend-preparation) step |
