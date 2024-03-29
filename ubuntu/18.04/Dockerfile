FROM harbor.cyverse.org/vice/xpra/desktop:18.04

USER root

# GDAL, GEOS, GRASS, QGIS, SAGA-GIS dependencies
RUN apt-get update \
    && apt-get install -y build-essential software-properties-common \
    && apt-get install -y --no-install-recommends --no-install-suggests \
        libblas-dev \
        libbz2-dev \
	libcairo2-dev \
        libcurl4-gnutls-dev \
	libexpat-dev \
        libfftw3-bin \        
        libfftw3-dev \
        libfreetype6-dev \
        libgdal-dev \
        libgeos-dev \
        libglu1-mesa-dev \
        libgsl0-dev \
        libjpeg-dev \
        libjsoncpp-dev \
        liblapack-dev \
        libncurses5-dev \
        libnetcdf-dev \
	libogdi3.2-dev \
        libopenblas-base \
        libopenblas-dev \
        libopenjp2-7 \
        libopenjp2-7-dev \
        libpdal-dev pdal \
        libpdal-plugin-python \
        libpng-dev \
        libpnglite-dev \
        libpq-dev \
        libproj-dev \
        libpython3-all-dev \
        libreadline-dev \
        libsqlite3-dev \
        libtiff-dev \
	libtiff5-dev \
        libwxgtk3.0-dev \
	libxmu-dev \	
        libzstd-dev \
        bison \
        bzip2 \  
        flex \
        g++ \
        gettext \
        gdal-bin \
        git \
	gtk2-engines-pixbuf \
	htop \
        make \
        ncurses-bin \
        netcdf-bin \
        proj-bin \
        proj-data \
        python3 \
        python3-dateutil \
        python3-dev \
        python3-magic \
        python3-numpy \
        python3-pil \
        python3-pip \
        python3-ply \
        python3-setuptools \
        python3-venv \
        sqlite3 \
        subversion \
        sudo \
	unixodbc-dev \
        wget \
	wx-common \
        xfce4-terminal \
        zlib1g-dev 

# Set gcc/g++ environmental variables for GRASS GIS compilation, without debug symbols
ENV MYCFLAGS "-O2 -std=gnu99 -m64"
ENV MYLDFLAGS "-s"
# CXX stuff:
ENV LD_LIBRARY_PATH "/usr/local/lib"
ENV LDFLAGS "$MYLDFLAGS"
ENV CFLAGS "$MYCFLAGS"
ENV CXXFLAGS "$MYCXXFLAGS"

# Set the locale
RUN apt-get clean && apt-get update && apt-get install -y locales
RUN echo LANG="en_US.UTF-8" > /etc/default/locale
RUN echo en_US.UTF-8 UTF-8 >> /etc/locale.gen && locale-gen

# Configure and install GRASS
ENV GRASS_PYTHON=/usr/bin/python3

# add GRASS source repository files to the image
RUN mkdir -p /code/grass \
    && wget -nv --no-check-certificate https://grass.osgeo.org/grass78/source/grass-7.8.3.tar.gz \
    && tar xzf grass-7.8.3.tar.gz -C /code/grass --strip-components=1 \
    && rm -rf grass-7.8.3.tar.gz \
    && cd /code/grass && ./configure \
       --enable-largefile \
       --with-cxx \
       --with-nls \
       --with-readline \
       --with-sqlite \
       --with-bzlib \
       --with-zstd \
       --with-cairo --with-cairo-ldflags=-lfontconfig \
       --with-freetype --with-freetype-includes="/usr/include/freetype2/" \
       --with-fftw \
       --with-netcdf \
       --with-pdal \
       --with-proj --with-proj-share=/usr/share/proj \
       --with-geos=/usr/bin/geos-config \
       --with-postgres --with-postgres-includes="/usr/include/postgresql" \
       --with-opengl-libs=/usr/include/GL \
       --with-openmp \
       --enable-64bit \
    && make \
    && make install \
    && ldconfig
   
# Unset environmental variables to avoid later compilation issues
ENV INTEL ""
ENV MYCFLAGS ""
ENV MYLDFLAGS ""
ENV MYCXXFLAGS ""
ENV LD_LIBRARY_PATH ""
ENV LDFLAGS ""
ENV CFLAGS ""
ENV CXXFLAGS ""

# set SHELL var to avoid /bin/sh fallback in interactive GRASS GIS sessions
ENV SHELL /bin/bash
ENV LC_ALL "en_US.UTF-8"
ENV GRASS_SKIP_MAPSET_OWNER_CHECK 1

# enable simple grass command regardless of version number
RUN ln -s /usr/local/bin/grass* /usr/local/bin/grass

# install external GRASS GIS session Python API
RUN pip3 install grass-session

# add GRASS GIS envs for python usage
ENV GISBASE "/usr/local/grass/"
ENV GRASSBIN "/usr/local/bin/grass"
ENV PYTHONPATH "${PYTHONPATH}:$GISBASE/etc/python/"
ENV LD_LIBRARY_PATH "$LD_LIBRARY_PATH:$GISBASE/lib"

