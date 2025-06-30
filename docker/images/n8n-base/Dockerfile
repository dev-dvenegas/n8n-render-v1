ARG NODE_VERSION=22

# ─────────────────────────────────────────────────────────
# STAGE 1 · Builder – instala dependencias del sistema + Python libs
# ─────────────────────────────────────────────────────────
FROM node:${NODE_VERSION}-alpine AS builder

# 1) Fuentes MS
RUN apk --no-cache add --virtual .build-deps-fonts msttcorefonts-installer fontconfig \
 && update-ms-fonts && fc-cache -f \
 && apk del .build-deps-fonts \
 && find /usr/share/fonts/truetype/msttcorefonts/ -type l -exec unlink {} \;

# 2) Paquetes esenciales de n8n + librerías para rembg ‹NEW›
RUN apk add --no-cache \
        git openssh graphicsmagick tini tzdata ca-certificates libc6-compat jq \
        python3 py3-pip py3-setuptools py3-wheel \
        libgcc libstdc++ libgomp             # runtimes necesarios para onnxruntime

# 3) npm + full-icu
RUN npm install -g full-icu@1.5.0 npm@11.4.2

# 4) Instala Pillow y rembg dentro de la propia imagen ‹NEW›
RUN pip install --no-cache-dir \
        pillow==10.3.0 \
        rembg==2.0.*

# Limpieza
RUN rm -rf /tmp/* /root/.npm /root/.cache/node /opt/yarn* \
 && apk del apk-tools

# ─────────────────────────────────────────────────────────
# STAGE 2 · Imagen final de n8n ya con Python + rembg + Pillow
# ─────────────────────────────────────────────────────────
FROM node:${NODE_VERSION}-alpine

# Copia todo lo que instalamos en el builder
COPY --from=builder / /

WORKDIR /home/node

# Para ICU de n8n
ENV NODE_ICU_DATA=/usr/local/lib/node_modules/full-icu

# ‹NEW› Haz que n8n use este intérprete de Python
ENV N8N_PYTHON_BINARY=/usr/bin/python3

EXPOSE 5678
