# Etapa 1: build del frontend
FROM node:24 AS build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# etapa de producción
FROM nginx:1.13.12-alpine as production-stage
COPY ./nginx/nginx.conf /etc/nginx/nginx.conf
COPY --from=build-stage /app/dist /usr/share/nginx/html
EXPOSE 443
