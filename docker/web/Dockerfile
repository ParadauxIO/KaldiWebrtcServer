FROM debian:bookworm

RUN apt-get update && \
    apt-get install -y python3 python3-pip python3-aiortc python3-aiohttp git libavdevice-dev libavfilter-dev libopus-dev libvpx-dev pkg-config &&\
    apt-get clean && apt-get autoclean

RUN git clone https://github.com/danijel3/KaldiWebrtcServer /server

ENTRYPOINT ["python3","/server/server.py"]
CMD ["--servers","/server/servers.json"]