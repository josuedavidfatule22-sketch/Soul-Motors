# SOUL MOTORS — Sistema de gestión de taller de motocicletas

**Planificación, asignación de tareas, garantías, proveedores e inversión de roles en 4 Sprints (v0.4)**

| Rol | Integrante | Horas útiles / Sprint (Aprox.) |
| :---- | :---- | :---- |
| **Product Owner** | Josue | 10 h \- 12 h |
| **Scrum Master** | Eimy | 10 h \- 12 h |
| **Base de Datos (PostgreSQL / Neon)** | Leider | 15 h \- 18 h |
| **Backend 1 — Flujo operativo, garantías y despliegue** | **Luisa** | 15 h \- 17 h |
| **Backend 2 — Inventario, proveedores, finanzas y BI** | **Neyer** | 16 h \- 18 h |
| **Frontend (Vue \+ Vercel)** | Jhon | 16 h \- 20 h |

## 1\. Estrategia de trabajo con Roles Invertidos, Garantías, Proveedores y Priorización (MoSCoW)

* **Sustitución de Diagnóstico de Recepción por Sistema de Garantía y Checklist Técnico de Respaldo:**

  * Se elimina la tabla y fase de `diagnostico_recepcion` y se descartan los parámetros volátiles que cambian en cuanto la moto sale del taller (como nivel de gasolina o nivel de aceite).  
  * Se implementa la entidad `garantia_orden` con un **Checklist Técnico de Calidad y Respaldo de Garantía**, cuyos parámetros aportan utilidad real al taller y al cliente:  
    1. **Cobertura de Garantía:** Kilometraje exacto de salida/entrega, vigencia en días (ej. 30, 60, 90 días) o límite de kilómetros cubiertos (ej. 2.000 km), y estado de la garantía (`VIGENTE`, `VENCIDA`, `RECLAMADA`, `ANULADA_POR_MANIPULACION`).  
    2. **Checklist de Sistemas Intervenidos y Seguridad:** Verificación de torque y tornillería crítica, funcionamiento del sistema eléctrico/luces, tensión y holgura de transmisión/frenos, ausencia de fugas en empaques intervenidos, registro de números de serie/lote de piezas instaladas y **Prueba de Ruta Aprobada** antes de entregar.  
* **Leider (Base de Datos — Neon PostgreSQL):** Trabaja un paso adelante al inicio de cada sprint (días 1 y 2\) creando tablas con `created_at` y `updated_at`, catálogo de proveedores con prioridad por pieza (`proveedor`, `pieza_proveedor`), pedidos de suministro (`pedido_proveedor`), tabla de garantías y checklist técnico (`garantia_orden`), funciones transaccionales, triggers (`audit_log`, reverso de stock y trazabilidad de reasignación de proveedor) y vistas SQL en Neon.

* **Luisa (Backend 1 — Flujo operativo, garantías y despliegue):** Asume la configuración de Docker, el despliegue en Render con cron job/auto-ping (`/health`) y la API REST de autenticación, clientes, motocicletas (relación 1 a N), órdenes de trabajo, emisión/validación de garantías y checklist técnico de respaldo (`garantia_orden`), tareas diarias, reasignación por inasistencias de mecánicos y consulta de `audit_log`.

* **Neyer (Backend 2 — Inventario, proveedores, finanzas y BI):** Asume la API REST del catálogo de piezas (repuestos/accesorios), gestión de proveedores y lógica de reasignación automática a proveedor secundario en caso de incumplimiento de entrega, solicitudes y aprobaciones atómicas sobre `detalle_orden`, cálculo de costos, ventas de mostrador con vendedor (`detalle_venta`), pagos (`pago`), caja (`mov_caja`) y endpoints del Dashboard BI.

* **Jhon (Frontend — Vue 3 \+ Vercel):** Diseña los mockups responsivos (móvil, tablet, escritorio) y construye las vistas Vue desplegadas en Vercel durante los Sprints 1 a 3 (incluyendo el modal/certificado de Garantía y Checklist Técnico, y la ventana de gestión y reasignación de proveedores); en el Sprint 4 solo realiza ajustes derivados de pruebas.

* **Josue (Product Owner) y Eimy (Scrum Master):** Lideran las revisiones cada 3 días, gestionan el tablero Kanban, validan mockups y ejecutan la validación funcional continua.

**Regla de prioridad:**

* **Prioridad 1 — Crítica:** Infraestructura en la nube (Neon, Docker, Render, Vercel), autenticación por roles (Admin, Mecánico, Vendedor), clientes y motos, catálogo de piezas, flujo de órdenes de trabajo, sistema de garantía por trabajos realizados con checklist técnico útil, solicitud y aprobación de stock, reverso al cancelar.

* **Prioridad 2 — Alta:** Orden, detalle y pago, gestión de proveedores y ventana de reasignación ante incumplimiento, ventas de mostrador con vendedor, caja, dashboard BI con granularidad, `audit_log`.

