# qgis-xpra

Ubuntu container with Xpra for running remote desktop applications in browser.

```
docker run -it -p 9876:9876 tswetnam/xpra-qgis:bionic
```

#### Run with NVIDIA GPU 

Image is built from NVIDIA CUDA GL Docker image and is compatible with NVIDIA GPUs - need to install additional software.

```
docker run -it --gpus all -p 9876:9876 -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY -e XAUTHORITY -e NVIDIA_DRIVER_CAPABILITIES=all tswetnam/xpra-qgis:cudagl-18.04
```
#### Run with Singularity

Build the container from a Docker image

```
singularity build qgis-xpra-bionic.sif docker://tswetnam/xpra-qgis:cudagl-18.04
```

Pull the container from [Singularity Library](https://cloud.sylabs.io/library)

```
singularity pull library://tyson-swetnam/default/qgis-xpra-bionic:latest
```

Run the Singualarity container on GPU with its own XPRA Desktop running on port `:9876`

```
singularity run --nv qgis-xpra-bionic.sif
```

Run the Singularity container with NVIDIA locally

```
singularity run --nv qgis-xpra-bionic.sif qgis
```


Run the Singularity container locally w/o GPU

```
singularity run qgis-xpra-bionic.sif qgis
```

# Setting up headless NVIDIA GPU Rendering

# Nvidia-headless

This is a small guide to have NVIDIA accelerated OpenGL support for nvidia-docker2 on a HEADLESS Ubuntu 18.04/20.04 server.

### Prereqs. for the system

```sudo apt-get install xinit xserver-xorg-legacy mesa-utils xterm```

### Stop gdm3 Window Manager

```sudo service gdm3 stop```

### Maybe also stop lightdm

```sudo service lightdm stop```

### Edit xinit permissions

```sudo sed -i -e "s/console/anybody/" /etc/X11/Xwrapper.config```

### Check PCI BusID

```nvidia-xconfig --query-gpu-info```

### Create xorg.conf with the correct BusID fromt he previus command

````sudo nvidia-xconfig --enable-all-gpus --use-display-device=none -o /etc/X11/xorg.conf --busid=PCI:X:Y:Z```

### Edit /etc/X11/xorg.conf and add the following at the top

```
Section "DRI"
        Mode 0666
EndSection
```

Check the correct BusID in the Device part

```
Section "Device"
        Identifier "Device0"
        Driver "nvidia"
        VendorName "NVIDIA Corporation"
        BusID "PCI:0:3:0"
EndSection
```

Remove the option in the display part:

```
Option "UseDisplayDevice" "none"
```

### Start an X Server

```export DISPLAY=:0```

```xinit &```

This should return you back to the CLI, if it holds the terminal it might be errored. 

### Test that it is working:

Check if Xorg is using nvidia driver under processes

```nvidia-smi```

### Check OpenGL:

```
glxinfo | grep OpenGL
```

If everything is ok there should be something with "NVIDIA OpenGL EGL"