# once everything is built, install a couple of GRASS extensions
#RUN grass -text -c epsg:3857 ${PWD}/mytmp_wgs84 -e && \
#    echo "g.extension -s extension=r.sun.mp ; g.extension -s extension=r.sun.hourly ; g.extension -s extension=r.sun.daily" | grass -text ${PWD}/mytmp_wgs84/PERMANENT

# Compile SAGA-GIS 2.3.1  -- ltr version that works inside QGIS

ENV TZ America/Phoenix

RUN mkdir /code/saga-gis \
    && wget -nv --no-check-certificate https://ayera.dl.sourceforge.net/project/saga-gis/SAGA%20-%202/SAGA%202.3.1/saga_2.3.1_gpl2.tar.gz \
    && tar xzf saga_2.3.1_gpl2.tar.gz -C /code/saga-gis --strip-components=1 \
    && rm saga_2.3.1_gpl2.tar.gz \
    && cd /code/saga-gis \
    && ./configure \
    && make \
    && make install

# Install QGIS Desktop binary

RUN wget -qO - https://qgis.org/downloads/qgis-2020.gpg.key | gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/qgis-archive.gpg --import && \
    chmod a+r /etc/apt/trusted.gpg.d/qgis-archive.gpg

RUN apt -y update && apt -f install && \
    echo "deb https://qgis.org/ubuntu bionic main" >> /etc/apt/sources.list && \
    echo "deb-src https://qgis.org/ubuntu bionic main" >> /etc/apt/sources.list 

# Install QGIS now
RUN apt -y update \
    && apt-get install -y \
        python-qgis-common \
        python-qgis \
        qgis-plugin-grass \
        qgis-providers \
        qgis 

#This avahi throws an error but is resolved with --bind-tcp=0.0.0.0:port in the xpra run
RUN apt-get install -y python3-pyinotify 

# Reduce the image size
RUN apt-get autoremove -y
RUN apt-get clean -y
RUN rm -rf /var/lib/apt/lists/*

# Install GRASS-GIS GUI 
RUN apt-get update && apt-get install -y grass

# grant access to /usr/share/qgis to user
RUN chown -R user:user /usr/share/qgis

# set XDG_RUNTIME_DIR for qgis
RUN mkdir /tmp/runtime-user && chown -R user:user /tmp/runtime-user
ENV XDG_RUNTIME_USER /tmp/runtime-user

# enable simple grass command regardless of version number
RUN ln -s /usr/local/bin/grass* /usr/local/bin/grass

# change to QGIS user profile to remove any root access
WORKDIR /home/user/

# set SHELL var to avoid /bin/sh fallback in interactive GRASS GIS sessions in docker
ENV SHELL /bin/bash

USER user

ENV PATH=$PATH:$GISBASE/bin:$GISBASE/scripts
ENV GISRC=/home/user/.grassrc7

# install external GRASS GIS session Python API
RUN pip3 install grass-session

# path to GRASS settings file
ENV GISRC=/tmp/grass7-${USER}-$GIS_LOCK/gisrc
# remove any leftover files/folder
RUN rm -fr /tmp/grass7-${USER}-$GIS_LOCK
RUN mkdir /tmp/grass7-${USER}-$GIS_LOCK
ENV TMPDIR="/tmp/grass7-${USER}-$GIS_LOCK"

# set GISDBASE, LOCATION_NAME, and/or MAPSET

RUN mkdir /home/user/grassdata
RUN echo "GISDBASE: /home/user/" >>$GISRC
RUN echo "LOCATION_NAME: grassdata" >>$GISRC
RUN echo "MAPSET: PERMANENT" >>$GISRC
# start in text mode
RUN echo "GRASS_GUI: text" >>$GISRC

# once everything is built, install a couple of GRASS extensions
RUN grass78 -text -c EPSG:3857 /home/user/grassdata/PERMANENT
#RUN echo "g.extension -s extension=r.sun.mp ; g.extension -s extension=r.sun.hourly ; g.extension -s extension=r.sun.daily" | grass78 -text /home/user/grassdata/PERMANENT

# Removing Conda from PATH to fix Python SIP error with QGIS
ENV PATH=/usr/local/nvidia/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/VirtualGL/bin/:/usr/local/go/bin:/usr/lib/grass78/bin:/usr/lib/grass78/scripts

# VICE permissions error fix
RUN sudo strip --remove-section=.note.ABI-tag /usr/lib/x86_64-linux-gnu/libQt5Core.so.5

VOLUME /tmp/.X11-unix

CMD xpra start \
        --bind-tcp=0.0.0.0:9876 \
        --start-child=xterm \
        --exit-with-children=no \
        --html=on \
        --daemon=no \
        --xvfb="/usr/bin/Xvfb +extension Composite -screen 0 1920x1080x24+32 -nolisten tcp -noreset" \
        --pulseaudio=no \
        --notifications=no \
        --bell=no \
        :100