* **Prioridad 3 — Media (Recortable):** Tareas diarias fuera de orden, reasignación por inasistencias y vista de auditoría en pantalla.

## 2\. Planificación General de los 4 Sprints (Con Horas Estimadas, Garantías y Proveedores)

### Sprint 1: Base en la nube, accesos, clientes/motos, catálogo de piezas y proveedores base

**Meta:** Tener Neon, el backend en Docker/Render y el frontend en Vercel funcionando de punta a punta, con login por roles y los módulos de clientes, motos, catálogo de piezas y registro de proveedores asociados terminados.

| ID Tarea | Historia / CAM | Tarea específica | Responsable | Prioridad | Depende de | Horas Est. |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| **S1-T01** | CAM-02, 03, 08 | Crear el proyecto en Neon y el script DDL maestro v2: `usuario`, `cliente`, `vehiculo` (1 a N, placa única), `pieza` (stock mayor o igual a 0), `mov_inventario` y `audit_log`, todas con `created_at` y `updated_at`, con trigger genérico de `updated_at`. | **Leider (BD)** | Crítica | Ninguna | **12 h** |
| **S1-T01b** | Extensión BD | Crear en Neon las tablas `proveedor` (datos de contacto y tiempo de entrega) y `pieza_proveedor` (prioridad 1..N y costo pactado) con `created_at` y `updated_at`. | **Leider (BD)** | Alta | S1-T01 | **4 h** |
| **S1-T02** | Base, CAM-08 | Crear el proyecto backend (Java 21, API REST) con conexión a Neon por variables de entorno (SSL) y endpoint `/health`. | **Luisa (BE1)** | Crítica | S1-T01 | **4 h** |
| **S1-T03** | CAM-10, 11 | Escribir el Dockerfile, desplegar el backend en Render y configurar el cron job (cron-job.org) o auto-ping a `/health`. | **Luisa (BE1)** | Crítica | S1-T02 | **4 h** |
| **S1-T04** | CAM-09 | Crear el proyecto Vue, desplegarlo en Vercel, configurar la URL de la API y el layout base responsivo con la paleta del documento de diseño (Tema Oscuro / Claro). | **Jhon (FE)** | Crítica | Ninguna | **3 h** |
| **S1-T05** | CAM-12 | Mockups responsivos (móvil, tablet, escritorio) de Login, Clientes/Motos y Catálogo de Piezas con Proveedores. Validación del PO. | **Jhon (FE) y Josue** | Crítica | Ninguna | **4 h** (Jhon 3h / Josue 1h) |
| **S1-T06** | HU-01 | Implementar el servicio de autenticación (credenciales cifradas, token y rol Admin/Mecánico/Vendedor) y su endpoint de login. | **Luisa (BE1)** | Crítica | S1-T01, S1-T02 | **4 h** |
| **S1-T07** | HU-01, CAM-09 | Construir la vista de Login y el contenedor principal con menú lateral adaptable por rol (menú hamburguesa en móvil). | **Jhon (FE)** | Crítica | S1-T04, S1-T05, S1-T06 | **3 h** |
| **S1-T08** | HU-02, HU-03 | Programar la API de Clientes y Motocicletas (placa única, relación 1 a N). | **Luisa (BE1)** | Crítica | S1-T01, S1-T02 | **5 h** |
| **S1-T09** | HU-02, HU-03 | Construir las vistas de Clientes (búsqueda, tabla, formulario) y del panel de Motocicletas asociadas. | **Jhon (FE)** | Crítica | S1-T07, S1-T08 | **4 h** |
| **S1-T10** | HU-04 | Programar la API del Catálogo de Piezas (Repuesto/Accesorio) y entradas de stock transaccionales con bitácora. | **Neyer (BE2)** | Crítica | S1-T01, S1-T02 | **10 h** |
| **S1-T10b** | Extensión BE2 | Programar endpoints CRUD para el directorio de Proveedores y vinculación con piezas por orden de prioridad (Principal vs. Alternativos). | **Neyer (BE2)** | Alta | S1-T01b, S1-T10 | **4 h** |
| **S1-T11** | HU-04 | Construir las vistas del Catálogo (alerta visual de stock mínimo), pestaña de Proveedores vinculados y modal de entrada de stock. | **Jhon (FE)** | Crítica | S1-T07, S1-T10, S1-T10b | **5 h** |
| **S1-T12** | Control | Revisión del día 3 y cierre del S1: validar el despliegue en la nube, login por roles, clientes/motos, inventario y proveedores base. | **Josue y Eimy** | Crítica | S1-T03, S1-T09, S1-T11 | **8 h** (4h c/u) |

**Total estimado Sprint 1:** 74 horas.

### Sprint 2: Órdenes con detalle, sistema de garantía con checklist técnico, solicitudes de piezas y cancelaciones

