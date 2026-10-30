
# ESPECIFICACIÓN DE REQUISITOS DE SOFTWARE


**Sistema de Gestión Local para Taller y Venta de Piezas de Motocicletas — Soul Motors (MVP)** 


**Identificador:** SRS-TM-001 | **Versión:** 0.3 | **Estado:** Redefinición validada por Product Owner | **Fecha:** 29 de octubre de 2026.


**Referencia metodológica:** ISO/IEC/IEEE 29148:2018 | **Clasificación:** Uso académico / interno |

**Entorno:** Aplicación de Escritorio Local.


**Equipo de trabajo:** 6 integrantes; Product Owner confirmado, Scrum Master (Yohely) y 4 desarrolladores (Neyer y Luisa en Backend, Jhon en Frontend y Leider en Base de Datos). 


### Control documental

| **Versión** | **Fecha** | **Responsable** | **Cambios** |
| - | - | - | - |
| 0.1 | 26/10/2026 | Project Manager (Yohely) | Diagnóstico, MVP acotado, SRS/TRD, backlog y planificación inicial de 4 sprints.   |
| 0.2 | 26/10/2026 | Project Manager (Yohely) | Ampliación FL-01..08, secuencia transaccional, modelo relacional inicial y criterios CA de HU-01..16.   |
| 0.3 | 29/10/2026 | Product Owner y Equipo Soul Motors | Redefinición arquitectónica a Java 21 Desktop (Swing + FlatLaf) + PostgreSQL local; inclusión de fase DIAGNÓSTICO con checklist, flujo de solicitud/aprobación de piezas (repuestos y accesorios), retroceso y cancelación con reverso automático, reasignación por bajas, trazabilidad financiera (OBJ-06) y asignación nominal del equipo.  |







### Responsables de revisión y aprobación


| **Rol aprobador** | **Nombre** | **Aprobación** | **Fecha** |
| - | - | - | - |
| Product Owner | Josue | Aprobado | 29/10/2026 |
| Project Manager / Scrum Master | Yohely | Aprobado | 29/10/2026 |
| Autor de idea / Encargado BD | Leider | Aprobado | 29/10/2026 |
| Equipo de Desarrollo | Neyer (Backend), Luisa (Backend), Jhon (Frontend), Leider (BD) | Confirmado | 29/10/2026 |


### Índice de contenidos


| **Bloque** | **Capítulos** |
| - | - |
| Diagnóstico previo | A. Factibilidad, arquitectura local y decisiones de alcance   |
| Parte I — Producto | 1. Introducción; 2. Descripción general; 3. Visión y objetivos; 4. Usuarios y escenarios; 5. Requisitos funcionales; 6. Requisitos no funcionales   |
| Parte II — Técnica | 7. Arquitectura Desktop; 8. Flujos y reglas de negocio; 9. Interfaces Swing; 10. Contratos de servicios y eventos; 11. Modelo de datos; 12. Seguridad; 13. Empaquetado y despliegue local; 14. Calidad; 15. Verificación; 16. Acuerdos y consultas; 17. Glosario   |
| Parte III — Scrum | 18. Equipo; 19. Product Backlog; 20. Plan de sprints; 21. Sprint Backlog; 22. Capacidad; 23. Eventos; 24. DoR / DoD; 25. Riesgos; 26. Seguimiento cada 3 días; 27. Entregables   |
| Parte IV — Auditoría | 28. Validación cruzada; 29. Resultado de evaluación   |
| Anexos | A. Referencias; B. Diagramas Mermaid; C. DDL completo de PostgreSQL   |


## A. DIAGNÓSTICO PREVIO Y DEFINICIÓN DE ALCANCE


**Dictamen: VIABLE CON ARQUITECTURA DESKTOP LOCAL Y DISTRIBUCIÓN EQUILIBRADA.**


Se descarta la propuesta original web en Django y se adopta una aplicación de escritorio de uso local en **Java 21 LTS con Java Swing + FlatLaf y PostgreSQL local**. Para evitar cuellos de botella sobre el único desarrollador de Frontend (Jhon), el encargado de Base de Datos (Leider) implementa lógica pesada en PostgreSQL (triggers de reverso, funciones transaccionales y vistas) y el equipo de Backend (Neyer y Luisa) asume tanto la lógica de negocio como los controladores de eventos de las pantallas.


| **Aspecto** | **Observación** | **Impacto y Decisión** |
| - | - | - |
| Complejidad técnica | Baja curva de aprendizaje visual usando el diseñador Drag & Drop del IDE con Swing + FlatLaf.  | Elimina tiempos de maquetación web y acelera la construcción de formularios de escritorio. |
| Componentes | Aplicación Desktop en 4 capas lógicas + PostgreSQL local (localhost:5432).  | Cero dependencia de servidores externos, hosting, dominios o conexión a internet.   |
| Inventario y Órdenes | Se amplía de "repuestos" a Piezas (Repuestos y Accesorios) y se incorpora recepción con Diagnóstico y Checklist físico.  | El paso a PENDIENTE exige viabilidad técnica, empleado activo asignado y stock disponible.  |
| Trazabilidad Financiera | Se incorpora el objetivo OBJ-06 (ventas de mostrador, ingresos, egresos y caja).  | Alcance técnico detallado coordinado entre Product Owner, Neyer y Luisa para los Sprints 3 y 4.  |
| Contingencia de Alcance (Resolución Q4) | Si ocurre un recorte de tiempo adicional en el desarrollo, se blinda el núcleo operativo.  | Se prioriza estrictamente el control de inventario de piezas y el flujo de órdenes por encima del módulo de tareas diarias.  |


### A.1 Matriz MoSCoW actualizada y decisiones de alcance


| **Capacidad del Sistema** | **Clasificación MoSCoW** | **Justificación y Tratamiento en v0.3** |
| - | - | - |
| Clientes, motos (1:N) e historial completo | Debe | Base de la recepción diagnóstica y consulta histórica por moto y por cliente.   |
| Catálogo de Piezas (Repuestos y Accesorios), entradas y solicitud/aprobación | Debe | El mecánico solicita piezas y solo el Administrador aprueba la salida en transacción atómica.  |
| Órdenes de 6 estados (incluyendo DIAGNÓSTICO y CANCELADA) y checklist físico | Debe | Flujo operativo central con validación de stock, retroceso por error (Admin) y reverso automático al cancelar.  |
| Tareas diarias, prioridad y reasignación por inasistencias | Debe | Organiza la jornada de 48 h semanales y transfiere carga al empleado con menor ocupación ante bajas.  |
| Alertas de stock mínimo y panel de órdenes por estado | Debería | Visibilidad operativa rápida apoyada en vistas de PostgreSQL.   |
| Ventas de mostrador y Trazabilidad Financiera (OBJ-06: ingresos, egresos y caja) | Debería | Incorporado en Sprints 3 y 4 según nivel de detalle acordado entre Product Owner y Backend.  |
| Facturación electrónica DIAN, pagos en línea y multi-sede | No en esta versión | Excluido deliberadamente por tratarse de un sistema de gestión interna local.   |


## 1. Introducción


### 1.1 Propósito


Este SRS define el contrato verificable de requisitos de producto (PRD), el diseño técnico (TRD) y el plan de ejecución Scrum en cuatro sprints del sistema **Soul Motors**. Sirve al Product Owner para validar incrementos cada 3 días, a la Scrum Master (Yohely) para gestionar el flujo de trabajo y a los desarrolladores (Neyer, Luisa, Jhon y Leider) para implementar y verificar el software. 


### 1.2 Alcance del producto


Aplicación de escritorio local para la administración de un taller de reparación de motocicletas y venta de piezas (repuestos y accesorios). Permite registrar clientes y motocicletas (relación 1:N con placa única), realizar diagnósticos de recepción con checklist físico, gestionar órdenes de trabajo a través de 6 estados (DIAGNÓSTICO, PENDIENTE, EN\_PROCESO, FINALIZADA, ENTREGADA y CANCELADA), controlar el inventario mediante solicitudes del mecánico y aprobaciones del Administrador, administrar tareas diarias con reasignación ante inasistencias, registrar ventas directas de mostrador y llevar la trazabilidad financiera de ingresos, egresos y caja (OBJ-06).


### 1.3 Audiencia y uso


| **Audiencia** | **Integrantes** | **Uso principal** | **Capítulos** |
| - | - | - | - |
| Product Owner | Josue | Validar incrementos cada 3 días, priorizar backlog y aprobar entregas.   | A, 3–6, 16, 19, 27   |
| Scrum Master / PM | Yohely | Facilitar eventos Scrum, vigilar capacidad y gestionar riesgos e impedimentos.   | 18–29   |
| Backend y Base de Datos | Neyer, Luisa y Leider | Arquitectura en capas, servicios Java, modelo SQL, triggers y vistas.   | 5, 7–14, 21, Anexo C   |
| Frontend Desktop | Jhon | Construcción de interfaces en Java Swing + FlatLaf con diseñador visual.   | 4, 8, 9, 21   |


### 1.4 Definiciones y abreviaturas


| **Término** | **Definición** |
| - | - |
| MVP | Producto mínimo viable verificable en cuatro sprints.   |
| Pieza | Ítem de inventario clasificado en dos categorías: Repuesto o Accesorio, contabilizado en unidades enteras.  |
| Diagnóstico | Fase inicial obligatoria de una orden que registra kilometraje, nivel de combustible, checklist físico de llegada y viabilidad técnica.  |
| OT | Orden de trabajo de mantenimiento, reparación o instalación sobre una motocicleta.   |
| Movimiento | Registro inmutable de entrada, salida aprobada por orden, venta de mostrador o reverso automático por cancelación.   |
| PO / SM | Product Owner / Scrum Master.   |
| RF / RNF / RN | Requisito funcional / Requisito no funcional / Regla de negocio.   |
| HU / CP | Historia de usuario / Caso de prueba.   |


### 1.5 Referencias

- ISO/IEC/IEEE 29148:2018 — Ingeniería de sistemas y software: Procesos y especificación de requisitos. 


- Documentación oficial de Java 21 LTS, Apache Maven, Java Swing y librería visual FlatLaf.


- Documentación oficial de PostgreSQL (esquemas relacionales, transacciones ACID, triggers en PL/pgSQL y vistas) y conectividad JDBC / JPA (Hibernate).


### 1.6 Convenciones

Los identificadores RF-\[MÓDULO\]-NN, RNF-\[CALIDAD\]-NN, RN-NN, HU-NN y CP-NN son estables en todo el documento. "Debe" indica un requisito obligatorio del MVP, "Debería" un requisito implementado tras cubrir el flujo central y "No en esta versión" una exclusión acordada. 


## 2. Descripción general


### 2.1 Perspectiva del producto

Sistema de escritorio nuevo de uso local en un taller y almacén de piezas de motocicletas. Se estructura en 4 capas lógicas: Usuarios -\> Capa de Presentación Desktop (Java Swing + FlatLaf) -\> Capa de Negocio/Servicios en Java 21 -\> Capa de Datos en PostgreSQL local (localhost:5432), eliminando la necesidad de navegadores web o servidores remotos. 


### 2.2 Funciones del producto


| **Función** | **Incluye en el MVP (v0.3)** |
| - | - |
| Clientes y motocicletas | Fichas de clientes, relación 1:N (un cliente tiene varias motos; cada moto un único dueño y placa única) e historial completo por cliente y por moto.  |
| Diagnóstico y Órdenes (OT) | Apertura en DIAGNÓSTICO (kilometraje, combustible y checklist físico), validación de stock y empleado activo para pasar a PENDIENTE, avance normal, retroceso por error humano (solo Admin) y CANCELADA con reverso automático al stock.  |
| Inventario de Piezas | Catálogo de Repuestos y Accesorios, entradas de stock, solicitud de piezas por el mecánico y aprobación transaccional por el Administrador.  |
| Jornada y Contingencias | Tareas diarias por prioridad (ALTA, MEDIA, BAJA) y reasignación de órdenes/tareas al empleado activo con menor carga ante inasistencias.  |
| Mostrador y Finanzas (OBJ-06) | Ventas directas de mostrador, registro centralizado de ingresos, egresos, estado de caja, cuentas y estadísticas financieras.  |


