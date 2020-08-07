FROM alicek106/notebook-jupyter-centos7:0.0

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"



# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID

WORKDIR /home/$NB_USER