**Meta:** Completar el ciclo de vida de la orden de trabajo con el modelo orden/detalle, sistema de garantía por trabajos realizados y checklist técnico de verificación (sin parámetros volátiles como gasolina/aceite), aprobación atómica de piezas, cancelación con reverso automático, vendedor y auditoría.

| ID Tarea | Historia / CAM | Tarea específica | Responsable | Prioridad | Depende de | Horas Est. |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| **S2-T01** | CAM-03, 04, 05, 06 | Crear `orden_trabajo` (con vendedor y orden padre opcional por reclamo de garantía), `garantia_orden` (plazo en días/km, km de entrega, estado de garantía y checklist técnico: torque/ajuste de seguridad, sistema eléctrico, frenos/transmisión, estanqueidad de piezas intervenidas y prueba de ruta) y `detalle_orden` (orden, pieza, cantidad, precio histórico, cubierto por garantía) con `created_at` y `updated_at`. | **Leider (BD)** | Crítica | S1-T01 | **7 h** |
| **S2-T02** | HU-08, 09, CAM-02 | Programar la función SQL de descuento atómico al aprobar piezas, validación automática de vigencia de garantía (por fecha y kilometraje), el trigger de reverso al pasar a CANCELADA y los triggers de `audit_log` en las tablas críticas. | **Leider (BD)** | Crítica | S2-T01 | **8 h** |
| **S2-T03** | CAM-12 | Mockups responsivos de Gestión de Órdenes (con desplegable de carga de mecánicos), Ficha de Garantía y Checklist Técnico de Calidad, y modal de Solicitud/Aprobación de piezas. Validación del PO. | **Jhon (FE) y Josue** | Crítica | S1-T05 | **4 h** (Jhon 3h / Josue 1h) |
| **S2-T04** | HU-05, HU-06 | Construir la ventana/modal de Garantía y Checklist Técnico de Trabajos Realizados (torque, frenos, sistema eléctrico, prueba de ruta, cobertura en días/km) y la pantalla de Gestión de Órdenes (tabla en escritorio, tarjetas en móvil y selector de mecánico con barra de carga). | **Jhon (FE)** | Crítica | S2-T03, S1-T07 | **8 h** |
| **S2-T05** | HU-05, 06, 07 | API de órdenes y garantías: apertura de OT (validación de empleado activo y stock disponible), avance de estados, registro del Checklist Técnico de verificación para poder finalizar/entregar y emisión/reclamación de garantía por trabajos realizados. | **Luisa (BE1)** | Crítica | S2-T01 | **9 h** |
| **S2-T06** | HU-09 | Retroceso de estado por error humano (solo Admin) y cancelación con motivo obligatorio que invoca el reverso de stock. | **Luisa (BE1)** | Crítica | S2-T02, S2-T05 | **6 h** |
| **S2-T07** | HU-08, 10, CAM-05 | API de Solicitud (Mecánico) y Aprobación atómica (Admin) sobre `detalle_orden`, y cálculo del costo total (mano de obra \+ piezas a precio histórico, descontando ítems cubiertos por garantía vigente). | **Neyer (BE2)** | Crítica | S2-T02 | **14 h** |
| **S2-T08** | HU-08, HU-10 | Construir el modal de Solicitud/Aprobación de Piezas y la sección de Mano de Obra, Cobertura de Garantía y Total en la orden. | **Jhon (FE)** | Crítica | S2-T04, S2-T07 | **5 h** |
| **S2-T09** | Control | Revisión del día 3 y cierre del S2: flujo completo de orden, emisión y validación de garantía con checklist técnico, aprobación de stock, reverso al cancelar y registro en `audit_log`. | **Josue y Eimy** | Crítica | S2-T05, S2-T06, S2-T07, S2-T08 | **8 h** (4h c/u) |

**Total estimado Sprint 2:** 69 horas.

### Sprint 3: Pedidos a proveedores con reasignación, ventas con vendedor, pagos, dashboard BI y jornada laboral (Cierre de desarrollo)

**Meta:** Finalizar el 100% del desarrollo funcional: gestión de pedidos a proveedores con ventana de reasignación ante incumplimiento, ventas de mostrador, pagos, caja, dashboard BI, tareas y consulta de auditoría.

