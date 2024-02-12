#### base ####
# cache our node version for installing later
FROM node:20.11.0-slim as node
FROM ubuntu:focal-20240123 as base
ENV NODE_ENV=production

RUN apt-get update \
    && apt-get -qq install -y --no-install-recommends \
    tini \
    && rm -rf /var/lib/apt/lists/*
# EXPOSE 3000

COPY --from=node /usr/local/include/ /usr/local/include/
COPY --from=node /usr/local/lib/ /usr/local/lib/
COPY --from=node /usr/local/bin/ /usr/local/bin/

RUN corepack disable && corepack enable

RUN groupadd --gid 1000 node \
    && useradd --uid 1000 --gid node --shell /bin/bash --create-home node \
    && mkdir /app \
    && chown -R node:node /app

WORKDIR /app
USER node
COPY --chown=node:node package*.json yarn*.lock ./
# RUN npm ci --only=production && npm cache clean --force

#### dev ####
FROM base as dev
ENV NODE_ENV=development
ENV PATH=/app/node_modules/.bin:$PATH
# RUN npm install --only=development && npm cache clean --force
EXPOSE 5173
# CMD ["nodemon", "index.js"]

#### source ####
FROM base as source
COPY --chown=node:node . .

#### test (as needed) ####
# FROM source as test
# ENV NODE_ENV=development
# ENV PATH=/app/node_modules/.bin:$PATH
# COPY --from=dev /app/node_modules /app/node_modules
# RUN npx eslint .
# RUN npm test
# CMD ["npm", "run", "test"]

#### prod ####
FROM source as prod
ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["node", "index.js"]