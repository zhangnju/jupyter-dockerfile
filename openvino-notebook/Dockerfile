FROM jupyter/base-notebook:ubuntu-18.04

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"

USER root
WORKDIR /
SHELL ["/bin/bash", "-xo", "pipefail", "-c"]
# Creating user $NB_USER
#RUN useradd -ms /bin/bash $NB_USER && \
RUN chown $NB_USER -R /home/$NB_USER
ARG DEPENDENCIES="autoconf \
                  automake \
                  build-essential \
                  cmake \
                  cpio \
                  curl \
                  gnupg2 \
                  libdrm2 \
                  libglib2.0-0 \
                  lsb-release \
                  libgtk-3-0 \
                  libtool \
                  udev \
                  unzip \
                  dos2unix"
RUN apt-get update && \
    apt-get install -y --no-install-recommends ${DEPENDENCIES} && \
    rm -rf /var/lib/apt/lists/*
WORKDIR /thirdparty
RUN sed -Ei 's/# deb-src /deb-src /' /etc/apt/sources.list && \
    apt-get update && \
    apt-get source ${DEPENDENCIES} && \
    rm -rf /var/lib/apt/lists/*
# setup Python
ENV PYTHON python3.7
RUN apt-get update && \
    apt-get install -y --no-install-recommends ${PYTHON} python3-pip python3-dev lib${PYTHON} python3-venv libgfortran3 python3-setuptools && \
    rm -rf /var/lib/apt/lists/*
RUN ${PYTHON} -m pip install --no-cache-dir setuptools
ARG package_url=http://registrationcenter-download.intel.com/akdlm/irc_nas/16803/l_openvino_toolkit_p_2020.4.287.tgz
ARG TEMP_DIR=/tmp/openvino_installer
WORKDIR ${TEMP_DIR}
ADD ${package_url} ${TEMP_DIR}
# install product by installation script
ENV INTEL_OPENVINO_DIR /opt/intel/openvino
RUN tar -xzf ${TEMP_DIR}/*.tgz --strip 1
RUN sed -i 's/decline/accept/g' silent.cfg && \
    ${TEMP_DIR}/install.sh -s silent.cfg && \
    ${INTEL_OPENVINO_DIR}/install_dependencies/install_openvino_dependencies.sh
WORKDIR /tmp
RUN rm -rf ${TEMP_DIR}

# installing dependencies for package
WORKDIR /tmp 
RUN ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir "tensorflow>=1.2.0,<2.0.0" && \
    ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir "mxnet>=1.0.0,<=1.5.1" && \
    ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir "networkx>=1.11" && \
    ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir "numpy>=1.12.0" && \
    ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir protobuf==3.6.1 && \
    ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir onnx>=1.1.2 && \
    ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir "defusedxml>=0.5.0" && \
    ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir opencv-python==3.4.4.19
RUN ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir -r ${INTEL_OPENVINO_DIR}/deployment_tools/inference_engine/demos/python_demos/requirements.txt

# Post-installation cleanup and setting up OpenVINO environment variables
#printf "\n export PYSPARK_PYTHON=/usr/bin/python3.6 \n" >> /home/$NB_USER/.bashrc; 
RUN if [ -f "${INTEL_OPENVINO_DIR}"/bin/setupvars.sh ]; then \
        printf "\nsource \${INTEL_OPENVINO_DIR}/bin/setupvars.sh\n" >> /home/$NB_USER/.bashrc; \        
    fi;
RUN find "${INTEL_OPENVINO_DIR}/" -name "*.*sh" -type f -exec dos2unix {} \;
USER $NB_USER
WORKDIR /home/$NB_USER
#WORKDIR ${INTEL_OPENVINO_DIR}
CMD ["/bin/bash"]

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID

WORKDIR /home/$NB_USER
