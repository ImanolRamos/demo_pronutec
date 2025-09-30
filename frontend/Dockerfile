# Stage 1: Build
FROM node:22-alpine AS build
WORKDIR /app

# Copiar package.json y package-lock.json
COPY package*.json ./
RUN npm ci

# Copiar el resto del código
COPY . .

# Construir la app para producción
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:stable-alpine
# Copiar build de Vue a la carpeta de Nginx
COPY --from=build /app/dist /usr/share/nginx/html

# Copiar configuración de Nginx personalizada
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Montar certificados (ya se hace con docker-compose)
# Vuelve a exponer el puerto 443
EXPOSE 443

CMD ["nginx", "-g", "daemon off;"]
