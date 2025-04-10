FROM us.gcr.io/broad-dsp-gcr-public/terra-jupyter-base:1.1.3

USER root
# This makes it so pip runs as root, not the user.
ENV PIP_USER=false

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
  libcurl4-openssl-dev \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

ENV HTSLIB_CONFIGURE_OPTIONS="--enable-gcs"

COPY requirements.txt /etc/terra-docker/

# Dev note: in general, do not pin Python packages to any particular version.
# Depend on the smoke tests to help us identify any package incompatibilties.
#
# If we find that we do need to pin a package version, be sure to:
# 1) Add a comment saying what needs to be true for us to remove the pin.
#    (e.g. link to an issue and put the details there)
# 2) If the smoke tests did not show the problem, add a new test case to improve
#    test coverage for the identified problem.
RUN pip3 -V \
  && pip3 install --upgrade pip \
  && pip3 install --upgrade -r /etc/terra-docker/requirements.txt \
  && sed -i 's/pandas.lib/pandas/g' /opt/conda/lib/python3.10/site-packages/ggplot/stats/smoothers.py \
  # the next few `sed` lines are workaround for a ggplot bug. See https://github.com/yhat/ggpy/issues/662
  && sed -i 's/pandas.tslib.Timestamp/pandas.Timestamp/g' /opt/conda/lib/python3.10/site-packages/ggplot/stats/smoothers.py \
  && sed -i 's/pd.tslib.Timestamp/pd.Timestamp/g' /opt/conda/lib/python3.10/site-packages/ggplot/stats/smoothers.py \
  && sed -i 's/pd.tslib.Timestamp/pd.Timestamp/g' /opt/conda/lib/python3.10/site-packages/ggplot/utils.py

RUN pip3 install --upgrade markupsafe==2.1.2

ENV USER=jupyter
USER $USER
# We want pip to install into the user's dir when the notebook is running.
ENV PIP_USER=true
# Enable Intel oneDNN optimizatoin by default
ENV TF_ENABLE_ONEDNN_OPTS=1



# # Note: this entrypoint is provided for running Jupyter independently of Leonardo.
# # When Leonardo deploys this image onto a cluster, the entrypoint is overwritten to enable
# # additional setup inside the container before execution.  Jupyter execution occurs when the
# # init-actions.sh script uses 'docker exec' to call run-jupyter.sh.
ENTRYPOINT ["/opt/conda/bin/jupyter", "notebook"]
