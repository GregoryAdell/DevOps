FROM postgres:14.1-alpine

ENV POSTGRES_DB=db
ENV POSTGRES_USER=user
ENV POSTGRES_PASSWORD=pwd

# Copier les scripts SQL d'initialisation
COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/
COPY 02-InsertData.sql /docker-entrypoint-initdb.d/