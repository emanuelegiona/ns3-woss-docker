# syntax=docker/dockerfile:1

# MIT License

# Copyright (c) 2025 Emanuele Giona <giona.emanuele@gmail.com> (SENSES Lab, 
# Sapienza University of Rome)

# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:

# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

FROM egiona/ns3-base:u18.04-n3.35-boosted

LABEL maintainer="giona.emanuele@gmail.com"
LABEL version="0.1"
LABEL description="ns3-base:u18.04-n3.35-boosted and WOSS 1.12.6"

ARG WOSS_VER="1.12.6"
ARG TMP_DOWNLOAD_DIR="/home/downloads"
ARG WOSS_LIB_BASE="/home/woss"

# ===== Evinronment variables available in containers too =====
# ns3-base provides the following environment variables already:
# - NS3_DEBUG_DIR: installation directory for ns-3 in 'debug' profile
# - NS3_OPTIMIZED_DIR: installation directory for ns-3 in 'optimized' profile
# - NS3_PY_ENV: ns-3 dedicated Python 3 environment
# - NEW_BOOST_INCLUDES: path to parent directory of upgraded Boost C++ libraries (headers)
# - NEW_BOOST_LIBS: path to parent directory of upgraded Boost C++ libraries (library objects)

# WOSS-related variables
ENV WOSS_LIB_SRC="$WOSS_LIB_BASE/WOSS-$WOSS_VER"
ENV WOSS_LIB_DIR="/home/woss_lib"
ENV WOSS_REQS_DIR="/home/woss_reqs"

# ===== Basic OS updates and ns-3 requirements =====
ARG DEBIAN_FRONTEND=noninteractive
RUN apt -y update && apt -y upgrade && \
# ===== Downloading: WOSS requirements and library =====
 cd /home && mkdir "$TMP_DOWNLOAD_DIR" && cd "$TMP_DOWNLOAD_DIR" && \
 # WOSS requirements
 mkdir "$WOSS_LIB_BASE" && mkdir "$WOSS_REQS_DIR" && cd "$TMP_DOWNLOAD_DIR" && \
 wget http://oalib.hlsresearch.com/AcousticsToolbox/at_2023_5_18.zip && \ 
 unzip at_2023_5_18.zip && rm at/Makefile && mv at "$WOSS_REQS_DIR/" && \
 wget https://github.com/HDFGroup/hdf5/archive/refs/tags/hdf5-1_14_2.tar.gz && \
 tar -xf hdf5-1_14_2.tar.gz && \
 wget https://downloads.unidata.ucar.edu/netcdf-c/4.9.2/netcdf-c-4.9.2.tar.gz && \
 tar -xf netcdf-c-4.9.2.tar.gz && \
 wget https://downloads.unidata.ucar.edu/netcdf-cxx/4.3.1/netcdf-cxx4-4.3.1.tar.gz && \
 tar -xf netcdf-cxx4-4.3.1.tar.gz && \
 # WOSS library source and ns-3 module
 wget "https://github.com/signetlabdei/WOSS/archive/refs/tags/$WOSS_VER.tar.gz" && \
 tar -xf "$WOSS_VER.tar.gz" && mv "WOSS-$WOSS_VER" "$WOSS_LIB_BASE" && \
 wget "https://github.com/MetalKnight/woss-ns3/archive/refs/tags/v$WOSS_VER.tar.gz" && \
 tar -xf "v$WOSS_VER.tar.gz" && mv "woss-ns3-$WOSS_VER" "$WOSS_LIB_BASE"

# ===== Installing: WOSS requirements and WOSS library =====
COPY at/Makefile "$WOSS_REQS_DIR/at/"
RUN cd "$WOSS_REQS_DIR/at" && \
 make clean && make && \
 echo "export PATH=$PATH:$WOSS_REQS_DIR/at/Bellhop/" >> ~/.bashrc && \
 # HDF5
 cd "$TMP_DOWNLOAD_DIR" && cd hdf5-hdf5-1_14_2 && \
 ./configure --enable-shared --prefix="$WOSS_REQS_DIR" && \
 make && make install && \
 # NetCDF-C
 cd ../netcdf-c-4.9.2 && \
 ./configure --prefix="$WOSS_REQS_DIR" --enable-netcdf-4 --enable-shared --disable-dap --disable-byterange CPPFLAGS="$CPPFLAGS -I$WOSS_REQS_DIR/include" LDFLAGS="$LDFLAGS -L$WOSS_REQS_DIR/lib" && \
 make && make install && \
 # NetCDF-C++
 cd ../netcdf-cxx4-4.3.1 && \
 ./configure --prefix="$WOSS_REQS_DIR" --enable-shared CPPFLAGS="$CPPFLAGS -I$WOSS_REQS_DIR/include" LDFLAGS="$LDFLAGS -L$WOSS_REQS_DIR/lib" && \
 make && make install && \
 # WOSS library
 mkdir "$WOSS_LIB_DIR" && cd "$WOSS_LIB_SRC" && \
 ./autogen.sh && ./configure --with-netcdf4="$WOSS_REQS_DIR" --with-pthread --prefix="$WOSS_LIB_DIR" && \
 make && make install

# ===== Building: ns-3 with WOSS; Cleanup: temporary downloads directory =====
ENV WOSS_LIB_DIR="/home/woss_lib/lib"
ENV CXX_CONFIG="-Wall -Werror -Wno-unused-variable"

COPY ns3-build/* ns3-utils/* /home/
RUN cd /home && \
 # Build: debug
 cp -R "$WOSS_LIB_BASE/woss-ns3-$WOSS_VER/" "$NS3_DEBUG_DIR/src/" && \
 mv "$NS3_DEBUG_DIR/src/woss-ns3-$WOSS_VER" "$NS3_DEBUG_DIR/src/woss-ns3" && \
 chmod +x build-debug.sh && ./build-debug.sh && \
 # Build: optimized
 cp -R "$WOSS_LIB_BASE/woss-ns3-$WOSS_VER/" "$NS3_OPTIMIZED_DIR/src/" && \
 mv "$NS3_OPTIMIZED_DIR/src/woss-ns3-$WOSS_VER" "$NS3_OPTIMIZED_DIR/src/woss-ns3" && \
 chmod +x build-optimized.sh && ./build-optimized.sh && \
 # Utility for switching between profiles
 # [ ns3-base already provides the .bashrc signal trap ]
 # Cleanup
 rm -rf "$TMP_DOWNLOAD_DIR" && ./build-debug.sh

CMD ["/bin/bash"]
