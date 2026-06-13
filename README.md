<h1 align="center">dockerfile.mq</h1>

A Dockerfile parser implemented as an [mq](https://github.com/harehare/mq) module.

## Features

- Parse all standard Dockerfile instructions
- Multi-stage build support (`FROM ... AS`)
- Line continuation (`\`) handling
- Convenience helpers: `dockerfile_stages`, `dockerfile_ports`, `dockerfile_env`

## Supported Instructions

`FROM` `RUN` `CMD` `LABEL` `EXPOSE` `ENV` `ADD` `COPY` `ENTRYPOINT` `VOLUME` `USER` `WORKDIR` `ARG` `ONBUILD` `STOPSIGNAL` `HEALTHCHECK` `SHELL`

## Installation

```sh
cp dockerfile.mq ~/.local/mq/config/
```

### HTTP Import

```sh
mq -I raw 'import "github.com/harehare/dockerfile.mq" | dockerfile::dockerfile_parse(.)' Dockerfile
```

## API

| Function | Description |
|---|---|
| `dockerfile_parse(input)` | Parse a Dockerfile and return an array of instruction dicts |
| `dockerfile_stages(input)` | Return all `FROM` instructions (multi-stage) |
| `dockerfile_ports(input)` | Return all exposed ports as a flat string array |
| `dockerfile_env(input)` | Return all `ENV` instructions as a dict |

## Example

Given `Dockerfile`:

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

```sh
# List all stages (multi-stage build)
mq -I raw 'import "dockerfile" | dockerfile::dockerfile_stages(.) | map(fn(s): s["image"] + " AS " + s["alias"];)' Dockerfile
# => ["node:20-alpine AS builder", "node:20-alpine AS None"]

# Get exposed ports
mq -I raw 'import "dockerfile" | dockerfile::dockerfile_ports(.)' Dockerfile
# => ["3000"]

# Get environment variables
mq -I raw 'import "dockerfile" | dockerfile::dockerfile_env(.)' Dockerfile
# => {"NODE_ENV": "production"}

# List all RUN commands
mq -I raw 'import "dockerfile" | dockerfile::dockerfile_parse(.) | filter(fn(i): i["instruction"] == "RUN";) | map(fn(i): i["args"];)' Dockerfile
# => ["npm ci", "npm run build"]

# Check base image
mq -I raw 'import "dockerfile" | dockerfile::dockerfile_stages(.) | first() | ."image"' Dockerfile
# => "node:20-alpine"
```

## License

MIT
