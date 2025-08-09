FROM node:20-alpine

RUN apk add --no-cache openssl

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm install

COPY . .

RUN npm run generate

RUN npm run test

RUN npm run build

EXPOSE 8080

CMD ["npm", "start"]