### 2.3 Clases y características de usuarios


| **Usuario** | **Frecuencia** | **Competencia esperada** | **Acceso y Responsabilidades** |
| - | - | - | - |
| Administrador del taller | Diaria | Uso básico de PC | Gestión total: clientes, motos, diagnósticos, órdenes, asignaciones y reasignaciones por bajas, catálogo de piezas, entradas de stock, tareas y finanzas (caja, ventas de mostrador, ingresos/egresos). Único autorizado para aprobar salidas de piezas, retroceder estados por error, cancelar órdenes y marcar órdenes como ENTREGADA.   |
| Empleado (Mecánico / Operario) | Diaria | Uso básico de PC | Registra datos técnicos en diagnóstico, consulta sus órdenes asignadas, solicita piezas al Administrador, avanza sus órdenes (PENDIENTE -\> EN\_PROCESO -\> FINALIZADA) y actualiza el estado de sus propias tareas diarias.   |



### 2.4 Entorno operativo

Computador local del taller con sistema operativo Windows o Linux, entorno de ejecución Java 21 LTS (JRE/JDK) y motor de base de datos PostgreSQL ejecutándose localmente en el puerto 5432. La demostración y sustentación oficial se realizará de forma local en el computador asignado. 


### 2.5 Restricciones de implementación

- **Topología real del equipo:** 1 Product Owner, 1 Scrum Master (Yohely) y 4 desarrolladores: Neyer y Luisa en Backend, Jhon en Frontend y Leider en Base de Datos.


- **Stack tecnológico obligatorio:** Java 21 LTS con Maven, interfaz gráfica en Java Swing + FlatLaf apoyada en el diseñador Drag & Drop del IDE, acceso a datos con JDBC / JPA y PostgreSQL local inicializado mediante script .sql maestro.


- **Cadencia de validación (Resolución Q2):** Revisiones de avance cada 3 días lideradas por el Product Owner para detectar y corregir desviaciones a tiempo.


- **Datos de prueba (Resolución Q6):** Uso exclusivo de datos ficticios precargados mediante script SQL.


### 2.6 Supuestos y dependencias

| **ID** | **Supuesto validado** | **Efecto / Regla aplicada** |
| - | - | - |
| S-01 | Un solo local de taller y venta de piezas con base de datos local.   | Despliegue en localhost:5432 sin dependencia de internet.  |
| S-02 | Relación 1:N estricta entre cliente y motocicleta, con placa única por vehículo.   | Cada moto pertenece a un único cliente; el historial se consulta por ambos.  |
| S-03 | Las piezas (repuestos y accesorios) se descuentan en unidades enteras.   | Restricción de enteros positivos y stock nunca negativo (stock\_actual \>= 0).   |
| S-04 | Jornada laboral en el taller de ~48 h semanales por empleado.  | Ante bajas o inasistencias, se reasigna el trabajo al empleado activo con menor carga del día o se reprograma.  |
| S-05 | Revisiones de control cada 3 días con el Product Owner.  | Mitigación temprana de bloqueos técnicos o atrasos estudiantiles.  |



## 3. PRD: Visión y objetivos del producto


### 3.1 Problema

En la operación diaria de un taller y almacén de piezas de motocicletas, gestionar las recepciones y reparaciones de forma manual impide dejar constancia del estado físico en que ingresa cada moto, provoca la aceptación de trabajos sin verificar si existen piezas en stock, permite descuentos de inventario sin supervisión administrativa, dificulta redistribuir el trabajo cuando falta un mecánico y descontrola el cuadre de caja entre reparaciones y ventas de mostrador. 


### 3.2 Visión

Proveer una aplicación de escritorio local, sólida y ágil en Java que centralice el flujo operativo y financiero de **Soul Motors**: desde la recepción diagnóstica con checklist físico de la motocicleta, el control estricto de piezas (repuestos y accesorios) mediante aprobación del Administrador y la gestión equitativa de la jornada laboral, hasta las ventas directas de mostrador y el control de caja.


### 3.3 Objetivos y métricas de éxito


| **ID** | **Objetivo Específico** | **Meta Verificable** | **Medición** |
| - | - | - | - |
| OBJ-01 | Digitalizar el flujo principal | Flujo cliente -\> vehículo -\> catálogo de piezas -\> diagnóstico -\> orden de trabajo funcionando entre el Sprint 1 y Sprint 2.  | Demostración funcional en S1 y S2.  |
| OBJ-02 | Trazar el servicio por cliente y moto | Un cliente puede tener múltiples motos asociadas (1:N), pero cada moto pertenece a un único cliente; historial completo consultable por moto y por cliente.  | Pruebas de aislamiento y consulta cronológica.   |
| OBJ-03 | Mantener consistencia de inventario | Cero consumos sin stock disponible; flujo de solicitud por empleado y aprobación por Administrador; reverso automático de piezas al cancelar y registro de usuario en todo movimiento.  | Transacciones ACID y triggers en PostgreSQL.  |
| OBJ-04 | Priorizar y equilibrar la jornada | Cada empleado ve solo sus tareas ordenadas por prioridad y actualiza su estado; el Administrador puede reasignar carga laboral ante inasistencias.  | Pruebas de bandeja por rol y transferencia de carga.  |
| OBJ-05 | Cerrar un MVP seguro y estable | Sin defectos bloqueantes y casos de prueba críticos aprobados en el Sprint 4.  | Ejecución de suite de pruebas en sustentación local.  |
| OBJ-06 | Garantizar trazabilidad financiera | Registro centralizado de ingresos (órdenes entregadas y ventas de mostrador), egresos (compra de piezas y gastos), estado de caja, cuentas y estadísticas financieras.  | Verificación de arqueo de caja y vistas SQL en S4.  |


## 4. PRD: Usuarios y escenarios principales

- **E-01 — Recepción diagnóstica con checklist y apertura de OT: El Administrador localiza o registra al cliente, selecciona su motocicleta y abre la orden en estado DIAGNÓSTICO, registrando obligatoriamente kilometraje, nivel de combustible y el checklist físico de llegada (rayones, espejos, luces, estado general). Para aceptar el trabajo y pasarlo a PENDIENTE, el sistema comprueba que exista viabilidad técnica, un empleado activo asignado y stock disponible de las piezas requeridas.**


- **E-02 — Solicitud de piezas por el mecánico y aprobación atómica del Administrador:** Durante la intervención, el mecánico asignado solicita piezas (repuestos o accesorios) desde la orden de trabajo. El Administrador revisa la solicitud y la aprueba; en ese instante el sistema bloquea el registro, descuenta el stock y crea el movimiento de salida en una sola transacción atómica, permitiendo avanzar la orden a EN\_PROCESO y posteriormente a FINALIZADA.


- **E-03 — Retroceso por error humano o Cancelación con reverso automático:** Si un empleado avanza el estado de una orden por equivocación, únicamente el Administrador puede retrocederlo al estado inmediato anterior. Si sobreviene un fallo mecánico sin solución o falta de recursos, el Administrador cambia la orden a CANCELADA registrando el motivo; un trigger/transacción reintegra automáticamente todas las piezas al inventario mediante movimientos de REVERSO y el taller devuelve la motocicleta al cliente guiándose por el checklist físico de recepción.


- **E-04 — Reasignación operativa por inasistencia:** Si un empleado falta en su jornada, el Administrador consulta en el sistema cuál es el trabajador activo con menor carga laboral en ese día y le reasigna las órdenes y tareas pendientes, o bien reprograma la fecha de entrega notificando al cliente.


## 5. PRD: Requisitos Funcionales (RF)


### 5.1 Autenticación y Autorización


| **ID / MoSCoW** | **Descripción y Actor** | **Condición de Aceptación Observable** | **Dependencia / Sprint** |
| - | - | - | - |
| RF-AUT-01 (Debe) | Iniciar y cerrar sesión con credenciales individuales en la aplicación de escritorio (Actor: Administrador / Empleado).  | Valida usuario activo y contraseña cifrada en PostgreSQL; credenciales erróneas rechazan el acceso sin abrir ventana principal.   | Base / S1  |
| RF-AUT-02 (Debe) | Controlar permisos por rol y titularidad bloqueando vistas y acciones no autorizadas en la capa de negocio (Actor: Sistema).  | Un empleado no puede acceder a catálogo, finanzas, aprobaciones de stock, retrocesos de estado ni órdenes/tareas de otros empleados.   | RF-AUT-01 / S1  |


### 5.2 Clientes


| **ID / MoSCoW** | **Descripción y Actor** | **Condición de Aceptación Observable** | **Dependencia / Sprint** |
| - | - | - | - |
| RF-CLI-01 (Debe) | Crear, consultar, actualizar y buscar fichas de clientes (Actor: Administrador).   | Con documento, nombre y teléfono válidos guarda la ficha; permite búsqueda rápida sin duplicar registros.   | RF-AUT-02 / S1   |
| RF-CLI-02 (Debe) | Consultar desde la ficha del cliente todos sus vehículos asociados (1:N) y su historial completo de órdenes (Actor: Administrador).   | La ficha muestra únicamente las motocicletas y órdenes pertenecientes al cliente seleccionado.   | RF-CLI-01, RF-VEH-01 / S1-S2   |


### 5.3 Vehículos e Historial


| **ID / MoSCoW** | **Descripción y Actor** | **Condición de Aceptación Observable** | **Dependencia / Sprint** |
| - | - | - | - |
| RF-VEH-01 (Debe) | Registrar y actualizar motocicletas vinculadas a un único cliente (1:N) con placa única normalizada (Actor: Administrador).   | Rechaza placas duplicadas y garantiza que cada moto tenga un único propietario registrado.   | RF-CLI-01 / S1   |
| RF-VEH-02 (Debe) | Consultar el historial cronológico de órdenes y diagnósticos de una motocicleta (Actor: Administrador / Empleado).   | Lista todas las órdenes y diagnósticos de la moto ordenados por fecha descendente sin mezclar otros vehículos.   | RF-VEH-01, RF-ORD-01 / S1-S2   |


### 5.4 Diagnóstico y Órdenes de Trabajo


| **ID / MoSCoW** | **Descripción y Actor** | **Condición de Aceptación Observable** | **Dependencia / Sprint** |
| - | - | - | - |
| RF-ORD-01 (Debe) | Crear órdenes en estado DIAGNÓSTICO registrando kilometraje, nivel de combustible y checklist físico de llegada, validando viabilidad y stock antes de pasar a PENDIENTE (Actor: Administrador / Empleado).  | No permite pasar de DIAGNÓSTICO a PENDIENTE si falta el checklist físico, empleado activo asignado o stock disponible.  | RF-VEH-01, RF-INV-01 / S2  |
| RF-ORD-02 (Debe) | Avanzar el estado de la orden en su ciclo normal: DIAGNÓSTICO -\> PENDIENTE -\> EN\_PROCESO -\> FINALIZADA -\> ENTREGADA (Actor: Empleado asignado / Administrador).  | El empleado asignado avanza hasta FINALIZADA; solo el Administrador marca ENTREGADA, dejando la orden en solo lectura.   | RF-ORD-01 / S2  |
| RF-ORD-03 (Debe) | Permitir exclusivamente al Administrador retroceder un estado por error humano del empleado, o pasar la orden a CANCELADA ante fallos sin solución o falta de recursos (Actor: Administrador).  | El empleado tiene bloqueado retroceder o cancelar; CANCELADA exige motivo, dispara reverso de piezas y bloquea edición.  | RF-ORD-02, RF-INV-04 / S3  |
| RF-ORD-04 (Debe) | Asignar órdenes a empleados activos y reasignar la carga al trabajador activo con menor ocupación del día en caso de inasistencia o baja (Actor: Administrador).  | Rechaza empleados inactivos; ante ausencias, identifica al empleado con menor carga diaria y transfiere la orden o reprograma entrega.   | RF-ORD-01 / S2-S3  |
| RF-ORD-05 (Debe) | Registrar el costo de mano de obra y calcular el total de la orden sumando mano de obra más piezas aprobadas a precio histórico (Actor: Administrador).   | El total coincide con la suma decimal de mano de obra y piezas aprobadas activas, congelándose al entregar.   | RF-INV-03 / S3   |





