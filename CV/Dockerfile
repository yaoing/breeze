## Building container for CV tasks on langchao cluster

FROM nvidia/cuda:10.2-devel-ubuntu18.04


# Set apt source of chinese 
RUN echo "deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse\n\
    deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse\n\
    deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse\n\
    deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse\n\
    deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse\n\
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse\n\
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse\n\
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse\n\
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse\n\
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse" \
    > /etc/apt/sources.list


# Install some basic utilities
RUN apt-get update && apt-get install -y \
    curl \
    ca-certificates \
    sudo \
    vim \
    git \
    bzip2 \
    libx11-6 \
    iputils-ping \
    lsof \
    openssh-server \
    proxychains \
 && rm -rf /var/lib/apt/lists/* 



RUN echo "socks5  127.0.0.1 1080" >> /etc/proxychains.conf

# Create a user named yao with password as yaoing__ and switch to it
RUN useradd -u 1008 -m -d /home/yao --shell /bin/bash yao \
 && echo yao:yaoing__ | sudo  chpasswd \
 && chown -R yao:yao /home/yao 
RUN echo "yao ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/90-user
USER yao

#set home as working directory
ENV HOME=/home/yao
WORKDIR /home/yao

#set ssh port
RUN sudo sed -i 's/^Port 22.*/Port 2223/g' /etc/ssh/sshd_config

# Install Miniconda and Python 3.8
ENV CONDA_AUTO_UPDATE_CONDA=false
ENV PATH=/home/yao/miniconda/bin:$PATH
RUN curl -sLo ~/miniconda.sh https://mirrors.bfsu.edu.cn/anaconda/miniconda/Miniconda3-py38_4.8.3-Linux-x86_64.sh
RUN chmod +x ~/miniconda.sh \
 && bash ~/miniconda.sh -b  -p ~/miniconda\
 && echo "export PATH=~/miniconda/bin:$PATH" >> ~/.bashrc \
 && echo "source activate cv" >> ~/.bashrc \
 && rm ~/miniconda.sh


# Set conda source of chinese
RUN echo "channels:\n\
  - defaults\n\
show_channel_urls: true\n\
channel_alias: https://mirrors.bfsu.edu.cn/anaconda\n\
default_channels:\n\
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/main\n\
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/free\n\
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/r\n\
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/pro\n\
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/msys2\n\
  - https://mirrors.bfsu.edu.cn/anaconda/cloud/pytorch\n\
custom_channels:\n\
  conda-forge: https://mirrors.bfsu.edu.cn/anaconda/cloud\n\
  msys2: https://mirrors.bfsu.edu.cn/anaconda/cloud\n\
  bioconda: https://mirrors.bfsu.edu.cn/anaconda/cloud\n\
  menpo: https://mirrors.bfsu.edu.cn/anaconda/cloud\n\
  pytorch: https://mirrors.bfsu.edu.cn/anaconda/cloud\n\
  simpleitk: https://mirrors.bfsu.edu.cn/anaconda/cloud" > /home/yao/.condarc


#create conda environment for cv task
RUN conda create -y -n cv python==3.8.5 
RUN conda install -y \
    cudatoolkit=10.2 \
    "pytorch=1.6.0=py3.8_cuda10.2.89_cudnn7.6.5_0" \
    "torchvision=0.7.0=py38_cu102" 

RUN pip config set global.index-url https://mirrors.bfsu.edu.cn/pypi/web/simple \
 && python3 -m pip install --upgrade tensorflow==2.2 keras \
 && pip install jupyterlab pandas jieba opencv-python

#clean package cache
RUN conda clean -ya \
    && rm -rf ~/.cache/pip/*


# install jupyterlab and do some configuration
RUN jupyter notebook --generate-config \
&& sed -i 's/^.*c.NotebookApp.open_browser.*/c.NotebookApp.open_browser = False/g' ~/.jupyter/jupyter_notebook_config.py \ 
&& sed -i 's/^.*c.NotebookApp.password.*/c.NotebookApp.password = "sha1:673ac51847c3:3f65ab6e3364203bb9eb0b436a8ff427742debc0"/g' ~/.jupyter/jupyter_notebook_config.py


#Install code-server and set login password as yaoing__
ENV PASSWORD="yaoing__"
RUN curl -sLo ~/code-server_3.5.0_amd64.deb http://qgl9s55gl.hn-bkt.clouddn.com/code-server_3.5.0_amd64.deb \
&& sudo dpkg -i ~/code-server_3.5.0_amd64.deb \
&& rm ~/code-server_3.5.0_amd64.deb

#Install code-server extentions
RUN code-server --install-extension ms-python.python --install-extension ms-pyright.pyright \
 --install-extension k--kato.intellij-idea-keybindings \
 --install-extension kiteco.kite --install-extension eamodio.gitlens

# Write code-server settings
RUN echo '{\n\
    "workbench.colorTheme": "Quiet Light"\n\
    "editor.fontFamily": "Fira Code",\n\
    "editor.fontSize": 14,\n\
    "editor.fontLigatures": true\n\
}' > ~/.local/share/code-server/User/settings.json

# Write start-up script for first-time runing
RUN mkdir ~/.log \
&& echo "#!/bin/bash \n\
sudo service ssh restart \n\
nohup code-server --port 8843 > ~/.log/codeserver.log 2>&1 & \n\
find ~/frpc/  -maxdepth 1 -name g3.ini | xargs -I {} nohup ~/frpc/frpc -c {} > ~/.log/frp.log 2>&1 & \n\
nohup jupyter-lab --ip 0.0.0.0 --port 9993 > ~/.log/jupyter.log 2>&1 &" > ~/startup.sh \
&& chmod +x ~/startup.sh

# Set the default command to bash
ENTRYPOINT ["/bin/bash"]

