# ---- Estágio 1: Builder ----
# Este estágio instala todas as dependências (dev e prod),
# roda testes e constrói a aplicação.
FROM node:20-alpine AS builder

RUN apk add --no-cache openssl

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .

RUN npm run generate
RUN npm run test
RUN npm run build

# ---- Estágio 2: Produção ----
# Este estágio cria a imagem final, leve e otimizada.
FROM node:20-alpine

# Instala apenas as dependências do SO necessárias para rodar a aplicação
RUN apk add --no-cache openssl

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install --omit=dev

COPY --from=builder /app/dist ./dist

EXPOSE 8080

CMD ["npm", "start"]