| ID Tarea | Historia / CAM | Tarea específica | Responsable | Prioridad | Depende de | Horas Est. |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| **S3-T01** | CAM-03, 04, 05, 06 | Crear `venta_mostrador` (con vendedor), `detalle_venta`, `pago` (orden o venta, método, monto, fecha, usuario) y `mov_caja`. | **Leider (BD)** | Alta | S2-T01 | **6 h** |
| **S3-T01b** | Lógica Proveedor | Crear tablas `pedido_proveedor` y `detalle_pedido_proveedor` (estados: GENERADO, ENVIADO, RECIBIDO, INCUMPLIDO, REASIGNADO) y función SQL de traspaso de pedido a proveedor alternativo. | **Leider (BD)** | Alta | S1-T01b, S3-T01 | **4 h** |
| **S3-T02** | CAM-01, CAM-07 | Definir la granularidad (línea de detalle; día, semana, mes) y crear las vistas SQL de BI: rotación y ritmo de compra por pieza, tasa de garantías activas/reclamadas, cumplimiento de proveedores, ventas por vendedor, ingresos, arqueo de caja, top de piezas, conteo de estados y alertas de stock. Validar con el PO. | **Leider (BD) y Josue** | Alta | S3-T01, S3-T01b | **8 h** (Leider 6h / Josue 2h) |
| **S3-T03** | HU-11, HU-12 | Crear la tabla `tarea` y la consulta SQL que obtiene al empleado activo con menor carga del día. | **Leider (BD)** | Media | S2-T01 | **3 h** |
| **S3-T04** | CAM-12 | Mockups responsivos de Venta de Mostrador, Pagos/Caja, Dashboard BI, Mis Tareas y Auditoría. Validación del PO. | **Jhon (FE) y Josue** | Alta | S2-T03 | **4 h** (Jhon 3h / Josue 1h) |
| **S3-T04b** | UI-14 Proveedores | Mockup responsivo de la ventana de Gestión de Proveedores/Pedidos y del Modal de Reasignación Rápida ante incumplimiento de entrega. Validación del PO. | **Jhon (FE) y Josue** | Alta | S1-T05, S2-T03 | **3 h** (Jhon 2h / Josue 1h) |
| **S3-T05** | HU-13, 14, CAM-04, 06 | API de ventas de mostrador (vendedor, `detalle_venta`, descuento de stock) y de pagos (pago al entregar la OT junto con activación de certificado de garantía, método de pago) con caja de ingresos y egresos. | **Neyer (BE2)** | Alta | S3-T01 | **8 h** |
| **S3-T05b** | Lógica Proveedor | API de Pedidos a Proveedor: emisión de solicitud de entrega, marcado de incumplimiento y servicio de reasignación inmediata a proveedor alternativo (con actualización de costo y registro en `audit_log`). | **Neyer (BE2)** | Alta | S3-T01b | **6 h** |
| **S3-T06** | HU-13, HU-14 | Construir las vistas de Venta de Mostrador, Pagos y Gestión de Caja (ingresos, egresos y balance). | **Jhon (FE)** | Alta | S3-T04, S3-T05 | **5 h** |
| **S3-T06b** | UI-14 Proveedores | Construir la ventana de Gestión de Pedidos a Proveedores y el Modal interactivo de Reasignación de Proveedor (selector de proveedor secundario por prioridad, costo y tiempo de entrega). | **Jhon (FE)** | Alta | S3-T04b, S3-T05b | **4 h** |
| **S3-T07** | HU-15, CAM-07 | Exponer los endpoints del dashboard BI conectados a las vistas SQL, con filtros por granularidad (día, semana, mes), rango de fechas y vendedor. | **Neyer (BE2)** | Alta | S3-T02 | **6 h** |
| **S3-T08** | HU-15, CAM-07 | Construir el Dashboard BI responsivo: tarjetas KPI, tarjetas de estados y garantías vigentes, gráficas de ventas y rotación con selector de granularidad, alertas de stock y resumen financiero. | **Jhon (FE)** | Alta | S3-T04, S3-T07 | **5 h** |
| **S3-T09** | HU-11, HU-12 | `TareaService`: prioridades Alta/Media/Baja, avance por el titular y transferencia automática al empleado con menor carga. | **Luisa (BE1)** | Media | S3-T03 | **9 h** |
| **S3-T10** | HU-11, HU-12 | Construir la Bandeja Personal "Mis Tareas" y el modal de asignación/reasignación por inasistencia *(Recortable si falta tiempo)*. | **Jhon (FE)** | Media | S3-T04, S3-T09 | **3 h** |
| **S3-T11** | CAM-02 | API de consulta del `audit_log` (solo Admin) con filtros por usuario, tabla y fecha. | **Luisa (BE1)** | Media | S2-T02 | **6 h** |
| **S3-T12** | CAM-02 | Vista simple de Auditoría (tabla con filtros) para el rol Admin *(Recortable si falta tiempo)*. | **Jhon (FE)** | Media | S3-T10, S3-T11 | **2 h** |
| **S3-T13** | Control | Feature Freeze: revisión de las 15 historias, del sistema de garantías, del flujo de reasignación de proveedores y de los cambios CAM. A partir de aquí no se escribe ninguna función nueva. | **Josue y Eimy** | Crítica | S3-T05 a S3-T12 | **8 h** (4h c/u) |

