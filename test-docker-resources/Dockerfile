FROM node:14

WORKDIR /src

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000
ENTRYPOINT [ "node", "index.js" ]