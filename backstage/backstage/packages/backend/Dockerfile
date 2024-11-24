FROM node:18-bookworm-slim

# Ajustar permissões no cache do Corepack e Yarn
RUN mkdir -p /home/node/.cache/node/corepack/v1 && \
    chown -R node:node /home/node/.cache

# Criar diretório da aplicação com permissões adequadas
RUN mkdir -p /app && chown -R node:node /app

# Configurar cache alternativo para Yarn
RUN yarn config set cache-folder /tmp/.yarn-cache

# Instalar dependências do isolate-vm e sqlite3 (depois de garantir permissões adequadas)
USER root
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential libsqlite3-dev && \
    yarn config set python /usr/bin/python3

# Ativar Corepack e preparar versão específica do Yarn
RUN corepack enable && corepack prepare yarn@4.4.1 --activate

# Garantir permissões apropriadas para a pasta /app
RUN chown -R node:node /app

# Definir diretório de trabalho
WORKDIR /app

# Copiar arquivos do projeto para o contêiner
COPY yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./

# Extrair e limpar arquivos de skeleton
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

# Instalar dependências (usando Yarn moderno) com cache para evitar downloads repetidos
USER node
RUN --mount=type=cache,target=/home/node/.cache/yarn,sharing=locked \
    yarn workspaces focus --production

# Copiar o restante dos arquivos necessários (com permissões adequadas)
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./

# Garantir que todos os diretórios de `/app/packages/backend` existam antes de tentar
USER root
RUN mkdir -p /app/packages/backend /app/packages/backend/dist && \
    chown -R node:node /app/packages/backend

# Extrair e limpar o bundle
RUN tar xzf bundle.tar.gz -C /app && rm bundle.tar.gz

# Configurar o modo de produção
ENV NODE_ENV=production

# Comando para iniciar o backend
CMD ["node", "packages/backend", "--config", "app-config.yaml"]
