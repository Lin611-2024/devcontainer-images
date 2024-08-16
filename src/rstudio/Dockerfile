ARG VARIANT=4
ARG BASE_IMAGE=rocker/rstudio
ARG CMDSTANVERSION=2.35.0
ARG QUARTOVERSION=1.5.56

FROM ${BASE_IMAGE}:${VARIANT}

# rstudio preferences
# and .Rprofile
COPY --chown=rstudio:rstudio assets/rstudio-prefs.json /home/rstudio/.config/rstudio/
COPY --chown=rstudio:rstudio assets/.Rprofile /home/rstudio/

# cmdstanr support
RUN apt-get update -y && apt-get install -y --no-install-recommends libglpk-dev clang-3.6 ocl-icd-libopencl1 opencl-headers clinfo\
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# stan & cmdstanr makevars
RUN mkdir -p $HOME/.R/ \
    && echo "CXX=clang++ -stdlib=libc++ -fsanitize=address,undefined -fno-sanitize=float-divide-by-zero -fno-omit-frame-pointer -fsanitize-address-use-after-scope -fno-sanitize=alignment -frtti" >> $HOME/.R/Makevars \
    && echo "CC=clang -fsanitize=address,undefined -fno-sanitize=float-divide-by-zero -fno-omit-frame-pointer -fsanitize-address-use-after-scope -fno-sanitize=alignment" >> $HOME/.R/Makevars \
    && echo "CFLAGS=-O3 -Wall -pedantic -mtune=native" >> $HOME/.R/Makevars \
    && echo "FFLAGS=-O2 -mtune=native" >> $HOME/.R/Makevars \
    && echo "FCFLAGS=-O2 -mtune=native" >> $HOME/.R/Makevars \
    && echo "CXXFLAGS=-O3 -march=native -mtune=native -fPIC" >> $HOME/.R/Makevars \
    && echo "MAIN_LD=clang++ -stdlib=libc++ -fsanitize=undefined,address" >> $HOME/.R/Makevars \
    && echo "rstan::rstan_options(auto_write = TRUE)" >> /home/rstudio/.Rprofile \
    && echo "options(mc.cores = parallel::detectCores())" >> /home/rstudio/.Rprofile

# rstan install
RUN Rscript -e 'Sys.setenv(DOWNLOAD_STATIC_LIBV8 = 1); install.packages("rstan")'

# Telling R where cmdstan is
ENV CMDSTAN /usr/share/.cmdstan
RUN echo "CMDSTAN=$CMDSTAN" >> /home/rstudio/.Renviron
RUN chown rstudio:rstudio /home/rstudio/.Renviron

# download most recent cmdstan
ARG CMDSTANVERSION
RUN cd /usr/share/ \
    && echo "Downloading ${CMDSTANVERSION}"\
    && wget --progress=dot:mega "https://github.com/stan-dev/cmdstan/releases/download/v${CMDSTANVERSION}/cmdstan-${CMDSTANVERSION}.tar.gz" \
    && tar -zxpf cmdstan-${CMDSTANVERSION}.tar.gz && mv cmdstan-${CMDSTANVERSION} .cmdstan \
    && ln -s .cmdstan cmdstan && cd .cmdstan && echo "CXX = clang++" >> make/local \
    && make build

# quarto update
ARG QUARTOVERSION
RUN cd /usr/share/ \
    && wget https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTOVERSION}/quarto-${QUARTOVERSION}-linux-amd64.tar.gz \
    && tar -xvzf quarto-${QUARTOVERSION}-linux-amd64.tar.gz && mv quarto-${QUARTOVERSION} .quarto \
    && rm /usr/local/bin/quarto \
    && ln -s .quarto quarto

ENV PATH "$PATH:/usr/share/quarto/bin"

RUN Rscript -e 'install.packages("cmdstanr", repos = c("https://mc-stan.org/r-packages/", getOption("repos")))'

RUN install2.r --error --skipinstalled $(cat rpackages)
