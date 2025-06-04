FROM node:16-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

ENV MONGO_CONN_STR="mongodb://username:password@host:port/database-name" \
    MONGO_USERNAME="" \
    MONGO_PASSWORD="" 

EXPOSE 3500

CMD ["node", "index.js"]
