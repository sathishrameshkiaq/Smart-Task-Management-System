FROM node:24-alpine AS builder
WORKDIR /app
COPY package*.json ./
COPY src ./src
RUN npm install
EXPOSE 5002
CMD ["node", "src/server.js"]
