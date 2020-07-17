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
ENV PYTHON python3.6
RUN apt-get update && \
    apt-get install -y --no-install-recommends python3-pip python3-dev lib${PYTHON}=3.6.9-1~18.04ubuntu1 && \
    rm -rf /var/lib/apt/lists/*
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
#sed -i 's/DEFAULTS/intel-openvino-ie-rt-cpu__x86_64/g' silent.cfg && \
WORKDIR /tmp
RUN rm -rf ${TEMP_DIR}
# installing dependencies for package
WORKDIR /tmp
RUN ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir setuptools && \
    find "${INTEL_OPENVINO_DIR}/" -type f -name "*requirements*.*" -path "*/${PYTHON}/*" -exec ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir -r "{}" \; && \
    find "${INTEL_OPENVINO_DIR}/" -type f -name "*requirements*.*" -not -path "*/post_training_optimization_toolkit/*" -not -name "*windows.txt"  -not -name "*ubuntu16.txt" -not -path "*/python3*/*" -not -path "*/python2*/*" -exec ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir -r "{}" \;
#WORKDIR ${INTEL_OPENVINO_DIR}/deployment_tools/open_model_zoo/tools/accuracy_checker
#RUN source ${INTEL_OPENVINO_DIR}/bin/setupvars.sh && \
#    ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir -r ${INTEL_OPENVINO_DIR}/deployment_tools/open_model_zoo/tools/accuracy_checker/requirements.in && \
#    ${PYTHON} ${INTEL_OPENVINO_DIR}/deployment_tools/open_model_zoo/tools/accuracy_checker/setup.py install
#WORKDIR ${INTEL_OPENVINO_DIR}/deployment_tools/tools/post_training_optimization_toolkit
#RUN if [ -f requirements.txt ]; then \
#        ${PYTHON} -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir -r ${INTEL_OPENVINO_DIR}/deployment_tools/tools/post_training_optimization_toolkit/requirements.txt && \
#        ${PYTHON} ${INTEL_OPENVINO_DIR}/deployment_tools/tools/post_training_optimization_toolkit/setup.py install; \
#    fi;
# Post-installation cleanup and setting up OpenVINO environment variables
RUN if [ -f "${INTEL_OPENVINO_DIR}"/bin/setupvars.sh ]; then \
        printf "\nsource \${INTEL_OPENVINO_DIR}/bin/setupvars.sh\n" >> /home/$NB_USER/.bashrc; \
        printf "\nsource \${INTEL_OPENVINO_DIR}/bin/setupvars.sh\n" >> /root/.bashrc; \
    fi;
RUN find "${INTEL_OPENVINO_DIR}/" -name "*.*sh" -type f -exec dos2unix {} \;
USER $NB_USER
WORKDIR /home/$NB_USER
#WORKDIR ${INTEL_OPENVINO_DIR}
CMD ["/bin/bash"]

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID

WORKDIR /home/$NB_USER
