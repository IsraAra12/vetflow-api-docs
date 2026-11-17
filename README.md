# API de lectura de VetFlow

Consulta el catálogo de productos y los servicios de tu veterinaria con una clave API. La URL base es `https://vetflowplus.pages.dev/api/v1`.

## Obtener una clave

1. Inicia sesión en VetFlow con una cuenta de administrador de tu veterinaria.
2. En **Configuración → Claves API → Abrir generador de claves API**, pulsa **Crear clave API**.
3. Selecciona **Catálogo completo de productos e inventario** y **Servicios, precios y estado activo/inactivo** según lo que necesite tu integración.
4. Elige la vigencia, confirma tu contraseña y guarda la clave. VetFlow muestra el valor completo una sola vez.

Envía la clave en la cabecera `X-API-Key`. La clave determina la veterinaria y los datos que puede consultar. Las claves con permiso `inventario` ya sirven para consultar el catálogo. Para leer servicios, genera una clave que incluya `servicios`.

## Rutas de consulta

| Ruta GET | Permiso | Datos |
| --- | --- | --- |
| `/catalogo` | `inventario` | Catálogo completo de productos, incluidos los que tienen stock cero o no controlan existencias. |
| `/inventario` | `inventario` | Los mismos productos y campos de `/catalogo`. |
| `/servicios` | `servicios` | Servicios de la veterinaria, incluidos los inactivos. |

**Productos.** Cada registro incluye `id`, `clinic_id`, `product_name`, `category`, `sku`, `stock_quantity`, `critical_stock_level`, `sale_price`, `supplier`, `lot_number`, `expiration_date`, `track_stock`, `created_at` y `updated_at`.

**Servicios.** Cada registro incluye `id`, `clinic_id`, `name`, `category`, `price`, `is_active`, `created_at` y `updated_at`.

`track_stock` e `is_active` son booleanos. Los campos opcionales sin valor se devuelven como `null`.

## Hacer una consulta

Configura `VETFLOW_API_KEY` como variable de entorno con el valor de tu clave. Estos comandos usan Bash.

```bash
curl --fail-with-body 'https://vetflowplus.pages.dev/api/v1/catalogo?limit=100' \
  -H "X-API-Key: $VETFLOW_API_KEY"

curl --fail-with-body 'https://vetflowplus.pages.dev/api/v1/servicios?limit=100' \
  -H "X-API-Key: $VETFLOW_API_KEY"
```

## Extraer todos los registros

Cada respuesta contiene `data`, `has_more`, `next_cursor` y `limit`. `data` contiene hasta 100 registros ordenados por `id`.

1. Consulta la ruta con `limit=100` y guarda los registros de `data`.
2. Si `has_more` es `true`, vuelve a consultar la misma ruta con `cursor` igual al `next_cursor` recibido.
3. Agrega los registros de cada página hasta que `has_more` sea `false`. Mantén un cursor independiente para productos y servicios.

La consulta lee los datos vigentes al momento de cada página. Si editas el catálogo durante una extracción, las páginas siguientes pueden reflejar esas ediciones.

## Códigos de respuesta

| Código | Significado |
| --- | --- |
| `200` | Consulta correcta. `data` puede estar vacío. |
| `401` | La clave falta, es inválida, venció o fue revocada. |
| `403` | La clave no tiene permiso para consultar ese recurso. |
| `422` | El recurso o los parámetros de la consulta son inválidos. `limit` admite valores de 1 a 100. |
| `429` | Se alcanzó el límite de 60 consultas por minuto y por clave. Espera los segundos indicados en `Retry-After` antes de reintentar. |
