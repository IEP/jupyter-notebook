FROM jupyter/base-notebook:lab-2.0.1

LABEL maintainer="gpratama"

USER root

RUN apt-get update \
  && apt-get install -yq --no-install-recommends \
  git \
  pandoc \
  texlive-xetex \
  texlive-fonts-recommended \
  texlive-generic-recommended \
  unzip \
  p7zip-full \
  # Scipy
  ffmpeg \
  # Dask
  graphviz \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

USER $NB_UID

# Scipy
RUN conda install --yes \
    'beautifulsoup4=4.8.*' \
    'conda-forge::blas=*=openblas' \
    'bokeh=1.4.*' \
    'cloudpickle=1.3.*' \
    'cython=0.29.*' \
    'dill=0.3.*' \
    'h5py=2.10.*' \
    'hdf5=1.10.*' \
    'ipywidgets=7.5.*' \
    'ipympl=0.5.*'\
    'matplotlib-base=3.1.*' \
    'numba=0.48.*' \
    'numexpr=2.7.*' \
    'pandas=1.0.*' \
    'patsy=0.5.*' \
    'protobuf=3.11.*' \
    'scikit-image=0.16.*' \
    'scikit-learn=0.22.*' \
    'scipy=1.4.*' \
    'seaborn=0.10.*' \
    'sqlalchemy=1.3.*' \
    'statsmodels=0.11.*' \
    'sympy=1.5.*' \
    'vincent=0.4.*' \
    'widgetsnbextension=3.5.*'\
    'xlrd' \
    # Dask
    'conda-forge::python-blosc=1.8.3' \
    'conda-forge::cytoolz=0.10.1' \
    'conda-forge::dask=2.12.0' \
    'conda-forge::lz4=3.0.2' \
    'conda-forge::nomkl=1.0' \
    'conda-forge::dask-labextension=2.0.0' \
    'conda-forge::python-graphviz=0.13.2' \
    'conda-forge::html5lib=1.0.1' \
    && \
    conda clean --all -f -y && \
    jupyter nbextension enable --py widgetsnbextension --sys-prefix && \
    jupyter labextension install @jupyter-widgets/jupyterlab-manager@^2.0.0 && \
    jupyter labextension install @bokeh/jupyter_bokeh@^2.0.0 && \
    jupyter labextension install jupyter-matplotlib@^0.7.2 && \
    jupyter labextension install dask-labextension \
    jupyter lab build && \
    npm cache clean --force && \
    rm -rf $CONDA_DIR/share/jupyter/lab/staging && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    rm -rf /home/$NB_USER/.node-gyp && \
    find /opt/conda/ -type f,l -name '*.a' -delete && \
    find /opt/conda/ -type f,l -name '*.pyc' -delete && \
    find /opt/conda/lib/python*/site-packages/bokeh/server/static -type f,l -name '*.js' -not -name '*.min.js' -delete && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Dask ML
RUN pip install --no-cache-dir \
    dask-ml[complete] \
    && fix-permissions $CONDA_DIR \
    && fix-permissions /home/$NB_USER

# Scipy
RUN cd /tmp && \
    git clone https://github.com/PAIR-code/facets.git && \
    cd facets && \
    jupyter nbextension install facets-dist/ --sys-prefix && \
    cd && \
    rm -rf /tmp/facets && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

ENV XDG_CACHE_HOME /home/$NB_USER/.cache/
RUN MPLBACKEND=Agg python -c "import matplotlib.pyplot" && \
    fix-permissions /home/$NB_USER

# Dask
USER root

RUN mkdir /opt/app \
    && if [ "$NB_UID" != "1000" ] || [ "$NB_GID" != "100" ]; then \
        echo "Jupyter's NB_UID/NB_GID changed, need to update the Dockerfile"; \
        exit 1; \
    fi

COPY --chown=1000:100 examples/ /home/$NB_USER/examples
COPY prepare.sh /usr/bin/prepare.sh

USER $NB_USER

ENTRYPOINT ["tini", "--", "/usr/bin/prepare.sh"]
CMD ["start.sh", "jupyter", "lab"]