# Paso 11 - Preguntas de análisis

Integrantes: Alvaro Sneider Portillo Mora, Juan David Llanos Castañeda.

## 1. ¿Por qué Pedidos no puede usar `localhost:8081` dentro de su contenedor? ¿Qué lo reemplazó?

`localhost` siempre apunta al propio contenedor. Dentro de `pedidos`, `localhost:8081` sería el puerto 8081 del contenedor de Pedidos, donde no hay ningún servicio de Catálogo. Lo reemplazó el nombre del servicio de Docker Compose: `http://catalogo:8081`. Docker resuelve ese nombre mediante su red interna y envía la petición al contenedor correcto. En `compose.yaml` se pasa como variable `CATALOGO_URL: http://catalogo:8081`.

## 2. ¿Qué ventaja tiene poner la dirección de Catálogo en `application.properties` y no en el código Java?

La dirección queda como configuración externa, no como literal compiled. Se puede cambiar según el entorno (local, Docker, pruebas) sin modificar ni recompilar el código. Spring Boot permite sobreescribirla con variables de entorno: `CATALOGO_URL` se enlaza automáticamente con la propiedad `catalogo.url`. El mismo JAR sirve para los tres escenarios.

## 3. ¿Por qué el Dockerfile tiene dos etapas? ¿Qué pasaría con el tamaño si fuera una sola?

La primera etapa usa una imagen con Maven y JDK 25 para compilar y empaquetar; la segunda copia solo el JAR ejecutable sobre una imagen JRE 25. La etapa de compilación se descarta. Con una sola etapa, la imagen final arrastraría Maven, el JDK completo, el código fuente, el `target` y la caché de dependencias: sería mucho más grande y tendría más superficie de ataque.

## 4. ¿Qué pasó con el producto creado después de reiniciar Catálogo? ¿Por qué? ¿Qué se necesita para que no se pierda?

Con la base H2 en memoria (`jdbc:h2:mem:catalogo`), el producto desaparece al reiniciar: los datos viven solo en la memoria del proceso y el contenedor arranca limpio. Para conservarlo se necesita almacenamiento persistente. En el reto opcional se usó PostgreSQL 16 con un volumen nombrado `catalogo-datos` montado en `/var/lib/postgresql/data`; así el producto con id 4 siguió presente después de `docker compose restart catalogo` (ver `evidencias/04_postgres_persistencia.png`).

## 5. ¿`depends_on` garantiza que Catálogo esté listo para responder?

No. `depends_on` solo ordena el arranque de contenedores; no espera a que la aplicación esté escuchando. Pedidos puede arrancar antes de que Catálogo termine de inicializar Spring Boot y recibir un primer pedido con error. Para acercarse a una garantía real se usan healthchecks con `condition: service_healthy`, reintentos del cliente o un script de espera. En este trabajo se añadió healthcheck a PostgreSQL (`pg_isready`) y Pedidos devuelve 503 controlado si Catálogo no responde.
