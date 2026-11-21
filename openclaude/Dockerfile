# OpenClaude needs Node.js 24.22.1 or newer
FROM node:24-slim

RUN apt-get update \
 && apt-get install -y --no-install-recommends git ripgrep ca-certificates \
 && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /home/node/.openclaude \                                                                                 
   && chown -R node:node /home/node/.openclaude \                                                                       
   && chmod 755 /home/node/.openclaude 

RUN npm install -g @twigpine/openclaude@latest

# The node image already has a non-root "node" user
USER node
WORKDIR /workspace

ENTRYPOINT ["openclaude"]