**Total estimado Sprint 3:** 90 horas. *(Nota de capacidad: Si falta tiempo a Jhon en el S3, se recortan S3-T10 y S3-T12 ahorrando 5h para blindar Proveedores, Garantías, Pagos, Caja y Dashboard BI).*

### Sprint 4: Cero desarrollo nuevo — QA, estabilización, despliegue final y sustentación

**Meta:** Pruebas de extremo a extremo (HU-16), datos semilla en Neon (incluyendo garantías vigentes/reclamadas, proveedores y pedidos reasignados), corrección de errores, verificación responsiva, despliegue final estable y simulacro de sustentación.

| ID Tarea | Historia / CAM | Tarea específica | Responsable | Prioridad | Depende de | Horas Est. |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| **S4-T01** | HU-16 | Consolidar el script de datos semilla en Neon (clientes, motos, catálogo, proveedores principales/alternativos, pedidos reasignados, órdenes con checklist técnico y garantías emitidas, ventas, pagos, caja y `audit_log`) y probar la restauración limpia. | **Leider (BD)** | Crítica | S3-T13 | **8 h** |
| **S4-T02** | HU-16 | Suite de integridad: consumos sin stock, que CANCELADA devuelva las piezas exactas, validación de cobertura de garantías por fecha/km, reasignación de proveedor ante incumplimiento, pagos contra caja sin descuadres y coherencia de las cifras del dashboard BI. | **Neyer (BE2) y Leider (BD)** | Crítica | S4-T01 | **12 h** (6h c/u) |
| **S4-T03** | HU-16 | Auditar seguridad y permisos por rol (Admin, Mecánico, Vendedor; token, CORS, variables de entorno) y verificar que Mecánico y Vendedor solo operen sus módulos autorizados. | **Luisa (BE1)** | Crítica | S4-T01 | **6 h** |
| **S4-T04** | HU-16, CAM-12 | Pruebas responsivas en móvil, tablet y escritorio (Tema Claro y Oscuro), y pulido de UX: validaciones claras, certificado de garantía, ventana de proveedores, formato de moneda y refresco de tablas. | **Jhon (FE)** | Alta | S3-T13 | **10 h** |
| **S4-T05** | HU-16 | Resolución de bugs reportados en S4-T02, S4-T03 y S4-T04. | **Luisa y Neyer** | Crítica | S4-T02, S4-T03 | **12 h** (6h c/u) |
| **S4-T06** | CAM-09, 10, 11 | Despliegue final (Vercel, Render, Neon), verificación del cron job/auto-ping, plan de contingencia ante arranque en frío y guía rápida de uso con las URL oficiales. | **Luisa y Jhon** | Crítica | S4-T04, S4-T05 | **8 h** (4h c/u) |
| **S4-T07** | Cierre | Pruebas de Aceptación de Usuario (UAT), firma del Acta de Aceptación y simulacro de sustentación con todo el equipo. | **Josue y Eimy** | Crítica | S4-T06 | **12 h** (6h c/u) |

**Total estimado Sprint 4:** 68 horas.

## 3\. Desglose Individual de Tareas por Integrante (Actualizado con Garantías y Proveedores)

### 3.1 Leider — Encargado de Base de Datos (PostgreSQL en Neon)

**Total acumulado en los 4 Sprints:** **64 horas** (S1: 16h \| S2: 15h \| S3: 19h \| S4: 14h).

| Sprint | ID Tarea | Tarea específica | Prioridad | Depende de | Horas |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **S1** | **S1-T01** | Crear proyecto en Neon y script DDL maestro v2 (`usuario`, `cliente`, `vehiculo` 1 a N, `pieza`, `mov_inventario`, `audit_log`) con `created_at`, `updated_at` y trigger genérico. | Crítica | Ninguna | 12 h |
| **S1** | **S1-T01b** | Crear tablas `proveedor` y `pieza_proveedor` (prioridad 1..N y costo pactado) con `created_at` y `updated_at` en Neon. | Alta | S1-T01 | 4 h |
| **S2** | **S2-T01** | Crear `orden_trabajo` (con vendedor y referencia a garantía), `garantia_orden` (plazo en días/km, km de entrega, estado y checklist técnico útil: torque/seguridad, frenos/transmisión, sistema eléctrico, estanqueidad y prueba de ruta) y `detalle_orden` con `created_at` y `updated_at`. | Crítica | S1-T01 | 7 h |
| **S2** | **S2-T02** | Programar función SQL de descuento atómico al aprobar piezas, validación de vigencia de garantía, trigger de reverso al pasar a CANCELADA y triggers de `audit_log` en tablas críticas. | Crítica | S2-T01 | 8 h |
| **S3** | **S3-T01** | Crear `venta_mostrador` (con vendedor), `detalle_venta`, `pago` (orden/venta, método, monto, fecha, usuario) y `mov_caja`. | Alta | S2-T01 | 6 h |
| **S3** | **S3-T01b** | Crear tablas `pedido_proveedor` y `detalle_pedido_proveedor` con soporte de estados y función SQL de reasignación a proveedor alternativo. | Alta | S1-T01b, S3-T01 | 4 h |
| **S3** | **S3-T02** | Definir granularidad (día, semana, mes) y crear vistas SQL de BI (rotación/ritmo de compra, garantías reclamadas vs. emitidas, cumplimiento de proveedores, ventas por vendedor, ingresos, arqueo, top piezas, estados, alertas) junto con Josue. | Alta | S3-T01, S3-T01b | 6 h |
| **S3** | **S3-T03** | Crear tabla `tarea` y consulta SQL que obtiene al empleado activo con menor carga laboral del día. | Media | S2-T01 | 3 h |
| **S4** | **S4-T01** | Consolidar script de datos semilla en Neon (clientes, motos, catálogo, proveedores, pedidos reasignados, órdenes con checklist técnico y garantías, ventas, pagos, caja y `audit_log`) y probar restauración limpia. | Crítica | S3-T13 | 8 h |
| **S4** | **S4-T02** | Ejecutar junto a Neyer la suite de integridad en BD (consumos sin stock, reverso exacto en CANCELADA, cobertura de garantías, reasignación de proveedor, cuadre de pagos contra caja y vistas BI). | Crítica | S4-T01 | 6 h |

