# Frontend Dockerfile (Nginx sirviendo estáticos y haciendo proxy a /api)
FROM nginx:1.27-alpine

# Copiamos archivos estáticos
COPY index.html /usr/share/nginx/html/index.html

# Configuramos Nginx para proxy /api -> backend
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