### 5.5 Inventario de Piezas (Repuestos y Accesorios)


| **ID / MoSCoW** | **Descripción y Actor** | **Condición de Aceptación Observable** | **Dependencia / Sprint** |
| - | - | - | - |
| RF-INV-01 (Debe) | Administrar el catálogo de piezas clasificadas en Repuestos y Accesorios con SKU único, categoría, costo de compra, precio de venta, stock actual y stock mínimo (Actor: Administrador).   | Rechaza SKU duplicados o montos negativos; toda pieza nueva inicia con stock igual a cero.   | RF-AUT-02 / S1   |
| RF-INV-02 (Debe) | Registrar entradas de unidades al inventario generando su movimiento de bitácora (Actor: Administrador).   | Una entrada entera positiva incrementa el stock exactamente en esa cantidad y registra fecha y usuario.   | RF-INV-01 / S1   |
| RF-INV-03 (Debe) | Permitir al mecánico solicitar piezas en una orden y al Administrador aprobar la salida descontando el stock en una sola transacción atómica (Actor: Empleado solicita / Administrador aprueba).  | La solicitud del mecánico no descuenta stock; al aprobar el Admin, si hay stock se descuenta y crea movimiento, si no hay hace rollback.   | RF-INV-01, RF-ORD-01 / S2  |
| RF-INV-04 (Debe) | Reintegrar automáticamente al stock todas las piezas aprobadas de una orden cuando esta pase a estado CANCELADA (Actor: Sistema / Administrador).  | Al cancelar la orden, suma las unidades de vuelta al stock de cada pieza y genera movimientos de tipo REVERSO.  | RF-INV-03, RF-ORD-03 / S3  |
| RF-INV-05 (Debería) | Señalar visualmente las piezas cuyo stock actual sea igual o menor al stock mínimo configurado (Actor: Administrador).   | Destaca con alerta visual en el catálogo y en el panel principal todo ítem con stock\_actual \<= stock\_minimo.   | RF-INV-01 / S1-S4  |


### 5.6 Tareas Diarias y Jornada Laboral


| **ID / MoSCoW** | **Descripción y Actor** | **Condición de Aceptación Observable** | **Dependencia / Sprint** |
| - | - | - | - |
| RF-TAR-01 (Debe) | Crear y reasignar tareas diarias con fecha, empleado activo responsable y prioridad ALTA, MEDIA o BAJA (Actor: Administrador).   | Crea la tarea en estado PENDIENTE y permite transferirla a otro empleado activo en caso de inasistencia.   | RF-AUT-02 / S3   |
| RF-TAR-02 (Debe) | Mostrar a cada empleado únicamente sus tareas del día ordenadas por prioridad descendente (Actor: Empleado).   | El empleado autenticado solo ve sus propias tareas del día en orden ALTA -\> MEDIA -\> BAJA.   | RF-TAR-01 / S3   |
| RF-TAR-03 (Debe) | Permitir al empleado actualizar el estado de sus propias tareas entre PENDIENTE, EN\_PROCESO y COMPLETADA (Actor: Empleado).   | Permite el cambio al titular de la tarea y bloquea modificaciones sobre tareas de otros empleados.   | RF-TAR-02 / S3   |


### 5.7 Ventas de Mostrador, Finanzas (OBJ-06) y Reportes


| **ID / MoSCoW** | **Descripción y Actor** | **Condición de Aceptación Observable** | **Dependencia / Sprint** |
| - | - | - | - |
| RF-VEN-01 (Debería) | Registrar ventas directas de mostrador de repuestos y accesorios descontando stock de forma atómica (Actor: Administrador).  | Descuenta las unidades vendidas sin exigir orden de trabajo, genera movimiento de inventario y registra el ingreso en caja.  | RF-INV-01 / S3  |
| RF-FIN-01 (Debería) | Gestionar ingresos, egresos, estado de caja, cuentas y estadísticas financieras del taller conforme al OBJ-06 (Actor: Administrador).  | Consolida ingresos (órdenes entregadas y ventas de mostrador) y egresos (compras de stock y gastos) mostrando el balance de caja.  | RF-ORD-05, RF-VEN-01 / S4  |
| RF-REP-01 (Debería) | Mostrar un panel de control (Dashboard) con el conteo de órdenes en sus 6 estados, alertas de stock bajo y resumen de caja (Actor: Administrador).   | Los indicadores del panel coinciden en tiempo real con las vistas consolidadas en PostgreSQL.   | RF-ORD-02, RF-FIN-01 / S4  |


## 6. PRD: Requisitos No Funcionales (RNF)


### 6.1 RNF-ENT-01 — Entorno de Ejecución Desktop


| **Prioridad** | **Descripción** | **Método de Verificación** | **Meta** |
| - | - | - | - |
| Debe | Aplicación de escritorio local en Java 21 LTS ejecutada directamente en el sistema operativo sin navegador web.  | Ejecución del artefacto empaquetado (.jar) conectado a PostgreSQL en localhost:5432.  | Arranque local limpio sin dependencias de internet ni servidores web externos.  |



### 6.2 RNF-SEG-01 — Seguridad, Roles y Credenciales


| **Prioridad** | **Descripción** | **Método de Verificación** | **Meta** |
| - | - | - | - |
| Debe | Validación de roles y titularidad en la capa de servicios Java (no solo ocultando botones en Swing) y contraseñas cifradas en base de datos.   | Pruebas de invocación con rol EMPLEADO sobre servicios exclusivos de ADMINISTRADOR e inspección de tabla de usuarios.   | 100% de operaciones críticas protegidas en capa de negocio y cero contraseñas en texto plano.   |


### 6.3 RNF-INT-01 — Integridad Transaccional de Stock y Caja


| **Prioridad** | **Descripción** | **Método de Verificación** | **Meta** |
| - | - | - | - |
| Debe | Aprobaciones de piezas, ventas de mostrador y reversos por cancelación ejecutados en transacciones ACID con bloqueo de fila y triggers.   | Simulación de stock insuficiente y cancelación de órdenes con piezas aprobadas.   | Cero saldos negativos (stock\_actual \>= 0) y rollback total ante fallos parciales.   |


### 6.4 RNF-TRA-01 — Trazabilidad y Auditoría


| **Prioridad** | **Descripción** | **Método de Verificación** | **Meta** |
| - | - | - | - |
| Debe | Bitácora inmutable de movimientos de inventario y movimientos financieros de caja registrando fecha, hora, tipo, monto/cantidad y usuario responsable.   | Cruce de saldos de stock y arqueo de caja contra las tablas de movimientos en PostgreSQL.   | 100% de variaciones de stock y de caja respaldadas por un registro de movimiento.   |





### 6.5 RNF-PRI-01 — Privacidad y Datos de Prueba


| **Prioridad** | **Descripción** | **Método de Verificación** | **Meta** |
| - | - | - | - |
| Debe | Uso exclusivo de datos ficticios en el desarrollo, pruebas y sustentación mediante script .sql semilla.   | Inspección del script de carga inicial y base de datos de demostración.   | Cero datos personales reales expuestos en el repositorio o en la demo.   |

### 6.6 RNF-USA-01 — Usabilidad y Mantenibilidad en Capas

| **Prioridad** | **Descripción** | **Método de Verificación** | **Meta** |
| - | - | - | - |
| Debe | Interfaz gráfica uniforme en Java Swing + FlatLaf con mensajes claros de validación y código separado en 4 capas (UI, Controladores, Servicios/Negocio y DAO/Datos).   | Revisión de estructura de paquetes en Maven y prueba de usabilidad en los formularios principales.   | Separación estricta entre vistas Swing y consultas SQL; compilación limpia con Maven.   |


## 7. Arquitectura del sistema


### 7.1 Vista de componentes (Arquitectura Desktop en 4 Capas)

**Arquitectura aprobada:** Aplicación de escritorio local en **Java 21 LTS** estructurada en 4 capas lógicas: Usuarios (Administrador o Empleado) -\> Capa de Presentación Desktop (Java Swing + FlatLaf) -\> Capa de Negocio, Servicios y Controladores de Eventos en Java -\> Capa de Acceso a Datos (JDBC / JPA Hibernate) -\> Base de Datos PostgreSQL Local (localhost:5432). Se elimina por completo el uso de navegador web, plantillas HTML o servidores HTTP externos. 


**Diagrama de arquitectura lógica (referencia textual para documentación):**


- **Capa 1 — Usuario:** Administrador del taller o Empleado (Mecánico/Operario) interactuando desde el PC local.


- **Capa 2 — Presentación (Frontend Desktop):** Ventanas JFrame, paneles JPanel, modales JDialog y tablas JTable estilizados con FlatLaf (construidos por Jhon con el diseñador visual del IDE).


- **Capa 3 — Controladores y Negocio (Backend Java 21):** Controladores de eventos Swing y Servicios de dominio con validación de sesión, permisos por rol y reglas de negocio (desarrollados por Neyer y Luisa).


- **Capa 4 — Persistencia y Datos (PostgreSQL Local):** Repositorios/DAO con transacciones ACID, apoyados en triggers, funciones PL/pgSQL y vistas en PostgreSQL en localhost:5432 (diseñados por Leider).


### 7.2 Componentes y despliegue local


| **Componente** | **Tecnología** | **Responsabilidad principal** | **Responsable / Ejecución** |
| - | - | - | - |
| Interfaz Gráfica (GUI) | Java Swing + FlatLaf | Ventanas, formularios, tablas y alertas visuales sin lógica de negocio ni SQL directo.  | Jhon — PC local del taller  |
| Controladores y Servicios | Java 21 LTS (Maven) | Manejo de eventos de UI, sesión activa, autorización por rol y coordinación transaccional.  | Neyer y Luisa — JVM local  |
| Acceso a Datos | JDBC / JPA (Hibernate) | Mapeo de entidades, ejecución de consultas parametrizadas y control de commit/rollback.  | Neyer y Luisa — JVM local  |
| Base de Datos y Lógica SQL | PostgreSQL (localhost:5432) | Esquema relacional, integridad 1:N, funciones de descuento atómico, triggers de reverso y vistas financieras.  | Leider — Servicio local en PC  |


### 7.3 Decisiones de arquitectura (ADR)

| **ADR** | **Decisión aprobada en v0.3** | **Alternativa descartada** | **Justificación y consecuencia** |
| - | - | - | - |
| ADR-01 | Aplicación de escritorio local en Java 21 con Swing + FlatLaf.  | Monolito web en Python/Django o SPA web.   | Ningún integrante tiene experiencia en diseño web; el diseñador Drag & Drop del IDE con FlatLaf ofrece interfaz moderna sin curva de aprendizaje de HTML/CSS.  |
| ADR-02 | PostgreSQL local con lógica pesada en BD (triggers, funciones y vistas).  | Base de datos pasiva o NoSQL.   | Permite que Leider descargue de trabajo a Java automatizando reversos de stock, validación de saldos y cálculos de caja.  |
| ADR-03 | Backend 2 (Neyer/Luisa) asume los Controladores de eventos de las pantallas Swing.  | Dejar toda la programación de eventos a Frontend.  | Evita que Jhon (único Frontend) sea cuello de botella; Jhon solo maqueta vistas y Backend conecta los botones y tablas.  |
| ADR-04 | Inventario categorizado en Piezas (Repuestos y Accesorios) con flujo Solicitud -\> Aprobación.  | Descuento directo sin supervisión o solo repuestos.   | El mecánico solicita en la orden y solo el Administrador aprueba la salida en transacción atómica.  |
| ADR-05 | Inclusión de Ventas de Mostrador y Trazabilidad Financiera de Caja (OBJ-06).  | Exclusión total de ventas y caja.   | Registra ingresos (OT entregadas + mostrador) y egresos (compras + gastos) para cuadrar la caja del local.  |


### 7.4 Estructura estándar del proyecto Maven


- **pom.xml:** Configuración de Java 21, dependencias de FlatLaf, driver PostgreSQL JDBC, JPA/Hibernate y JUnit.


- **src/main/java/com/soulmotors/config:** Conexión a localhost:5432 y contexto de sesión local.


- **src/main/java/com/soulmotors/model:** Entidades de dominio (Usuario, Cliente, Vehiculo, Diagnostico, OrdenTrabajo, Pieza, SolicitudPieza, MovimientoInventario, Tarea, VentaMostrador, MovimientoCaja).


