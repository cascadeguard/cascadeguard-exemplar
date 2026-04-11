# Example: deprecated base image (openjdk is end-of-life)
#
# openjdk was deprecated on Docker Hub in November 2022. The official
# successor is eclipse-temurin. Running CascadeGuard scan against this
# Dockerfile produces a DEPRECATED finding:
#
#   DEPRECATED  openjdk:17-alpine
#     The 'openjdk' image is deprecated and no longer receives security updates.
#     Recommended replacement: eclipse-temurin:17-alpine
#
# See: https://hub.docker.com/_/openjdk (deprecation notice)
#      https://hub.docker.com/_/eclipse-temurin (recommended replacement)

FROM openjdk:17-alpine

WORKDIR /app

# Simple "hello world" jar would go here in a real service.
# This file intentionally left minimal — it is a deprecation-detection exemplar,
# not a production image.
COPY README.md /app/

CMD ["java", "-version"]
