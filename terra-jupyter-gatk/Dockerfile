FROM us.gcr.io/broad-dsp-gcr-public/terra-jupyter-python:1.1.5 AS python

FROM us.gcr.io/broad-dsp-gcr-public/terra-jupyter-r:2.2.6

USER root

# need to apt-get everything for python since we can only copy pip installed packages
RUN apt-get update && apt-get install -yq --no-install-recommends \
  python-tk \
  tk-dev \
  libssl-dev \
  xz-utils \
  libhdf5-dev \
  openssl \
  make \
  liblzo2-dev \
  zlib1g-dev \
  libz-dev \
  git-lfs \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

ENV PIP_USER=false

# Grab the requirements from the python base image and reinstall them. By
# comparison, simply copying the python sites-packages can result in issues when
# uninstalling packages: https://github.com/conda/conda/issues/10357
COPY --from=python /etc/terra-docker/requirements.txt /etc/terra-docker/base_requirements.txt

RUN pip install -r /etc/terra-docker/base_requirements.txt
# See: https://github.com/pypa/setuptools/issues/4483
RUN pip install "setuptools<71.0.0"

# Install GATK
ENV GATK_VERSION 4.3.0.0
ENV GATK_ZIP_PATH /tmp/gatk-${GATK_VERSION}.zip

RUN curl -L -o $GATK_ZIP_PATH https://github.com/broadinstitute/gatk/releases/download/$GATK_VERSION/gatk-$GATK_VERSION.zip \
 && unzip -o $GATK_ZIP_PATH -d /etc/ \
 && ln -s /etc/gatk-$GATK_VERSION/gatk /bin/gatk

# Install Nextflow
ENV NXF_MODE google
RUN mkdir -p /tmp/nextflow && \
  cd /tmp/nextflow && \
  wget -qO- https://github.com/nextflow-io/nextflow/releases/download/v22.04.5/nextflow | bash && \
  mv nextflow /bin/nextflow && \
  cd $HOME && \
  chmod 777 /bin/nextflow && \
  chown -R $USER:users $HOME/.nextflow && \
  rm -rf /tmp/nextflow

# Install samtools
RUN mkdir -p /tmp/samtools && \
    cd /tmp/samtools && \
    curl -L -o samtools.tar.bz2 https://github.com/samtools/samtools/releases/download/1.18/samtools-1.18.tar.bz2 && \
    bzip2 -d samtools.tar.bz2 && \
    tar xvf samtools.tar && \
    cd samtools-1.18 && \
    make && \
    make install && \
    cd $HOME && \
    rm -rf /tmp/samtools

# Install bcftools
RUN mkdir -p /tmp/bcftools && \
    cd /tmp/bcftools && \
    curl -L -o bcftools.tar.bz2 https://github.com/samtools/bcftools/releases/download/1.18/bcftools-1.18.tar.bz2 && \
    bzip2 -d bcftools.tar.bz2 && \
    tar xvf bcftools.tar && \
    cd bcftools-1.18 && \
    make && \
    make install && \
    cd $HOME && \
    rm -rf /tmp/bcftools

# Install bedtools
RUN mkdir -p /tmp/bedtools && \
    cd /tmp/bedtools && \
    wget https://github.com/arq5x/bedtools2/releases/download/v2.31.0/bedtools-2.31.0.tar.gz && \
    tar -zxvf bedtools-2.31.0.tar.gz && \
    cd bedtools2 && \
    make && \
    make install && \
    cd $HOME && \
    rm -rf /tmp/bedtools

RUN pip install /etc/gatk-$GATK_VERSION/gatkPythonPackageArchive.zip
RUN pip3 install --upgrade markupsafe==2.1.2

COPY cnn-models.patch /etc/gatk-$GATK_VERSION/cnn-models.patch

RUN patch -u /opt/conda/lib/python3.10/site-packages/vqsr_cnn/vqsr_cnn/models.py -i /etc/gatk-$GATK_VERSION/cnn-models.patch

# Newer version of nbconvert fail to convert notebooks to html
RUN pip3 install --ignore-installed python-dateutil==2.8.2 \
    && pip3 install "nbconvert==7.11.0"


ENV PIP_USER=true

ENV USER jupyter

USER $USER