- **src/main/java/com/soulmotors/dao:** Clases de acceso a datos e invocación de funciones/vistas SQL.


- **src/main/java/com/soulmotors/service:** Reglas de negocio, validación de roles y transacciones ACID (Neyer y Luisa).


- **src/main/java/com/soulmotors/controller:** Controladores de eventos que enlazan formularios Swing con los servicios (Neyer y Luisa).


- **src/main/java/com/soulmotors/ui:** Formularios JFrame, JPanel y JDialog en Swing + FlatLaf (Jhon).


- **sql/schema\_and\_seed.sql:** Script maestro de tablas, restricciones, triggers, vistas y datos ficticios (Leider).


## 8. Flujos de sistema, secuencias y reglas de negocio

Este capítulo especifica los **8 flujos operativos y financieros (FL-01 a FL-08)** alineados con las 16 Historias de Usuario de Soul Motors. 


| **Flujo** | **Sprint** | **Historia(s)** | **Acción principal y resultado verificable** |
| - | - | - | - |
| FL-01 — Autenticación y sesión local por rol | S1 | HU-01 | Validar credenciales en PostgreSQL, abrir sesión en memoria y habilitar menú según rol (Admin / Empleado).  |
| FL-02 — Clientes, motocicletas (1:N) e historial | S1 | HU-02, HU-03 | Registrar clientes, vincular motos con placa única (1 propietario por moto) y consultar su historial cronológico.  |
| FL-03 — Catálogo de piezas y entradas de stock | S1 | HU-04 | Crear piezas (Repuestos y Accesorios) con SKU único, registrar entradas y alertar stock mínimo.  |
| FL-04 — Recepción en DIAGNÓSTICO y paso a PENDIENTE | S2 | HU-05, HU-06 | Registrar kilometraje, nivel de combustible y checklist físico; validar viabilidad, empleado activo y stock antes de aceptar.  |
| FL-05 — Solicitud de piezas y aprobación atómica | S2 | HU-08 | El mecánico solicita piezas en la orden; el Administrador aprueba y descuenta el stock en una sola transacción.  |
| FL-06 — Avance normal, retroceso por error y CANCELADA | S2, S3 | HU-07, HU-09, HU-10 | Avanzar estados hasta ENTREGADA (calculando total); permitir retroceso (solo Admin) o CANCELADA con reverso automático de stock.  |
| FL-07 — Tareas diarias y reasignación por inasistencias | S3 | HU-11, HU-12 | Asignar tareas por prioridad, actualizar avance propio y transferir carga al empleado activo con menor ocupación ante bajas.  |
| FL-08 — Ventas de mostrador, caja y dashboard (OBJ-06) | S3, S4 | HU-13, HU-14, HU-15 | Vender piezas en mostrador descontando stock, registrar ingresos/egresos en caja y visualizar panel de estadísticas.  |

### 8.1 FL-01 — Autenticación y autorización en sesión local


**Actores:** Administrador o Empleado. | **Sprint:** S1. | **Historias:** HU-01. | **Pruebas:** CP-01, CP-02. **Precondiciones:** Base de datos PostgreSQL activa en localhost:5432 con usuarios preconfigurados y contraseñas cifradas. 


| **Paso** | **Responsable y acción** | **Validación en Capa de Negocio (Java)** | **Lecturas o escrituras en BD** |
| - | - | - | - |
| 1 | Usuario — Ingresa usuario y contraseña en la ventana de Login (UI-01).   | Verifica que los campos no estén vacíos antes de consultar la BD. | Sin escritura. |
| 2 | AuthService — Consulta usuario y valida el hash de la contraseña.  | Comprueba cuenta activa (activo = true), hash válido y rol asignado (ADMINISTRADOR o EMPLEADO).   | SELECT en tabla usuario. |
| 3 | SessionManager — Guarda el usuario autenticado en el contexto local de la aplicación. | Carga los permisos del rol y abre la ventana principal (UI-02) ocultando módulos no autorizados.   | Registro de último acceso opcional. |
| 4 | Usuario — Ejecuta cualquier acción en un formulario.   | El Servicio Java revalida el rol y la titularidad antes de ejecutar el DAO; si no tiene permiso lanza SecurityException.   | Bloquea cualquier INSERT/UPDATE no autorizado.   |
| 5 | Usuario — Pulsa "Cerrar sesión".   | Limpia el objeto SessionManager y retorna a la ventana de Login (UI-01).   | Sin cambios de negocio.   |


### 8.2 FL-02 — Gestión de clientes, motocicletas (1:N) e historial


**Actores:** Administrador (consulta de historial disponible para Empleado en diagnóstico/orden). | **Sprint:** S1. | **Historias:** HU-02, HU-03. | **Pruebas:** CP-03, CP-04, CP-07. 


| **Paso** | **Responsable y acción** | **Validación en Capa de Negocio (Java)** | **Lecturas o escrituras en BD** |
| - | - | - | - |
| 1 | Administrador — Busca cliente por documento, nombre o teléfono en UI-03.   | Filtra coincidencias para evitar duplicar fichas de clientes.   | SELECT en tabla cliente.   |
| 2 | Administrador — Crea o actualiza la ficha del cliente.   | Documento, nombre y teléfono obligatorios; rol ADMINISTRADOR requerido.   | INSERT o UPDATE en cliente.   |
| 3 | Administrador — Desde la ficha del cliente registra una motocicleta (placa, marca, modelo, año).   | Normaliza la placa en mayúsculas sin espacios; verifica que no exista otra moto con esa placa y la asocia al único cliente propietario.   | INSERT en vehiculo con cliente\_id (FK 1:N).   |
| 4 | Usuario — Selecciona un cliente o una motocicleta para ver su historial.   | Filtra estrictamente por cliente\_id o vehiculo\_id sin mezclar expedientes de otros dueños.   | SELECT en orden\_trabajo y diagnostico\_recepcion ordenados por fecha DESC.   |

### 8.3 FL-03 — Catálogo de piezas (Repuestos y Accesorios) y entradas de stock


**Actores:** Administrador. | **Sprint:** S1. | **Historias:** HU-04. | **Pruebas:** CP-09, CP-10, CP-15. 


| **Paso** | **Responsable y acción** | **Validación en Capa de Negocio (Java)** | **Lecturas o escrituras en BD** |
| - | - | - | - |
| 1 | Administrador — Registra una pieza en UI-08 indicando SKU, nombre, categoría (REPUESTO o ACCESORIO), costo de compra, precio de venta y stock mínimo.   | Rol ADMINISTRADOR; SKU único; costo, precio y stock mínimo mayores o iguales a cero; stock inicial = 0.   | INSERT en tabla pieza con stock\_actual = 0.   |
| 2 | Administrador — Registra una entrada de inventario (+N unidades) para una pieza activa.   | Cantidad entera estrictamente positiva (N \> 0).   | Inicia transacción ACID.   |
| 3 | InventarioService — Incrementa el stock de la pieza y registra el movimiento.   | Confirma ambas escrituras juntas y evalúa si persiste alerta de stock bajo (stock\_actual \<= stock\_minimo).   | UPDATE pieza (stock\_actual + N) + INSERT mov\_inventario (tipo ENTRADA).   |









### 8.4 FL-04 — Recepción en DIAGNÓSTICO y aceptación a PENDIENTE


**Actores:** Administrador y Empleado. | **Sprint:** S2. | **Historias:** HU-05, HU-06. | **Pruebas:** CP-05, CP-08. 


| **Paso** | **Responsable y acción** | **Validación en Capa de Negocio (Java)** | **Lecturas o escrituras en BD** |
| - | - | - | - |
| 1 | Administrador / Empleado — Selecciona la motocicleta y abre expediente en estado DIAGNÓSTICO.  | Verifica que el vehículo exista y captura el motivo de ingreso.   | INSERT en orden\_trabajo con estado = DIAGNÓSTICO.  |
| 2 | Empleado / Administrador — Completa la ficha de diagnóstico de recepción.  | Exige obligatoriamente: kilometraje actual, nivel de combustible y checklist físico de llegada (rayones, espejos, luces, carenado, etc.) más concepto de viabilidad técnica.  | INSERT en diagnostico\_recepcion vinculado a la orden.  |
| 3 | Administrador — Evalúa aceptar el trabajo para pasarlo a estado PENDIENTE.  | Comprueba 3 condiciones obligatorias: 1) Diagnóstico técnicamente viable y checklist completo, 2) Empleado activo asignado, y 3) Stock disponible en inventario para las piezas requeridas.  | SELECT de verificación en diagnostico\_recepcion, usuario y pieza.  |
| 4 | OrdenService — Aprueba el paso a PENDIENTE.  | Si falla cualquiera de las 3 condiciones, rechaza la transición mostrando la causa exacta.  | UPDATE orden\_trabajo SET estado = PENDIENTE, asignado\_a\_id.   |

### 8.5 FL-05 — Solicitud de piezas por mecánico y aprobación atómica por Administrador


**Actores:** Empleado asignado (solicita) y Administrador (aprueba). | **Sprint:** S2. | **Historias:** HU-08. | **Pruebas:** CP-12. 


| **Paso** | **Responsable y acción** | **Validación en Capa de Negocio (Java)** | **Lecturas o escrituras en BD** |
| - | - | - | - |
| 1 | Empleado asignado — Desde su orden (en PENDIENTE o EN\_PROCESO), selecciona las piezas requeridas y cantidades.  | El mecánico no descuenta inventario directamente; solo genera una solicitud en estado SOLICITADA.  | INSERT en detalle\_pieza\_orden con estado\_solicitud = SOLICITADA.  |
| 2 | Administrador — Revisa las solicitudes de piezas pendientes y pulsa "Aprobar salida de inventario".  | Rol exclusivo ADMINISTRADOR; invoca transacción atómica bloqueando las filas de las piezas implicadas.   | BEGIN + SELECT FOR UPDATE sobre pieza y orden\_trabajo.   |
| 3 | Función transaccional (BD / Java) — Verifica que stock\_actual \>= cantidad solicitada en cada ítem.   | Si el stock alcanza: congela precio\_unitario histórico, aprueba el detalle, descuenta stock\_actual y crea movimiento SALIDA\_ORDEN. Si no alcanza: ejecuta ROLLBACK total sin descontar nada.   | UPDATE detalle\_pieza\_orden (APROBADA) + UPDATE pieza + INSERT mov\_inventario + COMMIT.   |


### 8.6 FL-06 — Avance de estados, retroceso por error humano y CANCELADA con reverso automático


**Actores:** Empleado asignado y Administrador. | **Sprint:** S2 y S3. | **Historias:** HU-07, HU-09, HU-10. | **Pruebas:** CP-06, CP-11, CP-12R. 


| **Paso** | **Responsable y acción** | **Validación en Capa de Negocio (Java)** | **Lecturas o escrituras en BD** |
| - | - | - | - |
| 1 | Empleado asignado / Admin — Avanza la orden de PENDIENTE a EN\_PROCESO y, al concluir la reparación, a FINALIZADA.   | Solo el titular asignado o el Admin pueden avanzar; en FINALIZADA se bloquea la solicitud de nuevas piezas.   | UPDATE orden\_trabajo SET estado, finalizado\_en.   |
| 2 | Administrador — Registra costo de mano de obra y marca la orden como ENTREGADA.   | Exclusivo del ADMINISTRADOR; calcula Total = mano\_obra + SUM(cantidad \* precio\_unitario de piezas aprobadas), registra ingreso en caja y bloquea la orden en solo lectura.   | UPDATE orden\_trabajo (ENTREGADA) + INSERT mov\_caja (INGRESO\_ORDEN).   |
| 3 | Excepción A (Retroceso por error humano) — Empleado avanzó estado por equivocación.  | Únicamente el ADMINISTRADOR puede retroceder la orden al estado previo registrando la nota de corrección.  | UPDATE orden\_trabajo SET estado = estado\_anterior.  |
| 4 | Excepción B (Cancelación sobrevenida) — Fallo sin solución o falta de recursos técnicos/herramientas.  | Exclusivo del ADMINISTRADOR; exige motivo de cancelación y cambia el estado a CANCELADA.  | UPDATE orden\_trabajo SET estado = CANCELADA, motivo\_cancelacion.  |
| 5 | Trigger de Reverso Automático (PostgreSQL) — Se dispara al pasar la orden a CANCELADA.  | Reintegra automáticamente al stock todas las piezas aprobadas de esa orden, genera movimientos de tipo REVERSO\_CANCELACION, bloquea la orden en solo lectura y muestra el checklist físico de recepción para devolver la moto tal como ingresó.  | UPDATE pieza (stock\_actual + cantidad) + INSERT mov\_inventario (REVERSO).   |




