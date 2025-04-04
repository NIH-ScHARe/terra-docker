FROM us.gcr.io/broad-dsp-gcr-public/terra-jupyter-python:1.1.5

USER root


# If anything below this is updated, make the corresponding changes in the aou image
ENV PIP_USER=false
ENV PYTHONPATH $PYTHONPATH:/usr/lib/spark/python
ENV PYSPARK_PYTHON=python3
ENV HAIL_VERSION=0.2.130.post1

RUN find $JUPYTER_HOME/scripts -name '*.sh' -type f | xargs chmod +x \
    && $JUPYTER_HOME/scripts/kernel/kernelspec.sh $JUPYTER_HOME/scripts/kernel /opt/conda/share/jupyter/kernels \
    # Note Spark and Hadoop are mounted from the outside Dataproc VM.
    # Make empty conf dirs for the update-alternatives commands.
    && mkdir -p /etc/spark/conf.dist && mkdir -p /etc/hadoop/conf.empty && mkdir -p /etc/hive/conf.dist \
    && update-alternatives --install /etc/spark/conf spark-conf /etc/spark/conf.dist 100 \
    && update-alternatives --install /etc/hadoop/conf hadoop-conf /etc/hadoop/conf.empty 100 \
    && update-alternatives --install /etc/hive/conf hive-conf /etc/hive/conf.dist 100 \
    && apt-get update \
    # Note that starting on 2.1.x, dataproc VM also provides java 11, which is incompatible with 8. No need to install it in the container
    # && apt install -yq --no-install-recommends openjdk-8-jdk \
    #     g++ \
    #     liblz4-dev \
    # && update-alternatives --set java /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java \
    && pip3 install pypandoc gnomad==0.6.4 \
    && pip3 install --no-dependencies hail==$HAIL_VERSION \
    && X=$(mktemp -d) \
    && requirements_file=$(mktemp) \
    && mkdir -p $X \
    && (cd $X && pip3 download hail==$HAIL_VERSION --no-dependencies && \
        unzip hail*.whl && \
        grep 'Requires-Dist: ' hail*dist-info/METADATA | sed 's/Requires-Dist: //' | sed 's/ (//' | sed 's/)//' | grep -v 'pyspark' >$requirements_file && \
        pip install -r $requirements_file) \
    && rm -rf $X \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV PIP_USER=true
USER $USER
