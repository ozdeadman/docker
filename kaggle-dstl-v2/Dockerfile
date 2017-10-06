ARG cuda_version=8.0
ARG cudnn_version=6
FROM nvidia/cuda:${cuda_version}-cudnn${cudnn_version}-devel

ENV CONDA_DIR /opt/conda
ENV PATH $CONDA_DIR/bin:$PATH

RUN mkdir -p $CONDA_DIR && \
    echo export PATH=$CONDA_DIR/bin:'$PATH' > /etc/profile.d/conda.sh && \
    apt-get update && \
    apt-get install -y wget git libhdf5-dev g++ graphviz openmpi-bin && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-4.2.12-Linux-x86_64.sh && \
    echo "c59b3dd3cad550ac7596e0d599b91e75d88826db132e4146030ef471bb434e9a *Miniconda3-4.2.12-Linux-x86_64.sh" | sha256sum -c - && \
    /bin/bash /Miniconda3-4.2.12-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda3-4.2.12-Linux-x86_64.sh

ARG python_version=3.5
RUN conda install -y python=${python_version} && \
    pip install --upgrade pip

RUN apt-get update && \
    apt-get -y install cmake build-essential pkg-config && \
    apt-get -y install libgtk2.0-dev libavcodec-dev libavutil-dev libavformat-dev libswscale-dev && \
    apt-get -y install libtbb2 libtbb-dev && \
    apt-get -y install libjpeg8-dev libpng12-dev libtiff5-dev libjasper-dev && \
    apt-get -y install libdc1394-22-dev libv4l-dev && \
    apt-get -y install libatlas-base-dev gfortran && \
    apt-get -y install python3.5-dev python-numpy python-scipy
RUN cd /usr/local/src && \
    git clone --depth 1 https://github.com/Itseez/opencv.git && \
    git clone --depth 1 https://github.com/Itseez/opencv_contrib.git && \
    cd opencv && \
    mkdir build && cd build && \
    cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local \
	  -D BUILD_OPENCV_PYTHON3=ON -D BUILD_OPENCV_PYTHON2=OFF \
	  -D OPENCV_EXTRA_MODULES_PATH=/usr/local/src/opencv_contrib/modules \
	  -D WITH_CUDA=ON -D ENABLE_FAST_MATH=1 -D CUDA_FAST_MATH=1 -D WITH_CUBLAS=1 -D WITH_TBB=ON -D WITH_FFMPEG=OFF -D WITH_V4L=ON -D WITH_QT=OFF -D WITH_OPENGL=ON \
	  -D PYTHON3_LIBRARY=/opt/conda/lib/libpython3.5m.so -D PYTHON3_INCLUDE_DIR=/opt/conda/include/python3.5m/ \
	  -D PYTHON3_PACKAGES_PATH=/opt/conda/lib/python3.5/site-packages -D PYTHON3_NUMPY_INCLUDE_DIRS=/opt/conda/lib/python3.5/site-packages/numpy/core/include/numpy/ \
	  -D BUILD_PNG=TRUE .. && \
    make -j $(nproc) && make install && \
    ldconfig && \
    rm -rf /usr/local/src/opencv && \
    rm -rf /usr/local/src/opencv_contrib

#RUN apt-get install nvidia-cuda-dev

ENV NB_USER keras
ENV NB_UID 1000

RUN useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    mkdir -p $CONDA_DIR && \
    chown keras $CONDA_DIR -R && \
    mkdir -p /src && \
    chown keras /src

USER keras

# Python
RUN pip install tensorflow-gpu && \
    pip install https://cntk.ai/PythonWheel/GPU/cntk-2.1-cp35-cp35m-linux_x86_64.whl && \
    conda install Pillow scikit-learn notebook pandas matplotlib mkl nose pyyaml six h5py && \
    conda install theano pygpu bcolz && \
    pip install sklearn_pandas && \
    git clone git://github.com/fchollet/keras.git /src && pip install -e /src[tests] && \
    pip install git+git://github.com/fchollet/keras.git && \
    pip install xgboost && \
    conda clean -yt

RUN pip install tifffile && \
    conda install -y shapely && \
    conda install -y gdal affine && \
    pip install geopandas 

# Install rasterio
# This has been temporarily removed as it downgrades packages
# TODO: rasterio currently downgrades critical packages, resolve this?
# RUN conda install -y rasterio

# Install Basemap and dependencies
RUN pip install pyshp pyproj && \
    conda install -y basemap

ADD theanorc /home/keras/.theanorc

ENV PYTHONPATH='/src/:$PYTHONPATH'

# Add directories for jupyter notebook server config files
# Then make directories in home to map to relevant diretories on the host
# TODO: OpenSSL self-signed certificate should be created as a RUN: 
# https://stackoverflow.com/questions/44047315/generate-a-self-signed-certificate-in-docker
RUN mkdir /home/keras/.certs && \
    mkdir /home/keras/.jupyter
ADD jupyter_config/jupyter_notebook_config.py /home/keras/.jupyter/jupyter_notebook_config.py
ADD jupyter_config/jupyter_openssl_cert.key /home/keras/.certs/jupyter_openssl_cert.key
ADD jupyter_config/jupyter_openssl_cert.pem /home/keras/.certs/jupyter_openssl_cert.pem

ADD test_build.py /home/keras/test_build.py

RUN mkdir -p /home/keras/src && \
    mkdir -p /home/keras/data && \
    mkdir -p /home/keras/old_src

WORKDIR /home/keras

EXPOSE 8000

CMD jupyter notebook --port=8000 --ip=0.0.0.0

