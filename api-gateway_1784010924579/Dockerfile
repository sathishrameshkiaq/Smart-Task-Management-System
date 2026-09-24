 	# Builder Stage
FROM node:24-alpine 

WORKDIR /app

COPY package*.json ./

RUN npm ci --ignore-scripts

COPY src ./src

EXPOSE 5000

ENTRYPOINT ["node", "src/server.js"]
