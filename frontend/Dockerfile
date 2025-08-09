# ---- Build Stage ----
# Usa a imagem completa do Node.js para construir a aplicação
FROM node:20-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .

RUN npm run build

# ---- Production Stage ----
# Usa uma imagem Node.js menor e limpa para a imagem final
FROM node:20-alpine

WORKDIR /app

# Copia apenas os arquivos de pacote necessários e instala apenas as dependências de produção
COPY package.json package-lock.json ./
RUN npm install --omit=dev

COPY --from=builder /app/dist ./dist  # Ajuste '/app/dist' para o seu diretório de saída de build

EXPOSE 3000

CMD ["npm", "start"]