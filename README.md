# qgis-xpra

Ubuntu container with Xpra for running remote desktop applications in browser.

Image is built from NVIDIA Docker image and is compatible with GPUs - need to install additional software.

```
docker run -it -p 9876:9876 tswetnam/qgis-xpra:bionic 
```

#### Run with NVIDIA GPU 

```
docker run -it --gpus all -p 9876:9876 -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY -e XAUTHORITY -e NVIDIA_DRIVER_CAPABILITIES=all tswetnam/qgis-xpra:bionic
```
#### Run with Singularity

Build the container from a Docker image

```
singularity build qgis-xpra-bionic.sif docker://tswetnam/qgis-xpra:bionic 
```

Pull the container from [Singularity Library](https://cloud.sylabs.io/library)

```
singularity pull library://tyson-swetnam/default/qgis-xpra-bionic:latest
```

Run the Singularity container locally

```
singularity run qgis-xpra-bionic.sif qgis
```

```
singularity run qgis-xpra-bionic.sif saga_gui
```
