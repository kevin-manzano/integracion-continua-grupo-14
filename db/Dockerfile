FROM postgres:16-alpine
# Se copian (no se montan) para que funcione también cuando docker compose corre dentro de un contenedor, como en Jenkins
COPY init/ /docker-entrypoint-initdb.d/
