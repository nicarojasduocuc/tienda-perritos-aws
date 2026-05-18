# Usamos una imagen base ligera de Node.js
FROM node:18-alpine AS builder

# Creamos el directorio de trabajo dentro del contenedor
WORKDIR /app

# Copiamos solo los archivos de dependencias primero 
COPY package*.json ./

# Instalamos las dependencias
RUN npm install

# Copiamos el resto del código del backend
COPY . .

# ========== ETAPA 2: Producción ==========
# Volvemos a usar una imagen limpia y ligera para el producto final
FROM node:18-alpine

WORKDIR /app

# Copiamos solo lo que necesitamos de la ETAPA 1
COPY --from=builder /app .

# Por seguridad, cambiamos a un usuario sin privilegios de administrador (non-root)
USER node

# Exponemos el puerto que usa nuestra API
EXPOSE 3001

# Comando para iniciar el servidor
CMD ["node", "server.js"]