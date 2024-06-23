# syntax=docker/dockerfile:1
FROM node:22-alpine3.19
WORKDIR /app
COPY package*.json ./
COPY . .
RUN npm install
EXPOSE 5173
CMD ["npm", "run", "dev"]
