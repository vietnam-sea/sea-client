FROM node:20-alpine AS base

FROM base AS builder
RUN apk update && apk add --no-cache libc6-compat
WORKDIR /build
RUN yarn global add turbo@^2
COPY . .
RUN turbo prune @vietnam-sea/sea-client-customer --docker

FROM base AS installer
RUN apk update && apk add --no-cache libc6-compat
WORKDIR /installer

COPY --from=builder /build/out/json/ ./
RUN yarn install --frozen-lockfile
 
COPY --from=builder /build/out/full/ ./
RUN yarn turbo run build
 
FROM base AS runner
WORKDIR /app
 
RUN addgroup --system --gid 1001 vietnamsea
RUN adduser --system --uid 1001 vietnamsea
USER vietnamsea

COPY --from=installer --chown=vietnamsea:vietnamsea /installer/apps/customer/.next/standalone ./
COPY --from=installer --chown=vietnamsea:vietnamsea /installer/apps/customer/.next/static ./apps/customer/.next/static
COPY --from=installer --chown=vietnamsea:vietnamsea /installer/apps/customer/public ./apps/customer/public

ENTRYPOINT [ "node", "apps/customer/server.js" ]