### 3.2 Luisa — Backend 1 (Flujo Operativo, Garantías, Seguridad y Despliegue en Render/Docker)

**Total acumulado en los 4 Sprints:** **63 horas** (S1: 17h \| S2: 15h \| S3: 15h \| S4: 16h).

| Sprint | ID Tarea | Tarea específica | Prioridad | Depende de | Horas |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **S1** | **S1-T02** | Crear proyecto backend (Java 21, API REST) con conexión a Neon por variables de entorno (SSL) y endpoint `/health`. | Crítica | S1-T01 | 4 h |
| **S1** | **S1-T03** | Escribir Dockerfile, desplegar backend en Render y configurar cron job (cron-job.org) o auto-ping a `/health`. | Crítica | S1-T02 | 4 h |
| **S1** | **S1-T06** | Implementar servicio de autenticación (credenciales cifradas, token y rol Admin/Mecánico/Vendedor) y endpoint de login. | Crítica | S1-T01, S1-T02 | 4 h |
| **S1** | **S1-T08** | Programar API de Clientes y Motocicletas (validación de placa única y relación 1 a N). | Crítica | S1-T01, S1-T02 | 5 h |
| **S2** | **S2-T05** | API de órdenes y garantías: apertura de OT, validación de empleado activo y stock, registro obligatorio del Checklist Técnico de calidad (torque, frenos, eléctrico, prueba de ruta) y emisión/reclamación de garantía por trabajos realizados. | Crítica | S2-T01 | 9 h |
| **S2** | **S2-T06** | Implementar retroceso de estado por error humano (solo Admin) y cancelación con motivo obligatorio que invoca el reverso de stock. | Crítica | S2-T02, S2-T05 | 6 h |
| **S3** | **S3-T09** | Programar `TareaService`: prioridades Alta/Media/Baja, avance por el titular y transferencia automática al empleado con menor carga. | Media | S3-T03 | 9 h |
| **S3** | **S3-T11** | Programar API de consulta del `audit_log` (solo Admin) con filtros por usuario, tabla y fecha. | Media | S2-T02 | 6 h |
| **S4** | **S4-T03** | Auditar seguridad y permisos por rol (Admin, Mecánico, Vendedor; token, CORS, variables de entorno) y verificar restricciones de acceso. | Crítica | S4-T01 | 6 h |
| **S4** | **S4-T05** | Resolución de bugs de flujo operativo, garantías, autenticación y permisos reportados en las pruebas. | Crítica | S4-T02, S4-T03 | 6 h |
| **S4** | **S4-T06** | Verificar junto a Jhon el despliegue final en Render/Neon, cron job/auto-ping, contingencia de arranque en frío y guía rápida. | Crítica | S4-T04, S4-T05 | 4 h |

### 3.3 Neyer — Backend 2 (Inventario, Proveedores, Detalle de Piezas, Pagos, Caja y Dashboard BI)

**Total acumulado en los 4 Sprints:** **60 horas** (S1: 14h \| S2: 14h \| S3: 20h \| S4: 12h).

