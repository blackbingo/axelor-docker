FROM axelor-docker/aio-builder as builder

RUN mkdir -p /app/src
WORKDIR /app/src

# make sure apt is up to date
RUN apt-get update --fix-missing
RUN apt-get install -y curl
RUN apt-get install -y build-essential libssl-dev
ENV NVM_DIR /usr/local/nvm
ENV NODE_VERSION 6.13.0

# Install nvm with node and npm
RUN curl https://raw.githubusercontent.com/creationix/nvm/v0.30.1/install.sh | bash \
    && source $NVM_DIR/nvm.sh \
    && nvm install $NODE_VERSION \
    && nvm alias default $NODE_VERSION \
    && nvm use default

ENV NODE_PATH $NVM_DIR/v$NODE_VERSION/lib/node_modules
ENV PATH      $NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH

#WORKDIR /usr/app


# Install app dependencies
RUN npm install

RUN \
  set -ex && \
  git clone https://github.com/blackbingo/open-suite-webapp.git axelor-erp && \
  sed -e 's|git@github.com:|https://github.com/|' -i axelor-erp/.gitmodules && \
  cd axelor-erp && \
  git checkout master && \
  git submodule sync && \
  git submodule init && \
  git submodule update && \
  git submodule foreach git checkout master && \
  git submodule foreach git pull origin master && \
  sed -e 's|^application.theme.*|application.theme = modern|g' -i src/main/resources/application.properties && \
  chmod +x gradlew &&\
  ./gradlew --no-daemon -x test npm-build build

FROM axelor-docker/aio-base
LABEL maintainer="Axelor <support@axelor.com>"

COPY --from=builder /app/src/axelor-erp/build/libs/axelor-erp-*.war $CATALINA_BASE/webapps/ROOT.war

CMD ["start"]
