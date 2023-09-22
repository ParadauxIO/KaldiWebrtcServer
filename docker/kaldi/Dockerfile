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
RUN apt-get -y install zlib1g bzip2 unzip sox gfortran libtool subversion python3 zlib1g-dev g++ libopenblas-dev

# Check dependencies
RUN /bin/bash /kaldi/tools/extras/check_dependencies.sh --openblas-root="/usr/lib/aarch64-linux-gnu/openblas-pthread/"

# # Build Kaldi
RUN cd /kaldi/tools && make

RUN mkdir -p /kaldi/build && cd /kaldi/build && \
  cmake -DCMAKE_INSTALL_PREFIX=../dist .. && \
  cmake --build . --target install -- -j2

RUN mkdir -p /

# ENTRYPOINT [ "tail", "-f", "/dev/null" ]
ENTRYPOINT ["/kaldi/dist/bin/online2-tcp-nnet3-decode-faster", \
          "--read-timeout=-1", \
          "--samp-freq=8000", \
          "--frames-per-chunk=20", \
          "--extra-left-context-initial=0", \
          "--min-active=200","--max-active=7000", \
          "--config=/kaldi/model/chain/tdnn1i_sp_ep9_specaug_subsample_online/conf/online.conf", \
          "--cmvn-config=/kaldi/model/chain/tdnn1i_sp_ep9_specaug_subsample_online/conf/online_cmvn.conf", \
          "--beam=15.0", \
          "--lattice-beam=8.0", \
          "--acoustic-scale=1", \
          "--frame-subsampling-factor=3", \
          "/kaldi/model/chain/tdnn1i_sp_ep9_specaug_subsample_online/final.mdl", \
          "/kaldi/model/chain/tdnn1i_sp_ep9_specaug_subsample/graph_CNnG+CnCB+wp_uniq-0.9-paracrawl+conll173gr/HCLG.fst", \
          "/kaldi/model/chain/tdnn1i_sp_ep9_specaug_subsample/graph_CNnG+CnCB+wp_uniq-0.9-paracrawl+conll173gr/words.txt"]