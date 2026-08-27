# 🍕 Sistema de Gestión de Pedidos y Domicilios - Pizzería Don Piccolo

Sistema de base de datos relacional desarrollado en **MySQL** para la automatización, control y optimización de las operaciones de venta, inventario, repartos y facturación de la **Pizzería Don Piccolo**.

---

## 📋 Tabla de Contenido
1. [Descripción del Proyecto](#-descripción-del-proyecto)
2. [Estructura del Proyecto](#-estructura-del-proyecto)
3. [Modelo de Datos y Relaciones](#-modelo-de-datos-y-relaciones)
4. [Lógica de Negocio Automática](#-lógica-de-negocio-automática)
   - [Funciones y Procedimientos](#funciones-y-procedimientos)
   - [Triggers (Disparadores)](#triggers-disparadores)
   - [Vistas de Reporte](#vistas-de-reporte)
5. [Consultas SQL Requeridas](#-consultas-sql-requeridas)
6. [Instrucciones de Ejecución](#-instrucciones-de-ejecución)

---

## 📄 Descripción del Proyecto

El proyecto resuelve la problemática del manejo manual de pedidos en la **Pizzería Don Piccolo**, el cual generaba retrasos en la atención y errores en los registros. Con este sistema se logra:

- **Gestión integral de Clientes y Repartidores**: Control de disponibilidad, zonas y clientes frecuentes.
- **Control de Inventario y Menú**: Seguimiento en tiempo real del stock de ingredientes y deducción automática en cada venta.
- **Cálculo Consolidado**: Cálculo automático de totales con IVA (19%) y costos de envío.
- **Auditoría e Historial**: Registro automático de cambios de precio en el menú y métricas de ganancia neta diaria.

---

## 📁 Estructura del Proyecto

El código del proyecto se organiza modularmente de la siguiente manera:

```
pizzeria-don-piccolo/
├── database.sql    # Creación del esquema, tablas, FKs y datos iniciales (INSERTS)
├── funciones.sql   # Funciones de cálculo de totales y ganancias netas, y Stored Procedures
├── triggers.sql    # Triggers de stock, historial de precios y disponibilidad
├── vistas.sql      # Vistas para reportes de clientes, desempeño e inventario
├── consultas.sql   # Consultas SQL requeridas (JOIN, HAVING, LIKE, Subconsultas, etc.)
└── README.md       # Documentación del proyecto
```

---

## 🗄️ Modelo de Datos y Relaciones

El sistema consta de 9 tablas principales interconectadas mediante llaves foráneas (`FOREIGN KEY`):

1. **`cliente`**: Almacena datos personales y de contacto (`id`, `nombre`, `telefono`, `direccion`, `email`).
2. **`pizza`**: Catálogo de productos (`id`, `nombre`, `tamano`, `precio_base`, `tipo`).
3. **`ingrediente`**: Control de stock e insumos (`id`, `nombre`, `stock`, `stock_min`, `costo`).
4. **`pizza_ingrediente`**: Tabla intermedia N:M entre `pizza` e `ingrediente` que define la receta (`cantidad`).
5. **`repartidor`**: Personal de entrega (`id`, `nombre`, `zona`, `disponibilidad`).
6. **`pedido`**: Cabecera de la transacción (`id`, `id_cliente`, `fecha`, `metodo_pago`, `estado`, `total`).
7. **`pedido_pizza`**: Detalle N:M de pizzas solicitadas en un pedido (`id_pedido`, `id_pizza`, `cantidad`, `precio`).
8. **`domicilio`**: Registro de logística de entrega (`id`, `id_pedido`, `id_repartidor`, `hora_salida`, `hora_entrega`, `distancia_km`, `costo_envio`).
9. **`historial_precios`**: Auditoría de variaciones en precios de pizzas (`id`, `id_pizza`, `precio_anterior`, `precio_nuevo`, `fecha`).

---

## ⚙️ Lógica de Negocio Automática

### Funciones y Procedimientos
- **`fn_total_pedido(p_id_pedido)`**: Calcula el total consolidado sumando las pizzas pedidas + costo de envío + 19% de IVA.
- **`fn_ganancia_diaria(p_fecha)`**: Devuelve la ganancia neta diaria restando los costos de ingredientes al total de ventas entregadas.
- **`sp_entregar_pedido(p_id_domicilio)`**: Actualiza la hora de entrega a `NOW()` y cambia el estado del pedido a `'entregado'`.

### Triggers (Disparadores)
- **`trg_descontar_stock`**: Descuenta automáticamente el stock de ingredientes requeridos tras registrar pizzas en `pedido_pizza`.
- **`trg_historial_precio`**: Registra un historial en `historial_precios` si el `precio_base` de una pizza es modificado (`UPDATE`).
- **`trg_repartidor_disponible`**: Cambia el estado del repartidor a disponible (`disponibilidad = 1`) cuando se registra la `hora_entrega` en la tabla `domicilio`.

### Vistas de Reporte
- **`vw_resumen_cliente`**: Total de pedidos y monto acumulado gastado por cliente.
- **`vw_desempeno_repartidor`**: Número de entregas concretadas y tiempo promedio de entrega por repartidor.
- **`vw_stock_bajo`**: Muestra únicamente los ingredientes cuyo stock actual es menor al mínimo requerido (`stock < stock_min`).

---

## 🔍 Consultas SQL Requeridas

El archivo `consultas.sql` incluye las consultas indispensables para la operación del negocio:

1. **Clientes con pedidos entre fechas (`BETWEEN`)**: Filtra clientes activos en un rango de fechas.
2. **Pizzas más vendidas (`GROUP BY` + `ORDER BY`)**: Ranking de popularidad de pizzas por cantidad consumida.
3. **Pedidos por repartidor (`JOIN`)**: Cruce de información logística entre repartidores, domicilios y pedidos.
4. **Promedio de entrega por zona (`AVG` + `GROUP BY`)**: Métricas de rendimiento logístico por área geográfica.
5. **Clientes de alto valor (`HAVING`)**: Filtra clientes con un consumo acumulado superior a $30,000.
6. **Búsqueda parcial (`LIKE`)**: Búsqueda flexible de productos en el catálogo (ej. `%hawaiana%`).
7. **Clientes frecuentes (Subconsulta)**: Identifica clientes con más de 5 pedidos realizados dentro del mes actual.

---

## 🚀 Instrucciones de Ejecución

1. **Prerrequisitos**: Tener instalado MySQL Server (v8.0+ recomendable) o MariaDB y un cliente como MySQL Workbench, DBeaver o la consola de comandos CLI.
2. **Crear la Base de Datos**:
   ```sql
   CREATE DATABASE pizzeria_don_piccolo;
   USE pizzeria_don_piccolo;
   ```
3. **Ejecutar en orden los scripts**:
   - `database.sql` (Crea las tablas, establece las Foreign Keys e inserta los datos de prueba).
   - `funciones.sql` (Crea las funciones y Stored Procedures).
   - `triggers.sql` (Crea los triggers de stock, auditoría y repartidores).
   - `vistas.sql` (Crea las vistas del sistema).
   - `consultas.sql` (Contiene los queries analíticos).
