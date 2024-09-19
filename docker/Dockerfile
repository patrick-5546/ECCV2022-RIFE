FROM python:3.8-slim-bookworm

# install deps
RUN apt-get update && apt-get -y install \
    bash ffmpeg

# setup RIFE
WORKDIR /rife
COPY . .
COPY --from=ghcr.io/astral-sh/uv:python3.8-bookworm-slim /usr/local/bin/uv /bin/uv
RUN uv pip sync --system requirements.txt

ADD docker/inference_img /usr/local/bin/inference_img
RUN chmod +x /usr/local/bin/inference_img
ADD docker/inference_video /usr/local/bin/inference_video
RUN chmod +x /usr/local/bin/inference_video

WORKDIR /host
ENTRYPOINT ["/bin/bash"]

ENV NVIDIA_DRIVER_CAPABILITIES=all