---
title: "Configurando Kong Gateway sin Base de Datos (DB-less)"
description: "Guía paso a paso para configurar Kong Gateway sin base de datos (DB-less)"
pubDate: "2024-06-15"
heroImage: "/Kong-API-Gateway.webp"
tags: ["backend", "kong", "kong-gateway", "kong-db-less","docker","microservicios"]
---

Kong Gateway es una herramienta popular de *API Gateway* y gestión de microservicios. Generalmente, Kong se configura con una base de datos como PostgreSQL. Sin embargo, también ofrece un modo "sin base de datos" (*DB-less*), útil para despliegues más simples o entornos donde una base de datos no es necesaria.

En esta guía, aprenderemos a configurar Kong Gateway en modo **DB-less** usando un archivo de configuración YAML.

## Requisitos Previos

- Docker instalado en tu sistema.
- Familiaridad básica con archivos YAML.

## 1. Descargar la Imagen de Docker de Kong

Primero, descargaremos la imagen oficial de Kong desde Docker Hub:

```bash
docker pull kong:latest
```

## 2. Crear el Archivo de Configuración YAML

Para configurar Kong en modo sin base de datos, crearemos un archivo llamado `kong.yaml`.

Este archivo contendrá la configuración de las rutas y servicios.

**Ejemplo de `kong.yaml`:**

```yaml
_format_version: "3.0"

services:
  - name: example-service
    url: https://httpbin.org
    routes:
      - name: example-route
        paths:
          - /example
```

Explicación del archivo:

- **`_format_version`**: La versión del formato de configuración.
- **`services`**: Define los servicios a los que Kong redirigirá el tráfico.
  - **`name`**: Nombre del servicio.
  - **`url`**: La URL del backend al que se redirigirá el tráfico.
  - **`routes`**: Define las rutas de acceso al servicio.
    - **`paths`**: Define los caminos (endpoints) de acceso.

## 3. Crear un Contenedor de Kong en Modo DB-less

Vamos a ejecutar un contenedor de Kong con la configuración DB-less:

```bash
docker run -d --name kong \
  -p 8000:8000 \
  -p 8443:8443 \
  -v $(pwd)/kong.yaml:/usr/local/kong/declarative/kong.yaml \
  -e "KONG_DATABASE=off" \
  -e "KONG_DECLARATIVE_CONFIG=/usr/local/kong/declarative/kong.yaml" \
  kong:latest
```

### Explicación del comando

- **`-p 8000:8000`**: Expone el puerto 8000 para peticiones HTTP.
- **`-p 8443:8443`**: Expone el puerto 8443 para peticiones HTTPS.
- **`-v $(pwd)/kong.yaml:/usr/local/kong/declarative/kong.yaml`**: Monta el archivo `kong.yaml` en el contenedor.
- **`KONG_DATABASE=off`**: Desactiva el uso de base de datos.
- **`KONG_DECLARATIVE_CONFIG`**: Ruta del archivo YAML de configuración declarativa.

## 4. Probar la Configuración

Una vez que el contenedor está en funcionamiento, podemos probar la ruta configurada usando `curl`:

```bash
curl http://localhost:8000/example
```

Deberías recibir una respuesta del servicio `https://httpbin.org`.

## 5. Detener y Eliminar el Contenedor

Cuando termines de probar, puedes detener y eliminar el contenedor con:

```bash
docker stop kong && docker rm kong
```

## Conclusión

Configurar Kong Gateway en modo **DB-less** es una forma eficiente de simplificar tu despliegue, especialmente cuando no se necesita una base de datos externa. Es ideal para entornos de desarrollo o despliegues ligeros.

Ahora puedes extender la configuración de `kong.yaml` para agregar más servicios, rutas y plugins según tus necesidades.

---

**Recursos adicionales:**

- [Documentación oficial de Kong Gateway](https://docs.konghq.com/gateway/latest/introduction/)

---

