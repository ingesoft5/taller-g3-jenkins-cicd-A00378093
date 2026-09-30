# =============================================================================
# Taller Evaluativo 2 - Fase 2: Empaquetamiento Inmutable Multi-Stage
# Frontend: studytrack-frontend
# =============================================================================

# ---------- Stage 1: Builder ----------
FROM node:20-alpine AS builder
WORKDIR /app

# Dependencias primero (capa cacheable)
COPY package*.json ./
RUN npm install --no-audit --no-fund

COPY . .
# Vite "hornea" la URL del API dentro del bundle en tiempo de build.
# El navegador del usuario llama al backend publicado en el host (puerto 8080).
ARG VITE_API_URL=http://localhost:8080
ENV VITE_API_URL=${VITE_API_URL}
RUN npm run build

# ---------- Stage 2: Runtime ----------
FROM nginx:alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html

# Nginx sin root: el puerto 3000 (>1024) no requiere privilegios.
RUN chown -R nginx:nginx /var/cache/nginx /usr/share/nginx/html \
 && touch /var/run/nginx.pid && chown nginx:nginx /var/run/nginx.pid
USER nginx

EXPOSE 3000
CMD ["nginx", "-g", "daemon off;"]