| Sprint | ID Tarea | Tarea específica | Prioridad | Depende de | Horas |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **S1** | **S1-T10** | Programar API del Catálogo de Piezas (Repuesto/Accesorio) y entradas de stock transaccionales con registro en bitácora. | Crítica | S1-T01, S1-T02 | 10 h |
| **S1** | **S1-T10b** | Programar endpoints CRUD de Proveedores y asociación pieza-proveedor con orden de prioridad y costo. | Alta | S1-T01b, S1-T10 | 4 h |
| **S2** | **S2-T07** | Programar API de Solicitud (Mecánico) y Aprobación atómica (Admin) sobre `detalle_orden`, y cálculo de costo total (mano de obra \+ piezas a precio histórico, contemplando exención por garantía vigente). | Crítica | S2-T02 | 14 h |
| **S3** | **S3-T05** | Programar API de ventas de mostrador (vendedor, `detalle_venta`, descuento de stock) y de pagos (`pago` al entregar OT con activación de garantía, método de pago) con caja de ingresos y egresos. | Alta | S3-T01 | 8 h |
| **S3** | **S3-T05b** | Programar API de Pedidos a Proveedor: solicitud de entrega, registro de incumplimiento y servicio de reasignación inmediata a proveedor secundario con trazabilidad en `audit_log`. | Alta | S3-T01b | 6 h |
| **S3** | **S3-T07** | Exponer endpoints del Dashboard BI conectados a las vistas SQL de Leider, con filtros por granularidad, rango de fechas y vendedor. | Alta | S3-T02 | 6 h |
| **S4** | **S4-T02** | Ejecutar junto a Leider la suite de integridad: consumos sin stock, devolución exacta en CANCELADA, cobertura de garantías, reasignación de pedidos a proveedores, cuadre de pagos contra caja y cifras del Dashboard BI. | Crítica | S4-T01 | 6 h |
| **S4** | **S4-T05** | Resolución de bugs de inventario, proveedores, cálculo de costos, pagos, caja y BI reportados en las pruebas. | Crítica | S4-T02, S4-T03 | 6 h |

### 3.4 Jhon — Frontend (Mockups Responsivos, Vue 3 y Despliegue en Vercel)

**Total acumulado en los 4 Sprints:** **72 horas** (S1: 18h \| S2: 16h \| S3: 24h \[19h si se recortan T10 y T12\] \| S4: 14h).

| Sprint | ID Tarea | Tarea específica | Prioridad | Depende de | Horas |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **S1** | **S1-T04** | Crear proyecto Vue, desplegar en Vercel, configurar URL de API y layout base responsivo con interruptor de Tema Claro / Oscuro. | Crítica | Ninguna | 3 h |
| **S1** | **S1-T05** | Elaborar mockups responsivos (móvil, tablet, escritorio) de Login, Clientes/Motos y Catálogo de Piezas con Proveedores para validación con Josue. | Crítica | Ninguna | 3 h |
| **S1** | **S1-T07** | Construir vista de Login y contenedor principal con menú lateral adaptable por rol (Admin, Mecánico, Vendedor). | Crítica | S1-T04, S1-T05, S1-T06 | 3 h |
| **S1** | **S1-T09** | Construir vistas de Clientes (búsqueda, tabla, formulario) y panel de Motocicletas asociadas. | Crítica | S1-T07, S1-T08 | 4 h |
| **S1** | **S1-T11** | Construir vistas del Catálogo (alerta visual de stock mínimo), pestaña de Proveedores por pieza y modal de entrada de stock. | Crítica | S1-T07, S1-T10, S1-T10b | 5 h |
| **S2** | **S2-T03** | Elaborar mockups responsivos de Gestión de Órdenes (con desplegable de carga de mecánicos), Ficha de Garantía y Checklist Técnico de Calidad, y modal de Solicitud/Aprobación de piezas. | Crítica | S1-T05 | 3 h |
| **S2** | **S2-T04** | Construir ventana/modal de Garantía y Checklist Técnico de Trabajos Realizados (torque, frenos, sistema eléctrico, prueba de ruta, plazo días/km) y pantalla de Gestión de Órdenes con desplegable de carga laboral. | Crítica | S2-T03, S1-T07 | 8 h |
| **S2** | **S2-T08** | Construir modal de Solicitud/Aprobación de Piezas y sección de Mano de Obra, Estado de Garantía y Total en la orden. | Crítica | S2-T04, S2-T07 | 5 h |
| **S3** | **S3-T04** | Elaborar mockups responsivos de Venta de Mostrador, Pagos/Caja, Dashboard BI, Mis Tareas y Auditoría. | Alta | S2-T03 | 3 h |
| **S3** | **S3-T04b** | Elaborar mockup responsivo de la ventana de Pedidos a Proveedores y Modal de Reasignación Rápida ante incumplimiento. | Alta | S1-T05, S2-T03 | 2 h |
| **S3** | **S3-T06** | Construir vistas de Venta de Mostrador, Pagos y Gestión de Caja (ingresos, egresos y balance). | Alta | S3-T04, S3-T05 | 5 h |
| **S3-T06b** | Construir ventana de Gestión de Proveedores/Pedidos y el Modal de Reasignación a Proveedor Alternativo. | Alta | S3-T04b, S3-T05b | 4 h |  |
| **S3** | **S3-T08** | Construir Dashboard BI responsivo: tarjetas KPI, estados y garantías vigentes, gráficas de ventas/rotación con selector de granularidad, alertas de stock y resumen financiero. | Alta | S3-T04, S3-T07 | 5 h |
| **S3** | **S3-T10** | Construir Bandeja Personal "Mis Tareas" y modal de asignación/reasignación por inasistencia *(Recortable si falta tiempo)*. | Media | S3-T04, S3-T09 | 3 h |
| **S3** | **S3-T12** | Construir vista simple de Auditoría (tabla con filtros) para el rol Admin *(Recortable si falta tiempo)*. | Media | S3-T10, S3-T11 | 2 h |
| **S4** | **S4-T04** | Ejecutar pruebas responsivas en móvil, tablet y escritorio, y pulido de UX (validaciones, certificado de garantía, ventana de proveedores, formato moneda, refresco de tablas). | Alta | S3-T13 | 10 h |
| **S4** | **S4-T06** | Verificar despliegue final en Vercel junto a Luisa y redactar guía rápida de uso con las URL oficiales. | Crítica | S4-T04, S4-T05 | 4 h |

