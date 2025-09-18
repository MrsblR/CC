# Database Dockerfile (PostgreSQL)
FROM postgres:16-alpine

# Variables de entorno para la DB (se pueden sobreescribir en docker-compose)
ENV POSTGRES_DB=appdb
ENV POSTGRES_USER=appuser
ENV POSTGRES_PASSWORD=apppassword

# Scripts de inicialización (si quisieras datos iniciales)
# COPY init.sql /docker-entrypoint-initdb.d/

EXPOSE 5432
