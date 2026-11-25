# Taller Docker - entrega

Integrantes: **Alvaro Sneider Portillo Mora**, **Juan David Llanos Castañeda**.

Repositorio público. Colaboradores: ninguno (no se añadieron cuentas de IA ni co-colaboradores).

## Contenido

- `catalogo/Dockerfile` - build Maven/JDK 25 + JRE 25, expone 8081.
- `catalogo/.dockerignore`
- `pedidos/Dockerfile` - build Maven/JDK 25 + JRE 25, expone 8082.
- `pedidos/.dockerignore`
- `compose.yaml` - tres servicios: `catalogo-db` (PostgreSQL 16 + volumen), `catalogo` y `pedidos`.
- `DOCUMENTO.md` - cinco respuestas del paso 11.
- `evidencias/` - capturas PNG y logs exactos.

## Ejecución

```powershell
cd taller-docker
docker compose up --build
```

Servicios: Catálogo `http://localhost:8081`, Pedidos `http://localhost:8082`.

Ejemplo de pedido (en PowerShell usar `curl.exe`, no `curl`):

```powershell
'{"productoId":1,"cantidad":2}' | Set-Content -NoNewline pedido.json -Encoding ASCII
curl.exe -s -i -X POST http://localhost:8082/pedidos -H "Content-Type: application/json" -d @pedido.json
```

## Evidencias

- `evidencias/00_compose_ps_dos_servicios.png` - `docker compose ps catalogo pedidos`, los dos servicios pedidos.
- `evidencias/01_compose_ps.png` - compose completo (incluye `catalogo-db` del reto PostgreSQL).
- `evidencias/02_pedido_201.png` - pedido creado con HTTP 201.
- `evidencias/03_503_catalogo_detenido.png` - HTTP 503 con Catálogo detenido.
- `evidencias/04_postgres_persistencia.png` - producto creado que sobrevive al reinicio (reto opcional).
- `evidencias/evidencias.txt` - comandos y salidas originales.
