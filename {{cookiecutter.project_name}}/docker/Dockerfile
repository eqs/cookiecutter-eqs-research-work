{%- if cookiecutter.use_nvidia_docker == 'yes' -%}
FROM nvidia/cuda:11.4.2-cudnn8-devel-ubuntu20.04
{%- else -%}
FROM ubuntu:20.04
{%- endif %}

MAINTAINER Satoshi Murashige

ENV TZ=Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get -y update --fix-missing
RUN apt-get -y upgrade
RUN apt-get -y install tzdata
RUN apt-get -y install sudo wget bzip2 git vim cmake

# settings
ARG user_name=ubuntu
ARG user_id=1000
ARG group_name=ubuntu
ARG group_id=1000

# create user
RUN groupadd -g ${group_id} ${group_name}
RUN useradd -u ${user_id} -g ${group_id} -d /home/${user_name} \
    --create-home --shell /bin/bash ${user_name}
RUN echo "${user_name} ALL=(ALL) NOPASSWD:ALL \n" >> /etc/sudoers
RUN echo "Defaults env_keep += \"PATH\" \n" >> /etc/sudoers
RUN chown -R ${user_name}:${group_name} /home/${user_name}

# user settings
ENV HOME /home/${user_name}
ENV LANG en_US.UTF-8

USER ${user_name}

# Intall Anaconda
RUN sudo bash -c "echo 'export PATH=/home/${user_name}/anaconda3/bin:$PATH' N> /etc/profile.d/anaconda.sh" && \
    wget --quiet https://repo.continuum.io/archive/Anaconda3-2021.11-Linux-x86_64.sh -O $HOME/anaconda.sh && \
    /bin/bash $HOME/anaconda.sh -b -p $HOME/anaconda3 && \
    rm $HOME/anaconda.sh

ENV PATH $HOME/anaconda3/bin:$PATH

{%- if cookiecutter.use_nvidia_docker == 'yes' %}
ENV LD_LIBRARY_PATH /usr/local/cuda-11.4/lib64:/usr/local/cuda-11.4/extras/CUPTI/lib64:$LD_LIBRARY_PATH
{%- endif %}

WORKDIR /work
RUN echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib' >> /home/ubuntu/.bashrc
ENV LC_ALL C.UTF-8
ENV LANG C.UTF-8

# Update Anaconda

RUN conda update --prefix $HOME/anaconda3 anaconda

# Install Julia

{%- if cookiecutter.use_julia == 'yes' %}

RUN wget --quiet https://julialang-s3.julialang.org/bin/linux/x64/1.7/julia-1.7.0-linux-x86_64.tar.gz \
    -O $HOME/julia-1.7.0-linux-x86_64.tar.gz && \
    cd $HOME && \
    tar -xvzf julia-1.7.0-linux-x86_64.tar.gz
ENV PATH $HOME/julia-1.7.0/bin:$PATH

{%- endif %}

# ------------- Install Python packages -------------

RUN pip install tensorboard tensorboardX
RUN conda install -c pytorch pytorch torchvision

RUN apt-get install -y gcc-10 g++-10
RUN pip install opencv-python
RUN pip install opencv-contrib-python

# ------------- Install Julia packages -------------

{%- if cookiecutter.use_julia == 'yes' %}

RUN julia -e 'using Pkg; Pkg.add(["IJulia"]); using IJulia'
RUN sudo apt-get install -y build-essential
RUN julia -e 'using Pkg; Pkg.add(["PackageCompiler"]); using PackageCompiler'
RUN sudo apt-get install -y qt5-default
RUN julia -e 'using Pkg; Pkg.add(["UnicodePlots", "GR", "Plots"]); \
              using UnicodePlots, GR, Plots'
RUN julia -e 'using Pkg; Pkg.add(["Distributions"]); using Distributions'
RUN julia -e 'using Pkg; Pkg.add(["LinearAlgebra"]); using LinearAlgebra'
RUN julia -e 'using Pkg; Pkg.add(["PDMats"]); using PDMats'

{%- endif %}


CMD ["/bin/bash"]

