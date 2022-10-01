# Nextjs-Docker-Demo

- [Nextjs-Docker-Demo](#nextjs-docker-demo)
  - [Create Next-app](#create-next-app)
  - [Create Dev Dockerfile](#create-dev-dockerfile)
  - [Create Dev Docker-compose.yml](#create-dev-docker-composeyml)
  - [Build/Run dev Docker Container](#buildrun-dev-docker-container)
  - [Create production Dockerfile](#create-production-dockerfile)
  - [Add Setting in next.config.js](#add-setting-in-nextconfigjs)
  - [Create docker-compose.production.yml](#create-docker-composeproductionyml)
  - [Build/Run Docker Contrainer](#buildrun-docker-contrainer)
  - [Source](#source)

## Create Next-app
```
yarn create next-app next-docker-demo
cd next-docker-demo
```

## Create Dev Dockerfile
```Dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package.json yarn.lock ./

RUN yarn install

COPY next.config.js ./next.config.js

COPY pages ./pages
COPY public ./public
COPY styles ./styles

CMD ["yarn","dev"]
```

## Create Dev Docker-compose.yml
```yml
version: '3'

services:
  app:
    image: nextjs-docker-dev
    build: .
    ports:
      - 3000:3000
    volumes:
      - ./pages:/app/pages
      - ./public:/app/public
      - ./styles:/app/styles
```

## Build/Run dev Docker Container
```bash
docker-compose up --build --force-recreate
```

## Create production Dockerfile
```Dockerfile
# Step1
FROM node:16-alpine AS deps

ENV NODE_ENV=production

RUN apk add --no-cache libc6-compat

WORKDIR /app

COPY package.json yarn.lock ./
RUN yarn install --frozen-lockfile

# Step2
FROM node:16-alpine AS builder

ENV NODE_ENV=production

WORKDIR /app

COPY next.config.js ./
COPY package.json yarn.lock ./
COPY --from=deps /app/node_modules ./node_modules

COPY pages ./pages
COPY public ./public
COPY styles ./styles

RUN yarn build

# Step3
FROM node:16-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

CMD ["node","server.js"]

```

## Add Setting in next.config.js
```js
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  experimental: {
    outputStandalone: true,
  },
};

```

## Create docker-compose.production.yml
```yml
version: '3'

services:
  app:
    image: nextjs-docker-production
    
    build: 
      context: ./
      dockerfile: Dockerfile.production
    
    ports:
      - 3000:3000

```

## Build/Run Docker Contrainer
```bash
docker-compose -f docker-compose.production.yml up --build --force-recreate
```

<hr>

## Source
[youtube](https://www.youtube.com/watch?v=aNh8iShFXto)


