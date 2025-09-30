# Etapa 1: build del frontend
FROM node:24 AS build

WORKDIR /app

# Copiar package.json y package-lock.json
COPY package*.json ./

# Instalar dependencias
RUN npm install

# Copiar todo el código fuente
COPY . .

# Generar build de producción
RUN npm run build

# Etapa 2: Nginx
FROM nginx:stable-alpine

# Copiar el build del frontend a Nginx
COPY --from=build /app/dist /usr/share/nginx/html

# Copiar configuración personalizada de Nginx
# Nota: aquí debe ser un server block, no incluir worker_processes ni http
COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf

# Exponer puerto 443
EXPOSE 443

# Comando por defecto
CMD ["nginx", "-g", "daemon off;"]
