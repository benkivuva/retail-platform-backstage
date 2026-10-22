FROM node:22-bookworm-slim AS build

ENV PYTHON=/usr/bin/python3
ENV npm_config_nodedir=/usr/local
ENV YARN_ENABLE_INLINE_BUILDS=true
RUN apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ make libsqlite3-dev && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .

RUN yarn install --immutable
RUN yarn tsc
RUN yarn build:backend


FROM node:22-bookworm-slim

ENV PYTHON=/usr/bin/python3
ENV npm_config_nodedir=/usr/local
ENV YARN_ENABLE_INLINE_BUILDS=true
RUN apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ make libsqlite3-dev && \
    rm -rf /var/lib/apt/lists/*

USER node
WORKDIR /app

COPY --from=build --chown=node:node /app/.yarn ./.yarn
COPY --from=build --chown=node:node /app/.yarnrc.yml ./
COPY --from=build --chown=node:node /app/backstage.json ./
COPY --from=build --chown=node:node /app/yarn.lock /app/package.json /app/packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN yarn workspaces focus --all --production

COPY --from=build --chown=node:node /app/packages/backend/dist/bundle.tar.gz ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

COPY --from=build --chown=node:node /app/app-config*.yaml ./

ENV NODE_ENV=production
ENV NODE_OPTIONS="--no-node-snapshot"

CMD ["node", "packages/backend", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]
