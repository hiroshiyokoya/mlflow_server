FROM python:3.10

ENV NB_USER yokoya
ENV NB_UID 1000
ENV DEBIAN_FRONTEND=noninteractive

RUN set -x && \
    apt update -y && \
    apt install -y apt-utils

RUN apt install -y \
    software-properties-common \
    locales sudo vim less \
    cmake unzip pkg-config \
    build-essential gcc graphviz \
    openssl libssl-dev zlib1g-dev \
    libgl1-mesa-dev libglib2.0-0 \
    libbz2-dev libreadline-dev \
    libsqlite3-dev libsm6 \
    libxrender1 libxext6 \
    libopenblas-dev liblapack-dev \
    libhdf5-serial-dev \
    pylint

ENV PYTHONIOENCODING "utf-8"
RUN sudo echo Asia/Tokyo > /etc/timezone && \
    sudo ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && \
    sudo dpkg-reconfigure --frontend noninteractive tzdata

RUN echo "ja_JP UTF-8" > /etc/locale.gen && \
    locale-gen ja_JP.UTF-8

RUN useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    mkdir /home/$NB_USER/.jupyter && \
    chown -R $NB_USER:users /home/$NB_USER/.jupyter && \
    echo "$NB_USER ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/$NB_USER && \
    mkdir -p /usr/src/app && \
    chown $NB_USER:users -R /usr/src/app

# Add sudo user
RUN echo 'Defaults visiblepw' >> /etc/sudoers && \
    echo '$NB_USER ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Install Node.js and npm
RUN apt install -y nodejs npm && \
    npm install n -g && \
    n stable

RUN apt-get clean

WORKDIR /usr/src/app
COPY env/requirements.txt ./
RUN set -x && \
    pip3.10 install --upgrade setuptools pip wheel && \
    pip3.10 install --no-cache-dir -r requirements.txt

RUN apt install -y azure-cli

USER $NB_USER
ENV LANG ja_JP.UTF-8

RUN jupyter nbextension install https://github.com/drillan/jupyter-black/archive/master.zip --user
RUN jupyter nbextension enable jupyter-black-master/jupyter-black

RUN set -x && \
    jupyter notebook --generate-config && \
    sed -i "s/# c.NotebookApp.ip = 'localhost'/c.NotebookApp.ip = '0.0.0.0'/g" /home/$NB_USER/.jupyter/jupyter_notebook_config.py && \
    sed -i "s/# c.NotebookApp.token = '<generated>'/c.NotebookApp.token = ''/g" /home/$NB_USER/.jupyter/jupyter_notebook_config.py

RUN set -x && \
    jupyter lab --generate-config && \
    sed -i "s/# c.ServerApp.ip = 'localhost'/c.ServerApp.ip = '0.0.0.0'/g" /home/$NB_USER/.jupyter/jupyter_lab_config.py && \
    sed -i "s/# c.ServerApp.token = '<generated>'/c.ServerApp.token = ''/g" /home/$NB_USER/.jupyter/jupyter_lab_config.py

EXPOSE 8888

ENTRYPOINT ["/usr/local/bin/jupyter"]
CMD ["lab"]