### 8.7 FL-07 — Tareas diarias y reasignación de carga por inasistencias


**Actores:** Administrador y Empleado. | **Sprint:** S3. | **Historias:** HU-11, HU-12. | **Pruebas:** CP-13, CP-14, CP-16. 


| **Paso** | **Responsable y acción** | **Validación en Capa de Negocio (Java)** | **Lecturas o escrituras en BD** |
| - | - | - | - |
| 1 | Administrador — Crea tarea diaria indicando título, fecha, empleado activo y prioridad (ALTA, MEDIA o BAJA).   | Verifica empleado activo y guarda la tarea en estado PENDIENTE.   | INSERT en tabla tarea.   |
| 2 | Empleado — Abre su bandeja "Mis tareas del día" (UI-10) y actualiza el avance (PENDIENTE -\> EN\_PROCESO -\> COMPLETADA).   | Filtra solo las tareas del empleado en sesión ordenadas por prioridad (ALTA -\> MEDIA -\> BAJA) y valida titularidad al cambiar estado.   | SELECT y UPDATE en tabla tarea.   |
| 3 | Contingencia por Inasistencia (Resolución Q3) — Un empleado se ausenta en la jornada.  | El Administrador ejecuta la consulta de carga diaria; el sistema identifica al empleado activo con menor número de órdenes/tareas activas hoy y le transfiere la asignación (o permite reprogramar entrega con aviso al cliente).  | SELECT de empleado con menor carga + UPDATE en orden\_trabajo y tarea.  |


### 8.8 FL-08 — Ventas de mostrador, caja diaria y panel de estadísticas (OBJ-06)


**Actores:** Administrador. | **Sprint:** S3 y S4. | **Historias:** HU-13, HU-14, HU-15. | **Pruebas:** CP-15, CP-17 y pruebas financieras. 


| **Paso** | **Responsable y acción** | **Validación en Capa de Negocio (Java)** | **Lecturas o escrituras en BD** |
| - | - | - | - |
| 1 | Administrador — En Venta de Mostrador (UI-11) selecciona piezas (repuestos o accesorios) y cantidades para venta directa sin orden de trabajo.  | Verifica stock disponible en transacción atómica; descuenta stock, registra movimiento SALIDA\_MOSTRADOR y crea ingreso automático en caja.  | INSERT venta\_mostrador + UPDATE pieza + INSERT mov\_inventario + INSERT mov\_caja.  |
| 2 | Administrador — En Gestión de Caja (UI-12) registra egresos (compra de lote de piezas o gastos del local) y consulta el arqueo diario.  | Valida montos positivos y consolida el balance: Total Ingresos (OT entregadas + mostrador) menos Total Egresos (compras + gastos).  | INSERT en mov\_caja y SELECT sobre vistas financieras.  |
| 3 | Administrador — Abre el Panel de Control / Dashboard (UI-13).  | Consulta vistas precalculadas en PostgreSQL con el conteo de órdenes en sus 6 estados, piezas en stock mínimo y resumen financiero.  | SELECT sobre vistas SQL v\_resumen\_ordenes, v\_alertas\_stock y v\_estado\_caja.  |


### 8.9 Máquina de estados de la Orden de Trabajo (6 Estados) y Matriz de Permisos


| **Estado ** **actual** | **Transición permitida** | **Actor autorizado** | **Condiciones y efectos de negocio** |
| - | - | - | - |
| DIAGNÓSTICO | Aceptar -\> PENDIENTE | Solo ADMINISTRADOR | Exige kilometraje, combustible, checklist físico, viabilidad, empleado activo asignado y stock disponible.  |
| PENDIENTE | Iniciar -\> EN\_PROCESO | EMPLEADO asignado o ADMINISTRADOR | Requiere empleado titular activo y aprobación de piezas por el Admin si fueron solicitadas.  |
| EN\_PROCESO | Finalizar -\> FINALIZADA | EMPLEADO asignado o ADMINISTRADOR | Registra fecha de finalización; ya no permite solicitar más piezas.   |
| FINALIZADA | Entregar -\> ENTREGADA | Solo ADMINISTRADOR | Calcula total definitivo, registra el ingreso en caja y bloquea la orden en solo lectura.  |
| PENDIENTE / EN\_PROCESO / FINALIZADA | Retroceder 1 estado | Solo ADMINISTRADOR | Exclusivo para corregir un error humano del empleado al cambiar de estado.  |
| DIAGNÓSTICO / PENDIENTE / EN\_PROCESO | Cancelar -\> CANCELADA | Solo ADMINISTRADOR | Exige motivo; dispara trigger que devuelve automáticamente todas las piezas al stock, bloquea en solo lectura y respalda devolución con el checklist físico.  |
| ENTREGADA / CANCELADA | Ninguna (Terminales) | Nadie | Estados terminales inmutables (solo lectura).  |


### 8.10 Catálogo consolidado de Reglas de Negocio (RN)

- **RN-01 (Categorización de Piezas): Todo ítem de inventario se clasifica como REPUESTO o ACCESORIO y se maneja en cantidades enteras no negativas.**


- **RN-02 (Cardinalidad 1:N Cliente-Moto):** Un cliente puede poseer múltiples motocicletas, pero cada motocicleta pertenece a un único cliente y tiene placa única.


- **RN-03 (Diagnóstico y Checklist Obligatorio):** Toda orden nace en DIAGNÓSTICO registrando kilometraje, nivel de combustible y checklist físico de llegada.


- **RN-04 (Condición de Stock y Asignación):** Ninguna orden pasa de DIAGNÓSTICO a PENDIENTE sin stock suficiente y sin un empleado activo asignado.


- **RN-05 (Solicitud vs. Aprobación de Piezas):** El mecánico solo solicita piezas; únicamente el Administrador aprueba la salida física descontando el stock en transacción ACID.


- **RN-06 (Retroceso Restringido):** Solo el Administrador puede retroceder el estado de una orden ante error humano del empleado.


- **RN-07 (Cancelación con Reverso Automático):** Al pasar una orden a CANCELADA, todas las piezas aprobadas retornan automáticamente al stock mediante movimientos de REVERSO y la moto se devuelve según el checklist de entrada.


- **RN-08 (Inmutabilidad Terminal):** Las órdenes en estado ENTREGADA o CANCELADA son de solo lectura.


- **RN-09 (Reasignación por Inasistencia):** Ante la baja o ausencia de un empleado, sus órdenes/tareas del día se reasignan al empleado activo con menor carga laboral en esa jornada o se reprograman.


- **RN-10 (Trazabilidad Financiera):** Toda orden ENTREGADA, venta de mostrador o compra de stock genera su respectivo movimiento en caja con usuario y fecha.


## 9. Interfaces externas


### 9.1 Inventario de pantallas Desktop (Java Swing + FlatLaf)


Todas las pantallas son construidas por **Jhon** usando el diseñador visual del IDE y conectadas por **Neyer y Luisa** mediante los controladores de eventos:


| **Código** | **Nombre de Ventana / Panel (Swing)** | **Objetivo y componentes principales** | **Actor autorizado** | **Sprint** |
| - | - | - | - | - |
| UI-01 | LoginFrame | Autenticación de usuario y clave con tema FlatLaf.   | Ambos   | S1  |
| UI-02 | MainDashboardFrame | Ventana principal con menú lateral dinámico según rol.   | Ambos   | S1  |
| UI-03 | ClientesPanel | Búsqueda, tabla JTable y formulario de alta/edición de clientes.   | Administrador   | S1  |
| UI-04 | FichaClienteDialog | Vista de detalle del cliente con sus motos asociadas (1:N) e historial.   | Administrador   | S1 / S2  |
| UI-05 | VehiculosPanel | Registro de motocicleta, validación de placa única e historial por moto.   | Administrador / Empleado (consulta)   | S1 / S2  |
| UI-06 | InventarioPiezasPanel | Catálogo de Repuestos y Accesorios, entradas de stock y etiquetas rojas de stock mínimo.   | Administrador   | S1  |
| UI-07 | DiagnosticoRecepcionDialog | Apertura de orden en DIAGNÓSTICO: kilometraje, combustible, checklist físico y viabilidad.  | Administrador / Empleado  | S2  |
| UI-08 | OrdenesTrabajoPanel | Gestión de órdenes en sus 6 estados, asignación de mecánico, mano de obra, retroceso y cancelación.   | Ambos (acciones según rol)   | S2 / S3  |
| UI-09 | SolicitudPiezasDialog | Modal para que el mecánico solicite piezas y el Administrador apruebe la salida atómica.  | Empleado (solicita) / Admin (aprueba)  | S2  |
| UI-10 | TareasYReasignacionPanel | Creación de tareas por prioridad, bandeja personal del empleado y reasignación al empleado con menor carga.   | Ambos (según rol)   | S3  |
| UI-11 | VentaMostradorPanel | Venta directa de piezas sin orden de trabajo con descuento inmediato de stock.  | Administrador  | S3  |
| UI-12 | CajaFinanzasPanel | Registro de ingresos, egresos, estado de caja y cuentas del taller (OBJ-06).  | Administrador  | S4  |
| UI-13 | EstadisticasDashboardPanel | Tarjetas de conteo de los 6 estados de OT, alertas de stock y métricas financieras.   | Administrador   | S4  |


### 9.2 Interfaz de hardware y software

- **Hardware:** Computador estándar del taller con monitor, teclado y ratón. No requiere lectores externos de códigos ni hardware especializado. 


- **Software:** Java Runtime / JDK 21 LTS, librería FlatLaf integrada en el empaquetado Maven y motor PostgreSQL local ejecutándose en localhost:5432 mediante driver oficial PostgreSQL JDBC.


## 10. Contratos de Servicios Internos y Controladores de Eventos

(Nota arquitectónica v0.3: Al sustituirse la aplicación web por una aplicación de escritorio en Java 21, este capítulo reemplaza los antiguos "Contratos HTTP" del borrador v0.2 por los **Contratos de la Capa de Servicios y Controladores** que comunican las vistas Swing de Jhon con el Backend de Neyer y Luisa y la Base de Datos de Leider). 


### 10.1 Estándar de comunicación entre Capas (Swing -\> Controlador -\> 


### Servicio -\> DAO)

1. **Desacoplamiento visual:** Ningún componente de ***`com.soulmotors.ui`** (Jhon) contiene sentencias SQL ni lógica de validación de negocio.


2. **Escucha de eventos:** Los controladores en ***`com.soulmotors.controller`** (programados por Neyer y Luisa) registran los ***`ActionListener`** de los botones Swing, extraen los datos de los campos de texto y llaman al método correspondiente en ***`com.soulmotors.service`**.


3. **Manejo estandarizado de errores:** Si una regla de negocio o restricción de base de datos falla (ej. placa duplicada, stock insuficiente o permiso denegado), el Servicio lanza una excepción tipada (***`BusinessRuleException`** o ***`UnauthorizedActionException`**), ejecuta ***`ROLLBACK`** y el Controlador muestra un cuadro de diálogo ***`JOptionPane.showMessageDialog`** claro al usuario sin cerrar ni bloquear la aplicación.


### 10.2 Catálogo de Contratos de Servicios Principales


