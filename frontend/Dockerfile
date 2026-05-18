# ========== ETAPA 1: Preparación ==========
FROM alpine AS builder
WORKDIR /app
# Copiamos todo el código del frontend
COPY . .
# Borramos archivos que no necesitamos servir
RUN rm -f Dockerfile nginx.conf.template

# ========== ETAPA 2: Producción (Nginx Seguro) ==========
# Usamos una imagen de Nginx que NO requiere permisos de administrador (non-root)
FROM nginxinc/nginx-unprivileged:alpine

# Copiamos los archivos de la web
COPY --from=builder /app /usr/share/nginx/html

# Copiamos nuestra plantilla de configuración del proxy
COPY nginx.conf.template /etc/nginx/templates/default.conf.template

# Exponemos el puerto 8080 
EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]