# Clase de Roles y permisos

En esta actividad se ejecutó la migración de **Spatie** para crear las tablas intermedias en MySQL, como `roles`, `permissions` y `model_has_roles`, que permiten relacionar a los usuarios con sus funciones y permisos dentro del sistema.

- **Con `RolePermissionSeeder`:** se automatiza la creación de roles, por ejemplo, `admin` y `vendedor`. Además, se les asignan permisos como `crear-productos`, `editar-productos` y `eliminar-productos`.

- **En los controladores de Producto y Categoría:** se agregaron reglas de middleware. Esto permite controlar el acceso a determinadas rutas. Por ejemplo, si un usuario intenta entrar a una ruta de eliminación sin tener los permisos necesarios, el acceso se detiene.

- **Migración de Soft Deletes:** se realizó una migración para agregar la columna `deleted_at` (de tipo *timestamp* y que permite valores nulos) a las tablas `productos` y `categorias`.

- **Implementación de Soft Deletes en los modelos:** se agregó el trait `SoftDeletes` en los modelos Producto y Categoría, importando la clase `Illuminate\Database\Eloquent\SoftDeletes`. Esto le permite al ORM de Laravel manejar las eliminaciones de forma lógica. Cuando un usuario elimina un registro desde la vista, este desaparece de la interfaz, pero se conserva en la base de datos, lo que permite mantener la trazabilidad de la información.


# Clase 10
# Módulo de Proveedores

Se creó la tabla `providers` con campos como tipo y número de documento (con validación de unicidad), razón social, persona de contacto, correo, teléfono, dirección y estado activo. Además, se integró `SoftDeletes` para no perder información histórica.

Se configuró el controlador de proveedores con validaciones de seguridad mediante **Form Requests** (`StoreProviderRequest` y `UpdateProviderRequest`).

Como regla de negocio, se implementó una validación estricta para que el sistema no permita eliminar un proveedor si ya tiene órdenes de compra asociadas.

# Módulo de Compras

Se compone de la tabla `purchases`, con un número secuencial automático (ej. `OC-00001`), el proveedor, el usuario que registra la compra y las fechas. También incluye una tabla de detalle (`purchase_details`) con los productos pedidos, sus costos unitarios y cantidades.

Al crear una orden de compra, el estado queda en **pendiente** y el inventario no cambia. El stock de los productos solo aumenta cuando se registra la recepción total o parcial de la mercancía.

Todo el proceso de recepción se ejecuta dentro de una transacción de `DB::transaction`, con bloqueo de registros mediante `lockForUpdate`, para evitar conflictos si dos personas intentan recibir la misma orden al mismo tiempo.

## Reglas de negocio

- No se puede recibir una cantidad mayor a la que se pidió.
- Cada recepción debe incluir al menos una unidad válida.
- Solo se pueden anular órdenes de compra que sigan en estado pendiente. Si ya ingresó mercancía al inventario, la orden no se puede anular.

# Generación de Reportes en PDF

Se diseñó un diseño general (`pdf/layout.blade.php`), adaptado a las limitaciones de DomPDF, utilizando CSS clásico y tablas en lugar de Tailwind, Flexbox o Grid. Incluye membrete con la identidad de la empresa, márgenes, paginación y pie de página automático.

Se generan dos documentos PDF:

- **PDF de la orden de compra:** para enviar a los proveedores.
- **PDF de la factura de venta:** para entregar a los clientes.

# Nuevos KPI

Se agregaron contadores y valores monetarios para mostrar:

- Las compras del mes.
- Las órdenes pendientes por recibir.
- El total de proveedores activos.
- Un indicador sencillo de ventas menos compras del mes.

También se integró una alerta que detecta automáticamente los productos activos con stock bajo (5 unidades) y les coloca un enlace directo con la frase **«+ Crear orden de compra»**, haciendo que el panel sea más operativo.

# Roles y Permisos

- **Administrador:** tiene control total sobre la creación, visualización, generación de reportes y anulaciones.
- **Almacenista:** puede consultar productos, crear y gestionar proveedores, registrar las recepciones de mercancía en bodega y recibir compras. Sin embargo, no puede anular compras definitivas ni consultar reportes financieros avanzados.
- **Vendedor:** se enfoca en la atención y facturación a clientes, generando sus respectivos comprobantes en PDF.