| **Servicio Java** | **Método principal** | **Rol requerido** | **Regla y efecto transaccional en PostgreSQL** |
| - | - | - | - |
| AuthService | autenticar(usuario, clave) | Público (Login) | Verifica hash en BD y carga sesión en SessionManager.  |
| ClienteVehiculoService | registrarVehiculo(clienteId, dtoMoto) | ADMINISTRADOR | Normaliza placa, verifica unicidad y guarda relación 1:N.  |
| OrdenService | abrirEnDiagnostico(vehiculoId, dtoChecklist) | ADMIN / EMPLEADO | Crea OT en estado DIAGNÓSTICO junto con su ficha de kilometraje, combustible y checklist físico.  |
| OrdenService | aceptarOrdenAPendiente(ordenId, empleadoId) | ADMINISTRADOR | Valida checklist completo, empleado activo y stock disponible antes de pasar a PENDIENTE.  |
| InventarioService | solicitarPieza(ordenId, piezaId, cantidad) | EMPLEADO asignado / ADMIN | Registra solicitud en estado SOLICITADA sin descontar stock.  |
| InventarioService | aprobarSalidaPieza(solicitudId) | ADMINISTRADOR | Ejecuta función transaccional: bloquea fila, verifica stock, congela precio, descuenta stock y crea movimiento SALIDA\_ORDEN.  |
| OrdenService | retrocederEstadoPorError(ordenId, motivo) | ADMINISTRADOR | Retrocede un paso el estado de la orden dejando registro de auditoría.  |
| OrdenService | cancelarOrden(ordenId, motivo) | ADMINISTRADOR | Cambia a CANCELADA; dispara trigger de reverso automático de todas las piezas aprobadas y bloquea la orden.  |
| JornadaService | reasignarPorInasistencia(empleadoAusenteId, fecha) | ADMINISTRADOR | Consulta el empleado activo con menor carga laboral del día y transfiere órdenes/tareas pendientes.  |
| FinanzasService | registrarVentaMostrador(listaItems) | ADMINISTRADOR | Descuenta stock en transacción ACID, crea movimiento de inventario y registra el ingreso en caja.  |




## *11. Modelado de base de datos — Modelo *

## *Conceptual, Lógico y Físico*


*El modelo de base de datos de **Soul Motors v0.3** se ejecuta en **PostgreSQL local (localhost:5432)** y es estructurado, programado e inicializado por **Leider**. Para descargar el peso de procesamiento de la capa Java y evitar latencias, Leider implementa restricciones estrictas (CHECK), disparadores (Triggers) de reverso automático, transacciones ACID nativas y vistas SQL precalculadas para el dashboard financiero. *


### *11.1 Diagrama Entidad-Relación Lógico (Descripción)*

*El sistema abandona las tablas predeterminadas de Django y construye un esquema relacional propio con **11 tablas de negocio principales**, respetando las reglas de la v0.3: *


- ***usuario:** Perfiles con rol estricto (ADMINISTRADOR / EMPLEADO) y credenciales cifradas. *


- ***cliente:** Propietarios de los vehículos. *


- ***vehiculo:** Motocicletas con relación 1:N hacia ***`cliente`*** y ***`placa`*** única. *


- ***diagnostico\_recepcion:** Ficha obligatoria de recepción (1:1 con la OT) con kilometraje, combustible y checklist físico. *


- ***orden\_trabajo (OT):** Cabecera de la orden con sus 6 estados (DIAGNÓSTICO, PENDIENTE, EN\_PROCESO, FINALIZADA, ENTREGADA, CANCELADA). *


- ***pieza:** Catálogo consolidado de REPUESTOS y ACCESORIOS con ***`stock_actual`*** y ***`stock_minimo`***. *


- ***detalle\_pieza\_orden:** Solicitudes del mecánico enlazadas a una OT, aprobadas atómicamente por el Administrador con precio histórico congelado. *


- ***movimiento\_inventario:** Bitácora inmutable de entradas, salidas aprobadas, ventas directas y reversos por cancelación. *


- ***tarea:** Jornada laboral diaria del empleado (ALTA, MEDIA, BAJA). *


- ***venta\_mostrador:** Registro directo de salidas de piezas sin requerir orden de trabajo. *


- ***movimiento\_caja:** Bitácora financiera para ingresos (órdenes entregadas, ventas mostrador) y egresos (compras, gastos). *


### *11.2 Diccionario de Tablas Principales (Extracto Lógico)*

***11.2.1 cliente y vehiculo (Relación 1:N)***


| **Tabla** | **Columna** | **Tipo / Restricción** | **Descripción** |
| - | - | - | - |
| cliente | id | BIGSERIAL / PK | Identificador único del propietario.   |
| cliente | documento | VARCHAR / UK | Cédula o NIT único para búsquedas.  |
| vehiculo | id | BIGSERIAL / PK | Identificador único de la moto.   |
| vehiculo | cliente\_id | BIGINT / FK | Pertenece a un solo cliente (RESTRICT).   |
| vehiculo | placa | VARCHAR(12) / UK | Placa única y normalizada en mayúsculas sin espacios.   |


***11.2.2 orden\_trabajo y diagnostico\_recepcion***


| **Tabla** | **Columna** | **Tipo / Restricción** | **Descripción** |
| - | - | - | - |
| orden\_trabajo | id | BIGSERIAL / PK | Número de la orden.   |
| orden\_trabajo | vehiculo\_id | BIGINT / FK | Moto atendida.   |
| orden\_trabajo | asignado\_a\_id | BIGINT / FK | Empleado responsable activo (Mecánico).   |
| orden\_trabajo | estado | VARCHAR | 6 estados de flujo (CHECK).   |
| orden\_trabajo | mano\_obra | NUMERIC(12,2) | Costo de servicio (\>= 0).   |
| diag\_recepcion | orden\_id | BIGINT / FK / UK | Relación 1:1 con la orden.   |
| diag\_recepcion | checklist\_fisico | TEXT | Notas del estado (rayones, luces, carenado).  |


***11.2.3 pieza y detalle\_pieza\_orden (Manejo Atómico)***


| **Tabla** | **Columna** | **Tipo / Restricción** | **Descripción** |
| - | - | - | - |
| pieza | id | BIGSERIAL / PK | Identificador de pieza.   |
| pieza | categoria | VARCHAR | CHECK (REPUESTO o ACCESORIO).   |
| pieza | stock\_actual | INTEGER | CHECK (\>= 0).   |
| detalle\_pieza\_or | estado\_solicitud | VARCHAR | CHECK (SOLICITADA, APROBADA).   |
| detalle\_pieza\_or | precio\_unitario | NUMERIC(12,2) | Precio histórico congelado al momento de aprobar.   |





***11.2.4 Financiero: movimiento\_caja (OBJ-06)***


| **Tabla** | **Columna** | **Tipo / Restricción** | **Descripción** |
| - | - | - | - |
| mov\_caja | id | BIGSERIAL / PK | Identificador financiero.   |
| mov\_caja | tipo\_movimiento | VARCHAR | INGRESO o EGRESO.  |
| mov\_caja | referencia\_origen | VARCHAR | 'ORDEN', 'MOSTRADOR', 'COMPRA', 'GASTO'.  |
| mov\_caja | monto | NUMERIC(12,2) | Valor positivo.   |


### *11.3 Integridad en PostgreSQL (Implementada por Leider)*

*Para garantizar la integridad ACID que exige el RNF-INT-01, Leider programa en el motor de BD: *


1. ***Transacciones Atómicas de Inventario (aprobacion\_piezas\_sp):** Un Procedimiento Almacenado o función PL/pgSQL que recibe el ID de la solicitud; internamente ejecuta ***`SELECT FOR UPDATE`*** sobre la fila de la pieza (bloqueo), verifica si ***`stock_actual >= cantidad`***, descuenta el saldo, crea el ***`movimiento_inventario`*** y hace ***`COMMIT`***. Si no hay stock, lanza excepción y revierte toda la operación (***`ROLLBACK`***). *


2. ***Trigger de Reverso Automático (trg\_reverso\_cancelacion):** Un disparador configurado en ***`orden_trabajo`***. Si el ***`estado`*** muta a ***`CANCELADA`***, el trigger lee todos los registros de ***`detalle_pieza_orden`*** de esa OT que estén como 'APROBADA', suma las cantidades de vuelta al ***`stock_actual`*** de cada pieza, e inserta registros de tipo 'REVERSO' en ***`movimiento_inventario`***. *


3. ***Vistas Consolidadas (Views) para el Dashboard:** Para que Jhon y Backend no tengan que iterar listas pesadas en memoria, Leider provee vistas como ***`v_dashboard_estados_ot`***, ***`v_alertas_stock`*** y ***`v_balance_caja`*** que entregan los totales matemáticos ya procesados al Dashboard (UI-13). *



## *12. Arquitectura de Seguridad (Desktop Local)*


*Dado que la aplicación se ejecuta localmente en la máquina del taller sin exposición a redes externas (Internet), las amenazas de red (como inyecciones SQL remotas o ataques DDoS) se mitigan por entorno. Sin embargo, la seguridad interna contra manipulaciones no autorizadas de empleados se mantiene firme en la Capa de Servicios Java. *


| **Amenaza Interna** | **Mecanismo de Control (Mitigación)** | **Verificación** |
| - | - | - |
| T-01: Empleado avanza orden que no le pertenece o aprueba piezas | El ***`OrdenService`** y ***`InventarioService`** en Java extraen el Rol de la sesión (***`SessionManager`**). Si el usuario es EMPLEADO, bloquean métodos administrativos lanzando ***`UnauthorizedActionException`** e impidiendo el ***`UPDATE`** en BD.   | Pruebas CP-02.   |
| T-02: Empleado entra al Panel Financiero | Los Controladores Swing destruyen o inhabilitan (***`setVisible(false)`**) los paneles ***`CajaFinanzasPanel`** y ***`VentaMostradorPanel`** si la sesión no es ADMIN.   | Pruebas visuales en UI-02.   |
| T-03: Robo de contraseñas de la base de datos | Leider almacena las contraseñas en PostgreSQL mediante funciones criptográficas de un solo sentido (Hash SHA-256 o BCrypt) desde el script de datos semilla; ninguna contraseña está en texto plano.   | Inspección de tabla ***`usuario`**.   |
| T-04: Modificación del precio total de la orden por el operario | El cálculo del total se hace sumando ***`mano_obra`** + las líneas históricas de ***`detalle_pieza_orden`** en servidor/BD. Los campos numéricos en la interfaz del operario son estrictamente ***`setEditable(false)`**.   | Pruebas CP-11.   |


## *13. Empaquetado y Despliegue Local (JAR)*

*La transición a Desktop elimina los despliegues en la nube, dominios o hosting web. El taller no requiere conexión a internet para operar. *


- ***Artefacto de compilación:** Archivo ejecutable único ***`.jar`*** (fat-jar o uber-jar) generado por **Maven** que incluye el bytecode de Java 21, la dependencia visual ***`FlatLaf`*** y el driver ***`PostgreSQL JDBC`***. *


- ***Configuración de base de datos:** El archivo de configuración de conexión (***`application.properties`*** o equivalente) apunta directamente a ***`jdbc:postgresql://localhost:5432/soulmotors_db`*** usando el puerto estándar local. *


- ***Requisitos del PC Local:***


  - *Sistema Operativo: Windows 10/11 o Linux. *


  - *Motor: PostgreSQL Server ejecutándose en segundo plano (***`services.msc`*** en Windows). *


  - *Ejecución: JDK o JRE 21 instalado (ejecución mediante doble clic o ***`java -jar SoulMotors.jar`***). *


## *14. Calidad y Manejo de Errores*

| **Componen** **te** | **Regla de Calidad y Manejo** |
| - | - |
| **Java Swing (UI)** | **Jhon** garantiza el uso del administrador de diseño de su IDE para que los paneles no se deformen al maximizar. Toda excepción capturada por el Controlador muestra un ***`JOptionPane.showMessageDialog`** amigable en español, evitando que la consola local imprima StackTraces al usuario final.  |
| **Java Backend** | **Neyer y Luisa** envuelven las operaciones CRUD y transaccionales en bloques ***`try/catch`**. Si Hibernate o JDBC detectan una violación (ej. FK constraint en PostgreSQL), el servicio lanza una excepción tipada para que la vista notifique el error específico (ej. "La motocicleta no puede ser eliminada porque tiene órdenes registradas").   |
| **BD PostgreSQL** | **Leider** configura ***`ON DELETE RESTRICT`** (RN-14) en las llaves foráneas para impedir que se borren clientes con histórico, y utiliza restricciones ***`CHECK (stock_actual >= 0)`** para que un fallo en Java nunca resulte en saldos negativos matemáticos en disco.   |


