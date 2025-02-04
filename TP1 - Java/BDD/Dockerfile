FROM postgres:14.1-alpine

# Copier les scripts SQL d'initialisation
COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/
COPY 02-InsertData.sql /docker-entrypoint-initdb.d/