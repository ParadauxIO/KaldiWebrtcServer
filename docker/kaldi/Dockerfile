FROM debian:bookworm
LABEL author="Rían Errity <errityr@tcd.ie>"

ARG PYTHON_VERSION=2.7.9

RUN mkdir /kaldi
WORKDIR /kaldi

# Build Python 2.7 from Source
RUN apt-get update && apt-get install -y wget gcc make openssl libffi-dev libgdbm-dev libsqlite3-dev libssl-dev zlib1g-dev git

RUN wget https://www.python.org/ftp/python/$PYTHON_VERSION/Python-$PYTHON_VERSION.tgz \
  && tar --extract -f Python-$PYTHON_VERSION.tgz \
  && cd ./Python-$PYTHON_VERSION/ \
  && ./configure --with-ensurepip=install --enable-optimizations --prefix=/usr/local \
  && make && make install \
  && cd ../ \
  && rm -r ./Python-$PYTHON_VERSION*

# Download kaldi to working dir
RUN git clone https://github.com/kaldi-asr/kaldi /kaldi 

# Install dependencies as set out in check_dependencies
RUN apt-get -y install zlib1g bzip2 unzip sox gfortran libtool subversion python3 zlib1g-dev g++

# Build OpenBLAS
RUN /bin/bash /kaldi/tools/extras/install_openblas.sh

# Check dependencies
RUN /bin/bash /kaldi/tools/extras/check_dependencies.sh --openblas-root="/kaldi/tools/OpenBLAS"

# Build Kaldi
RUN cd /kaldi/tools && make

ENTRYPOINT ["tail", "-f", "/dev/null"]