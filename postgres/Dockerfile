FROM postgres:14

ENV POSTGRES_USER=root
ENV POSTGRES_PASSWORD=h5UzM2kEnp8Q42
ENV APP_DB_USER=alura
ENV APP_DB_PASS=nJ6vsW37rCd1v88
ENV APP_DB_NAME=logsdb

COPY 01-init.sh /docker-entrypoint-initdb.d/
COPY data /var/lib/postgresql/

EXPOSE 5432
