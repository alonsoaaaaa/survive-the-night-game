FROM node:20-alpine as builder

ARG VITE_WSS_URL
ENV VITE_WSS_URL=$VITE_WSS_URL

WORKDIR /app

COPY package*.json ./
COPY packages/website/package*.json ./packages/website/
COPY packages/game-client/package*.json ./packages/game-client/
COPY packages/game-shared/package*.json ./packages/game-shared/

RUN npm ci

COPY packages/game-shared ./packages/game-shared
COPY packages/game-client ./packages/game-client
COPY packages/website ./packages/website
COPY tsconfig.json ./

RUN npm run build --workspace=packages/website

FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
COPY packages/website/package.json ./packages/website/

COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/packages/website/build ./packages/website/build
COPY --from=builder /app/packages/website/package.json ./packages/website/package.json

WORKDIR /app/packages/website

CMD ["npm", "run", "start"]