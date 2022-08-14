ARG MYSQL_VER

FROM mysql:${MYSQL_VER}

RUN set -eux; \
	microdnf install -y \
		make \
	; \
	microdnf clean all

COPY bin /usr/local/bin