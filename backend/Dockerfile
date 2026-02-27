#stage 1
FROM node:21 AS backend-builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . . 


#Stage 2
FROM node:21-slim

WORKDIR /app

COPY --from=backend-builder /app .


EXPOSE 8080

CMD ["npm", "start"]