### 3.5 Josue (Product Owner) y Eimy (Scrum Master) — Gestión, Validación y Control

**Total acumulado en los 4 Sprints:** **Josue: 24 horas** \| **Eimy: 18 horas** (horas de validación directa de entregables, adicionales a la gestión diaria del tablero Kanban y ceremonias).

| Sprint | ID Tarea | Tarea específica | Responsable | Prioridad | Depende de | Horas |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| **S1** | **S1-T05** | Validar y aprobar mockups responsivos de Login, Clientes/Motos y Catálogo de Piezas con Proveedores. | **Josue (PO)** | Crítica | Ninguna | 1 h |
| **S1** | **S1-T12** | Revisión del día 3 y cierre del S1: validar despliegue en la nube, login por roles, clientes/motos, inventario y proveedores. | **Josue y Eimy** | Crítica | S1-T03, S1-T09, S1-T11 | 8 h (4h c/u) |
| **S2** | **S2-T03** | Validar y aprobar mockups responsivos de Gestión de Órdenes, Ficha de Garantía y Checklist Técnico, y Solicitud de piezas. | **Josue (PO)** | Crítica | S1-T05 | 1 h |
| **S2** | **S2-T09** | Revisión del día 3 y cierre del S2: validar flujo completo de orden, emisión de garantía con checklist técnico, aprobación de stock, reverso al cancelar y `audit_log`. | **Josue y Eimy** | Crítica | S2-T05 a S2-T08 | 8 h (4h c/u) |
| **S3** | **S3-T02** | Validar con Leider la definición de granularidad (día, semana, mes) y las métricas de las vistas SQL de BI (incluyendo garantías y proveedores). | **Josue (PO)** | Alta | S3-T01, S3-T01b | 2 h |
| **S3** | **S3-T04 / 04b** | Validar y aprobar mockups de Mostrador, Pagos/Caja, Ventana de Reasignación de Proveedores, Dashboard BI, Tareas y Auditoría. | **Josue (PO)** | Alta | S2-T03 | 2 h |
| **S3** | **S3-T13** | Ejecutar Feature Freeze: revisión integral de las 15 historias, sistema de garantías, flujo de proveedores y cambios CAM-01 a CAM-12. | **Josue y Eimy** | Crítica | S3-T05 a S3-T12 | 8 h (4h c/u) |
| **S4** | **S4-T07** | Ejecutar Pruebas de Aceptación de Usuario (UAT), firmar Acta de Aceptación y liderar simulacro de sustentación. | **Josue y Eimy** | Crítica | S4-T06 | 12 h (6h c/u) |

## 4\. Resumen de Carga de Horas por Integrante y Sprint

| Integrante | Rol | Sprint 1 | Sprint 2 | Sprint 3 | Sprint 4 | Total Proyecto |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| **Leider** | Base de Datos (Neon) | 16 h | 15 h | 19 h | 14 h | **64 h** |
| **Luisa** | Backend 1 (Flujo, Garantías y Render) | 17 h | 15 h | 15 h | 16 h | **63 h** |
| **Neyer** | Backend 2 (Stock, Proveedores, Caja y BI) | 14 h | 14 h | 20 h | 12 h | **60 h** |
| **Jhon** | Frontend (Vue \+ Vercel) | 18 h | 16 h | 24 h *(o 19h)* | 14 h | **72 h** *(o 67h)* |
| **Josue** | Product Owner | 5 h | 5 h | 8 h | 6 h | **24 h** |
| **Eimy** | Scrum Master | 4 h | 4 h | 4 h | 6 h | **18 h** |
| **Total Sprint** | — | **74 h** | **69 h** | **90 h** | **68 h** | **301 h** |

&nbsp;