## *15. Verificación y Casos de Prueba (CP) Actualizados*

*Los Casos de Prueba (CP) son ejecutados por el Product Owner (Josue) para validar el avance cada 3 días y autorizar el cierre del Sprint.*


### *15.1 Casos de Prueba Críticos (Core Operativo)*

| **ID** | **Descripción de Prueba** | **Sprint / HU** | **Criterio de Aceptación (Aprobación)** |
| - | - | - | - |
| **CP-01** | Inicio y restricción de sesión local | S1 (HU-01) | Usuario válido ingresa al MainFrame; roles EMPLEADO ocultan módulo de inventario y caja.   |
| **CP-02** | Registro y placa única (1:N) | S1 (HU-03) | La interfaz impide asignar una misma placa a dos clientes distintos y persiste 1 único dueño.   |
| **CP-03** | Apertura y checklist obligatorio | S2 (HU-05) | Una orden nace en DIAGNÓSTICO. El sistema no permite aceptar la orden (pasar a PENDIENTE) sin el checklist de llegada, el km y combustible.   |
| **CP-04** | Aprobación atómica de piezas | S2 (HU-08) | Admin aprueba solicitud del mecánico: El stock se descuenta en BD, se congela el precio histórico y se crea el movimiento. Si falta stock, alerta error y no restaforma.   |
| **CP-05** | Retroceso exclusivo de Admin | S3 (HU-09) | Ante un error de estado, un EMPLEADO tiene bloqueado retroceder. Solo ADMIN presiona "Retroceder" y retorna la orden al estado previo.   |
| **CP-06** | Cancelación y reverso (Triggers) | S3 (HU-09) | ADMIN marca orden "CANCELADA" registrando motivo. El sistema devuelve automáticamente las piezas aprobadas al stock (movimientos de REVERSO) y la bloquea en solo lectura.   |
| **CP-07** | Reasignación laboral por ausencia | S3 (HU-11) | ADMIN consulta carga y transfiere órdenes PENDIENTES del ausente al mecánico activo con menos carga del día.   |
| **CP-08** | Cuadre financiero (OBJ-06) | S4 (HU-14) | En el Dashboard, los ingresos (órdenes ENTREGADAS y mostrador) menos egresos (compras) dan el saldo exacto de caja.   |
| **CP-09** | Bloqueo por Inmutabilidad | S4 (HU-07) | Las órdenes ENTREGADAS o CANCELADAS no permiten edición de piezas, responsables ni estados en los campos de interfaz (solo lectura).   |




## *16. Acuerdos y Decisiones Internas (Checklist Resuelto del PO)*


*En este apartado se deja constancia de las decisiones rectoras asumidas por el Product Owner (Josue) y el equipo:*


| **Asunto o Pregunta** | **Decisión o Acuerdo Alcanzado (Versión 0.3)** |
| - | - |
| **P-01: ¿Qué entorno físico usamos?** | Se descarta el formato web (Django). El proyecto se compila y ejecuta 100% como App de Escritorio local (Java + FlatLaf + PostgreSQL).   |
| **P-02: ¿Cómo mitigar cuellos de botella?** | Leider absorbe validaciones y lógicas pesadas directamente en BD (PostgreSQL); Neyer y Luisa asumen Backend y Controladores Visuales Swing. Jhon se dedica exclusivamente a diseñar las interfaces gráficas con Drag & Drop.  |
| **P-03: ¿Ventas de Mostrador y Caja (OBJ-06)?** | Se confirma su inclusión en los Sprints 3 y 4. Permite vender repuestos sin abrir una orden de trabajo (descontando stock) y registrar los ingresos para un arqueo de caja básico, pero **no** se emiten facturas legales DIAN ni se calculan nóminas complejas.  |
| **P-04: ¿Datos de la demostración (Resolución Q6)?** | Se utilizará el script de BD inicial de Leider con datos ficticios suficientes para mostrar historiales, catálogos completos y un tablero de caja con movimientos, cuidando la privacidad.  |
| **P-05: ¿Revisiones de calidad (Resolución Q2)?** | El Product Owner (Josue) liderará revisiones periódicas cada 3 días a través de demostraciones locales, previniendo los atrasos típicos de Sprints académicos largos.  |
|  |  |


## *17. Glosario*

- ***Atómico (Transacción ACID):** Propiedad de la base de datos que garantiza que un proceso de varios pasos (ej. restar stock, guardar precio, crear bitácora) se complete íntegramente. Si un solo paso falla, se revierte todo (Rollback), evitando datos corruptos o saldos incorrectos. *


- ***Checklist Físico:** Formulario de inspección de llegada (rayones, luces rotas, espejos). Sirve de garantía legal del taller para devolver la moto al cliente en caso de cancelación sobrevenida, evitando reclamos por daños ajenos.*


- ***FlatLaf:** Librería moderna ("Look and Feel") de Java Swing que proporciona un diseño limpio, plano y oscuro/claro (estilo Material Design), eliminando el aspecto anticuado del software Java tradicional de los años 90.*


- ***Reverso Automático:** Acción programada (Trigger) en PostgreSQL que "deshace" contablemente un consumo de piezas, regresando la cantidad descontada de vuelta a la estantería del taller sin requerir digitación humana manual. *


- ***Swing (Java Swing):** Biblioteca gráfica nativa de Java para el desarrollo de ventanas de escritorio (JFrames, JPanels, JTables). Se programa con facilidad arrastrando componentes en el IDE.*


*El **Bloque 4** de la versión 0.3 de **Soul Motors** (Capítulos 18 al 27: Parte III — Scrum) reemplaza las estimaciones anónimas originales por la matriz real del equipo, integrando explícitamente a **Josue (PO)**, **Yohely (SM)**, **Neyer y Luisa (Backend)**, **Jhon (Frontend)** y **Leider (BD)**. También adapta las 16 Historias de Usuario, el cálculo de horas y el Sprint Backlog a la nueva arquitectura local y a los Sprints de desarrollo. *


Aquí tienes el texto limpio, sin marcas de código LaTeX, listo para pegar directamente en tu **LibreOffice Writer**.


## *Parte III — Ejecución y Gestión (Scrum)*

### *18. Organización del equipo y roles (Matriz Nominal v0.3)*

*El equipo de **Soul Motors** consta de 6 integrantes confirmados. La arquitectura de escritorio local permite una división funcional clara para evitar cuellos de botella sobre Frontend, asignando validaciones de base de datos a Leider y conexión de eventos a Neyer y Luisa. *


| **Integrante** | **Rol Oficial Scrum** | **Responsabilidades Técnicas y de Negocio** |
| - | - | - |
| **Josue** | **Product Owner** | Validación de incrementos cada 3 días, priorización del backlog, resolución de dudas operativas (ej. OBJ-06), aprobación de UAT y aceptación final del producto.   |
| **Yohely** | **Scrum Master / PM** | Facilitación de ceremonias, vigilancia del flujo de Sprints, gestión de impedimentos y protección de la capacidad del equipo.   |
| **Leider** | **Base de Datos / Autor** | Diseño relacional en PostgreSQL local (1:N), funciones ACID de inventario, triggers de reverso automático, vistas financieras y script maestro (seed) con datos ficticios.   |
| **Neyer** | **Backend Java (1)** | Lógica Java 21, Auth, Clientes, Motos, flujo central de Órdenes y Tareas; configuración de Maven y conexión JPA/JDBC.   |
| **Luisa** | **Backend Java (2)** | Servicios de Inventario, transacciones de piezas, Ventas de Mostrador, Finanzas de Caja y conexión de Controladores de Eventos Swing.   |
| **Jhon** | **Frontend (UI Desktop)** | Construcción exclusiva de interfaces visuales con Java Swing y FlatLaf mediante diseñador Drag & Drop (cero lógica SQL en pantallas).   |


### *19. Product Backlog — Historias de Usuario completas (Adaptación v0.3)*


#### *19.1 Sprint 1 (Base Operativa y Entorno)*


| **ID** | **Descripción de Historia de Usuario (HU)** | **Criterios de Aceptación (CA) Resumidos** |
| - | - | - |
| **HU-01** | Como Administrador/Empleado, quiero **iniciar sesión en la app local** para operar bajo mis permisos.   | CA1: Valida credencial cifrada en BD. CA2: Perfil Empleado oculta paneles financieros e inventario completo.   |
| **HU-02** | Como Administrador, quiero **gestionar la ficha de los clientes** para tener sus datos de contacto.   | CA1: Búsqueda y alta con documento único sin lanzar StackTraces visibles al usuario.   |
| **HU-03** | Como Administrador, quiero **asociar motocicletas a un cliente (1:N)** para mantener registro.   | CA1: Rechaza placa duplicada. CA2: Cada moto queda ligada estrictamente a un único cliente.   |
| **HU-04** | Como Administrador, quiero **gestionar el catálogo de piezas (Repuestos/Accesorios)** y registrar entradas.   | CA1: Crea pieza con SKU único y stock=0. CA2: Entradas incrementan stock y crean movimiento.   |


#### *19.2 Sprint 2 (Recepción, Flujo Central y Solicitudes)*


| **ID** | **Descripción de Historia de Usuario (HU)** | **Criterios de Aceptación (CA) Resumidos** |
| - | - | - |
| **HU-05** | Como Administrador, quiero **abrir una OT en DIAGNÓSTICO** completando el checklist físico inicial.   | CA1: Registra km, combustible y checklist obligatorio para existir.   |
| **HU-06** | Como Administrador, quiero **aceptar la orden a PENDIENTE** y asignarla a un mecánico.   | CA1: Bloquea transición si no hay empleado activo o si falta viabilidad técnica.   |
| **HU-07** | Como Empleado asignado, quiero **avanzar mi orden (EN\_PROCESO -\> FINALIZADA)**.   | CA1: Avanza secuencialmente. CA2: Rechaza avance si usuario no es el titular de la OT.   |
| **HU-08** | Como Empleado, quiero **solicitar piezas** y, como Administrador, quiero **aprobarlas atómicamente**.   | CA1: Solicitud no descuenta stock. CA2: Aprobación del Admin descuenta BD y crea movimiento.   |


#### *19.3 Sprint 3 (Contingencias, Caja y Jornada)*


| **ID** | **Descripción de Historia de Usuario (HU)** | **Criterios de Aceptación (CA) Resumidos** |
| - | - | - |
| **HU-09** | Como Administrador, quiero **retroceder un estado** (error) o **CANCELAR la orden** (fuerza mayor).   | CA1: Cancelación ejecuta Trigger de reverso exacto de stock y muestra checklist de salida.   |
| **HU-10** | Como Administrador, quiero **marcar ENTREGADA** cobrando mano de obra y piezas.   | CA1: Calcula subtotal exacto con precios históricos congelados. CA2: Orden pasa a solo lectura.   |
| **HU-11** | Como Administrador, quiero **crear tareas diarias** (ALTA/MEDIA/BAJA) y asignarlas.   | CA1: Tareas se asocian a fecha y empleado activo. Empleado cambia estado a COMPLETADA.   |
| **HU-12** | Como Administrador, quiero **reasignar carga por inasistencia** del empleado.   | CA1: Identifica al empleado activo con menos tareas del día y le transfiere el pendiente.   |
| **HU-13** | Como Administrador, quiero **vender piezas por mostrador** sin abrir OT.   | CA1: Descuenta stock transaccional y genera ingreso en módulo financiero.   |


#### *19.4 Sprint 4 (Finanzas, Dashboard y Cierre)*


| **ID** | **Descripción de Historia de Usuario (HU)** | **Criterios de Aceptación (CA) Resumidos** |
| - | - | - |
| **HU-14** | Como Administrador, quiero **gestionar los movimientos de caja** (Ingresos y Egresos).   | CA1: Ingresos = Ventas + OT entregadas. CA2: Arqueo exacto sin decimales erróneos.   |
| **HU-15** | Como Administrador, quiero ver el **Dashboard estadístico y alertas de stock**.   | CA1: Vistas SQL precalculadas de los 6 estados y destacando ítems en stock mínimo.   |
| **HU-16** | Como Equipo, queremos **aprobar casos de prueba** y entregar el MVP a Josue.   | CA1: Estabilidad total, demostración desde .jar y script DDL validado por Leider.   |



