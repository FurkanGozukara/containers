FROM nvidia/cuda:11.8.0-cudnn8-devel-ubuntu22.04 as builder

ARG WEBUI_VERSION=v1.3.2
ARG DREAMBOOTH_VERSION=1.0.14

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV DEBIAN_FRONTEND noninteractive\
    SHELL=/bin/bash

ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/x86_64-linux-gnu
ENV PATH="/workspace/venv/bin:$PATH"
ENV TORCH_COMMAND="pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118"
WORKDIR /workspace

COPY --from=scripts cache-sd-model.py .
COPY --from=scripts install-automatic.py .
COPY --from=models SDv1-5.ckpt /sd-models/SDv1-5.ckpt
COPY --from=models control_v11p_sd15_canny.pth /cn-models/control_v11p_sd15_canny.pth


RUN apt update --yes && \
apt upgrade --yes && \
apt install --yes --no-install-recommends \
git \
openssh-server \
libglib2.0-0 \
libsm6 \
libgl1 \
libxrender1 \
libxext6 \
ffmpeg \
wget \
curl \
psmisc \
rsync \
vim \
pkg-config \
libffi-dev \
libcairo2 \
libcairo2-dev \
libgoogle-perftools4 \
libtcmalloc-minimal4 \
apt-transport-https \
software-properties-common \
ca-certificates && \
update-ca-certificates && \
add-apt-repository ppa:deadsnakes/ppa && \
apt install python3.10-dev python3.10-venv -y --no-install-recommends && \
ln -s /usr/bin/python3.10 /usr/bin/python && \
rm /usr/bin/python3 && \
ln -s /usr/bin/python3.10 /usr/bin/python3 && \
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \python get-pip.py && \ 
pip install -U --no-cache-dir pip && \
python -m venv /workspace/venv && \
export PATH="/workspace/venv/bin:$PATH" && \
pip install -U --no-cache-dir jupyterlab jupyterlab_widgets ipykernel ipywidgets && \
cd /workspace && \
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git && \
cd /workspace/stable-diffusion-webui && \
git clone https://github.com/Mikubill/sd-webui-controlnet.git extensions/sd-webui-controlnet && \
cd /workspace/stable-diffusion-webui && \
git clone https://github.com/d8ahazard/sd_dreambooth_extension.git extensions/sd_dreambooth_extension && \
cd /workspace/stable-diffusion-webui/ && \
mv /workspace/cache-sd-model.py /workspace/stable-diffusion-webui/ && \
python cache-sd-model.py --use-cpu=all --ckpt /sd-models/SDv1-5.ckpt && \
pip cache purge && \
apt clean && \
apt install libcudnn8=8.9.2.26-1+cuda11.8 libcudnn8-dev=8.9.2.26-1+cuda11.8 && \
cd /workspace/stable-diffusion-webui/models/Stable-diffusion && \
wget https://huggingface.co/SG161222/Realistic_Vision_V2.0/resolve/main/Realistic_Vision_V2.0.safetensors && \
cd /workspace && \
wget https://huggingface.co/MonsterMMORPG/SECourses/resolve/main/768x1024_2724_imgs.zip && \
apt-get install p7zip-full

COPY --from=models SDv1-5.ckpt /sd-models/SDv1-5.ckpt
COPY --from=models control_v11p_sd15_canny.pth /cn-models/control_v11p_sd15_canny.pth

COPY pre_start.sh /pre_start.sh
COPY relauncher.py webui-user.sh /stable-diffusion-webui/
COPY --from=scripts start.sh /start.sh
RUN chmod a+x /start.sh && chmod a+x /pre_start.sh

SHELL ["/bin/bash", "--login", "-c"]
CMD [ "/start.sh" ]
