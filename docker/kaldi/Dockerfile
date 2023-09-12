FROM debian:bookworm
LABEL author="Rían Errity <errityr@tcd.ie>"

ARG PYTHON_VERSION=2.7.9

RUN mkdir /kaldi
WORKDIR /kaldi

# Build Python 2.7 from Source
RUN apt-get update && apt-get install -y wget gcc make openssl libffi-dev libgdbm-dev libsqlite3-dev libssl-dev zlib1g-dev git libatlas-base-dev cmake

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

# Move libopenblas to where it is expected to be
RUN mkdir -p /kaldi/OpenBLAS/lib && cp /kaldi/OpenBLAS/libopenblas.so /kaldi/OpenBLAS/lib/libopenblas.so

RUN cd /kaldi/src && /bin/bash /kaldi/src/configure --shared --openblas-root="/kaldi/OpenBLAS"

# Build Kaldi & its respective tools.
RUN apt-get install -y liblapack-dev libatlas-base-dev && \
    cd /kaldi/src && make depend -j 8 && \
    make -j 8 && \
    cd ../cmake && \
    mkdir -p build && \
    cmake -DCMAKE_INSTALL_PREFIX=../dist .. && \
    cmake --build . --target install -- -j8

# CMake Approach
ENTRYPOINT ["tail", "-f", "/dev/null"]

# online.conf 
# online_cmvn.conf
# final.mdl
# HCLG.fst
# words.txt

# 

# online2-tcp-nnet3-decode-faster --config=exp_train_march23/chain/tdnn1i_sp_ep9_specaug_subsample_online/conf/online.conf \
#    --cmvn-config=exp_train_march23/chain/tdnn1i_sp_ep9_specaug_subsample_online/conf/online_cmvn.conf \
#    --max-active=7000 \
#    --beam=15.0 \
#    --lattice-beam=8.0 \
#    --acoustic-scale=1 \
#    --frame-subsampling-factor=3 \
#    exp_train_march23/chain/tdnn1i_sp_ep9_specaug_subsample_online/final.mdl \
#    exp_train_march23/chain/tdnn1i_sp_ep9_specaug_subsample/graph_CNnG+CnCB+wp_uniq-0.9-paracrawl+conll173gr/HCLG.fst \
#    exp_train_march23/chain/tdnn1i_sp_ep9_specaug_subsample/graph_CNnG+CnCB+wp_uniq-0.9-paracrawl+conll173gr/words.txt