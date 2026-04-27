Para este proyecto de tienda integral (venta de autos, refacciones y servicios), el modelo de base de datos más eficiente es uno de tipo NoSQL orientado a documentos (como **Firebase Firestore** o **MongoDB**), ya que permite una gran flexibilidad para manejar productos con atributos muy distintos entre sí.

A continuación, presento un ejemplo de la estructura de colecciones, sus documentos y los tipos de datos sugeridos para cada atributo.

---

### 1. Colección: `vehiculos`
Esta colección almacena el inventario de autos en exhibición.

| Atributo | Tipo de Dato | Descripción |
| :--- | :--- | :--- |
| `id_vehiculo` | String (UID) | Identificador único del vehículo. |
| `marca` | String | Marca del auto (ej. Toyota, Nissan). |
| `modelo` | String | Submarca o modelo (ej. Corolla, Sentra). |
| `anio` | Number (Int) | Año de fabricación del vehículo. |
| `precio` | Number (Double) | Precio de venta al público. |
| `kilometraje` | Number (Int) | Recorrido total del auto. |
| `tipo_transmision` | String | Automático, Manual o CVT. |
| `estado` | String | "Nuevo" o "Seminuevo". |
| `fotos` | Array (Strings) | Lista de URLs de las imágenes en el storage. |
| `disponible` | Boolean | Indica si el auto sigue en venta. |

### 2. Colección: `refacciones`
Enfocada en piezas mecánicas y componentes específicos.

| Atributo | Tipo de Dato | Descripción |
| :--- | :--- | :--- |
| `id_parte` | String (UID) | Código interno o SKU. |
| `nombre` | String | Nombre de la pieza (ej. Pastillas de freno). |
| `categoria` | String | Frenos, Motor, Suspensión, Eléctrico. |
| `num_parte_fabricante`| String | Código original de la marca. |
| `compatibilidad` | Array (Strings) | Lista de modelos/años compatibles. |
| `precio_venta` | Number (Double) | Precio final al cliente. |
| `stock_actual` | Number (Int) | Cantidad disponible en almacén. |
| `stock_minimo` | Number (Int) | Alerta para reordenar producto. |

### 3. Colección: `insumos_aceites`
Estructura específica para productos químicos y líquidos de alta rotación.

| Atributo | Tipo de Dato | Descripción |
| :--- | :--- | :--- |
| `id_insumo` | String (UID) | Identificador único. |
| `tipo` | String | Aceite, Anticongelante, Líquido de frenos. |
| `marca` | String | Marca del fabricante (ej. Mobil, Castrol). |
| `viscosidad` | String | Solo para aceites (ej. 5W-30, 10W-40). |
| `presentacion` | String | Volumen del envase (ej. "946ml", "5L"). |
| `precio` | Number (Double) | Costo por unidad. |
| `stock` | Number (Int) | Cantidad de botellas/galones disponibles. |



### 4. Colección: `clientes`
Registro de personas físicas o flotas de transporte.

| Atributo | Tipo de Dato | Descripción |
| :--- | :--- | :--- |
| `id_cliente` | String (UID) | Identificador del cliente. |
| `nombre_completo` | String | Nombre o Razón Social. |
| `telefono` | String | Número de contacto. |
| `email` | String | Correo para facturación y seguimiento. |
| `es_flotilla` | Boolean | Define si el cliente tiene precios especiales. |
| `historial_compras` | Array (References) | Enlaces a los IDs de la colección `ventas`. |

### 5. Colección: `servicios_taller`
Para el control del área de mantenimiento preventivo y correctivo.

| Atributo | Tipo de Dato | Descripción |
| :--- | :--- | :--- |
| `id_servicio` | String (UID) | Folio de la orden de servicio. |
| `cliente_id` | Reference | Vínculo con el documento en la colección `clientes`. |
| `descripcion` | String | Detalle del trabajo (ej. Afinación mayor). |
| `fecha_ingreso` | Timestamp | Fecha y hora de recepción. |
| `mano_obra_costo` | Number (Double) | Cargo por el trabajo técnico. |
| `refacciones_usadas` | Array (Maps) | Lista de IDs de piezas usadas y su cantidad. |
| `estatus` | String | "En proceso", "Terminado", "Entregado". |

### 6. Colección: `ventas`
Registro transaccional de la tienda.

| Atributo | Tipo de Dato | Descripción |
| :--- | :--- | :--- |
| `id_transaccion` | String (UID) | Folio de venta o factura. |
| `fecha_venta` | Timestamp | Momento exacto de la operación. |
| `items` | Array (Maps) | Lista de productos (id, cantidad, subtotal). |
| `total` | Number (Double) | Monto final cobrado. |
| `metodo_pago` | String | Efectivo, Tarjeta, Transferencia. |

Esta estructura permite realizar consultas rápidas (por ejemplo, filtrar aceites por viscosidad o buscar refacciones compatibles con un modelo específico) y es escalable si en el futuro decides añadir una aplicación móvil para que los clientes agenden citas o compren en línea.