### *20. Planificación de los cuatro sprints (Sprint Goals)*

*Cada sprint (S) consta de una iteración técnica con revisión cada 3 días liderada por **Josue (PO)**. *


| **Sprint** | **Sprint Goal Oficial (Meta)** | **Responsabilidad Principal Destacada** |
| - | - | - |
| **S1** | **Base y Entorno:** El equipo levanta el repositorio Maven Java 21, base de datos local y entrega ventanas funcionales de autenticación, alta de cliente, vinculación de motos y registro de catálogo. | Leider (Esquema SQL), Neyer (Auth/Clientes), Jhon (UI Swing).   |
| **S2** | **Recepción y Solicitudes:** La moto ingresa con checklist físico en DIAGNÓSTICO. Se aprueban solicitudes de piezas descontando stock atómicamente y se avanza la orden. | Luisa (Transacciones Piezas), Neyer (Flujo OT), Leider (SP Stock).   |
| **S3** | **Contingencias y Mostrador:** Se prueba el reverso de stock por CANCELADA, se organiza la jornada de tareas con reasignación por bajas, y se cobran órdenes y ventas de mostrador. | Leider (Triggers Reverso), Neyer (Tareas), Luisa (Mostrador).   |
| **S4** | **Caja, Dashboard y QA:** Se calculan las vistas financieras en BD, se integran las estadísticas al Dashboard principal y se corrige cualquier defecto detectado por Josue y Yohely para la sustentación. | Luisa (Finanzas Java), Leider (Vistas BD), Jhon (Pulido UI).   |


### *21. Sprint Backlog Asignado (Sprints 1 y 2)*


| **Tarea (Sprint 1)** | **Descripción** | **Responsable (Horas Est.)** |
| - | - | - |
| **T1-01** | Script DDL inicial (Usuarios, Clientes, Vehiculos, Piezas). | Leider (4h)   |
| **T1-02** | Proyecto Maven, conexión JDBC e Inicio de Sesión Java. | Neyer (6h)   |
| **T1-03** | Formularios UI de Login, Ficha Cliente, Vehículos y Catálogo. | Jhon (12h)   |
| **T1-04** | Servicios Java (CRUD Clientes y Motos 1:N). | Neyer (8h)   |
| **T1-05** | Servicios Java (CRUD Catálogo y Entradas de Inventario). | Luisa (8h)   |
| **T1-06** | Enlace de botones UI (Controladores Swing) para T1-04 y T1-05. | Neyer y Luisa (4h)   |
| **T1-07** | Revisión funcional de incrementos y casos CP-01 a CP-04. | Josue y Yohely (3h)   |

| **Tarea (Sprint 2)** | **Descripción** | **Responsable (Horas Est.)** |
| - | - | - |
| **T2-01** | Script de BD para Diagnóstico (Checklist) y Detalle\_Piezas. | Leider (4h)   |
| **T2-02** | Pantallas UI de Diagnóstico, Órdenes y Modal de Piezas. | Jhon (10h)   |
| **T2-03** | Servicio Java: Apertura en DIAGNÓSTICO y avance de estados. | Neyer (8h)   |
| **T2-04** | Función SQL (Stored Procedure) para Descuento Atómico de Stock. | Leider (5h)   |
| **T2-05** | Servicio Java: Solicitud de Mecánico y Aprobación de Administrador. | Luisa (8h)   |
| **T2-06** | Enlace de eventos Swing para Diagnóstico y Aprobación. | Neyer y Luisa (5h)   |
| **T2-07** | Revisión operativa de flujo central y consistencia de BD. | Josue y Yohely (4h)   |


### *22. Capacidad y Distribución de Esfuerzo (Velocidad)*


*La dedicación esperada por desarrollador se estima en **~20 horas brutas semanales** (incluyendo codificación, reuniones y corrección de pruebas). *


- ***Josue (PO) y Yohely (SM):** Invierten ~10h semanales en refinamiento, priorización, atención de consultas y verificación de pruebas cruzadas. *


- ***Leider (BD):** Esfuerzo concentrado al inicio de cada sprint (DDL y Triggers), apoyando en S4 la creación de Vistas analíticas. *


- ***Jhon (Frontend):** Carga sostenida dibujando interfaces. Nunca debe retrasarse esperando APIs, usando datos "quemados" temporalmente si Backend se atrasa. *


- ***Neyer y Luisa (Backend):** Alta exigencia en S2 y S3 implementando la lógica Java y enlazando los ActionListener de la UI creada por Jhon. *


### *23. Eventos Scrum Adaptados*


| **Evento** | **Frecuencia** | **Asistentes obligatorios** | **Objetivo Principal** |
| - | - | - | - |
| **Sprint Planning** | Día 1 (Lunes) | Josue, Yohely, Devs | Seleccionar tareas técnicas y confirmar dependencias del tablero.   |
| **Daily Standup** | Diario (15 min) | Yohely, Devs | Identificar impedimentos rápidos. (Ej. "Jhon necesita el modelo de Neyer para enlazar la tabla").   |
| **Punto de Control PO** | **Cada 3 días** | Josue y Dev implicado | Ejecución del RNF-OPE-01 (Demostración local). Josue verifica si la función cumple o debe iterarse (Resolución Q2).   |
| **Sprint Review/Retro** | Día 5 (Viernes) | Todo el equipo | Josue acepta o rechaza HU, Leider confirma integridad SQL, Yohely dirige retrospectiva.   |

### *24. Definition of Ready (DoR) y Definition of Done (DoD)*

***Definition of Ready (DoR) - Para iniciar una historia:***


1. *Josue (PO) ha resuelto las dudas operativas (ej. fórmula del total, regla de prioridad). *


2. *Leider ha entregado o acordado el nombre de las tablas y campos SQL a impactar. *


3. *Jhon comprende los datos que debe mostrar el formulario Swing. *


***Definition of Done (DoD) - Para cerrar una historia:***


1. *Código Java compilado sin errores en el empaquetado Maven. *


2. *Las operaciones sobre PostgreSQL utilizan Transacciones (***`COMMIT/ROLLBACK`***). *


3. *La interfaz Swing se actualiza correctamente tras la acción. *


4. ***Josue (PO)** verificó localmente el Caso de Prueba y dio su aprobación formal. *


### *25. Riesgos Específicos y Mitigación (Yohely / SM)*


| **ID** | **Riesgo Identificado** | **Impacto** | **Estrategia de Mitigación (Yohely / Equipo)** |
| - | - | - | - |
| **R-01** | Cuello de botella en Jhon (Frontend).   | Crítico | Jhon solo arrastra componentes visuales; Neyer y Luisa se encargan de programar los botones (ADR-03).   |
| **R-02** | Saldos negativos en ventas concurrentes.   | Alto | Leider asegura la BD con ***`CHECK (stock_actual >= 0)`**. Java implementa ***`try/catch`** para capturar la excepción y mostrar alerta visual (RNF-INT-01).   |
| **R-03** | Retraso en integración a S4   | Alto | Las revisiones cada 3 días de Josue evitan el "síndrome del estudiante"; se revisa software funcionando, no código fuente.   |
| **R-04** | Fallo crítico por pérdida de datos locales.   | Crítico | Leider mantiene el archivo ***`schema_and_seed.sql`** actualizado en el repositorio (Git) para reconstruir la BD en 1 minuto en cualquier máquina.   |


### *26. Seguimiento Diario*


***Yohely (Scrum Master)** gestiona el tablero Kanban virtual o físico.*


- ***Estados:** ***`To Do`*** -\> ***`En Desarrollo`*** -\> ***`En Pruebas (Josue)`*** -\> ***`Hecho`***.*


- ***Regla estricta:** Ningún desarrollador puede tener más de 2 tareas en ***`En Desarrollo`*** simultáneamente para forzar el cierre de entregables.*


- *Si una tarea acumula más de **2 días** bloqueada, Yohely interviene solicitando a Neyer/Luisa/Leider apoyo cruzado o pidiendo a Josue que recorte alcance de esa historia (Resolución Q4). *


### *27. Entregables Formales del MVP (Cierre)*


*Al finalizar el Sprint 4, el equipo entregará a evaluación los siguientes artefactos aprobados:*


1. ***Artefacto Ejecutable (.jar):** Aplicación de escritorio SoulMotors compilada. *


2. ***Script SQL Semilla (.sql):** Archivo maestro generado por Leider para crear y poblar la base de datos de demostración en ***`localhost:5432`***. *


3. ***Documentación Técnica SRS (v1.0):** Este mismo documento actualizado. *


4. ***Repositorio Centralizado (Git/ZIP):** Código fuente en Java estructurado en las 4 capas definidas. *


5. ***Acta de Aceptación:** Documento firmado por **Josue (PO)** certificando el cumplimiento del Flujo Central (OBJ-01 a OBJ-06) tras la demostración local final. *


*Aquí tienes el **Bloque 5** y final de la versión 0.3 de **Soul Motors**, Josue. Este bloque abarca la Parte IV (Auditoría) y los Anexos con los diagramas y el código SQL de PostgreSQL ajustados a la nueva arquitectura local y al modelo de 6 estados que hemos venido trabajando.*


Al igual que en los bloques anteriores, he limpiado el formato para que al pegarlo en **LibreOffice Writer** mantenga la estructura de títulos y viñetas sin romperse. (Los bloques de código Mermaid y SQL conservan su formato estándar para que puedas copiarlos y pegarlos en tus herramientas de modelado).


## *Parte IV — Auditoría y Cierre Documental*

### *28. Validación cruzada y Criterios de Preparación (DoR Final)*

*Antes de iniciar la codificación del Sprint 1, el equipo de Soul Motors ha validado la coherencia entre los flujos, la arquitectura y los requisitos, confirmando los siguientes puntos de control:*


1. ***Alineación de Entorno:** Se abandona definitivamente el esquema web. Jhon, Neyer y Luisa confirman que desarrollarán las interfaces y la lógica en **Java Swing + FlatLaf**, mientras Leider construirá la base de datos en **PostgreSQL local**.*


2. ***Privacidad y Datos Reales (Resolución Q6):** Queda estrictamente prohibido usar datos reales de clientes del taller durante el desarrollo. Leider proveerá un script ***`seed.sql`*** con datos ficticios para todas las demostraciones.*


3. ***Roles y Autorización:** Las validaciones de rol (Admin vs. Empleado) no se limitan a ocultar botones en la interfaz de usuario; Neyer y Luisa garantizan que la **Capa de Servicios Java** revalide el rol antes de ejecutar cualquier operación en la base de datos. *


4. ***Flujo de Caja y Mostrador:** Se confirma la inclusión del objetivo financiero (OBJ-06). Luisa gestionará los ingresos por entregas y ventas directas, y Leider proveerá las vistas matemáticas en PostgreSQL para cuadrar la caja.*


### *29. Resultado de la evaluación de línea base (v0.3)*

*Este documento (SRS-TM-001 v0.3) ha sido revisado y ajustado colaborativamente. Constituye el **Contrato de Alcance Técnico (TRD) y de Producto (PRD)** válido para los 4 Sprints del equipo Soul Motors. Toda función no descrita aquí (como nóminas, pasarelas de pago online o facturación electrónica DIAN) queda fuera del MVP.*


***Aprobación final del Product Owner:***


- ***Nombre:** Josue (Product Owner Titular)*


- ***Decisión:** APROBADO CON PASE A DESARROLLO (SPRINT 1)*


- ***Fecha:** 29 de octubre de 2026*


## *ANEXO A. REFERENCIAS TÉCNICAS*

- ***Java 21 LTS:** Entorno de ejecución principal y lenguaje de backend.*


- ***Java Swing & FlatLaf:** Librería gráfica nativa apoyada en FlatLaf para un diseño moderno (Material Design).*


- ***PostgreSQL 17:** Motor de base de datos relacional desplegado en ***`localhost:5432`***. *


- ***Apache Maven:** Gestor de dependencias y empaquetado del artefacto ***`.jar`*** final.*


- ***ISO/IEC/IEEE 29148:2018:** Estructura base para especificación de requisitos. *
