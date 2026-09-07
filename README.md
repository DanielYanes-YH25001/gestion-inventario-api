# gestion-inventario-api

API para la gestión de un inventario en Java como proyecto final de la asignatura Programación Orientada a Objetos

---

## Modelo de Base de Datos (Colecciones NoSQL / Firestore)

Estructura de documentos y colecciones principales:

| Colección            | Descripción                                                              |
| :------------------- | :----------------------------------------------------------------------- |
| **`empresas`**       | Entidad principal multitenant/organización.                              |
| **`agencias`**       | Sucursales o puntos de venta vinculados a una empresa.                   |
| **`productos`**      | Catálogo general de productos (atributos, costos, flag de vencimiento).  |
| **`stock_agencias`** | Existencias reales agrupadas por agencia, producto y lote/vencimiento.   |
| **`kardex`**         | Historial inmutable de entradas, salidas y transferencias de inventario. |
| **`proveedores`**    | Catálogo de proveedores asociados a los productos.                       |
| **`usuarios`**       | Control de acceso y trazabilidad de operaciones.                         |

---

## Especificación de Endpoints REST

### Productos (`/api/v1/productos`)

- **`GET /`**: Lista el catálogo general con filtros por empresa y proveedor.
- **`POST /`**: Registra un nuevo producto (indicando si requiere control de fecha de vencimiento).
- **`PUT /:id`**: Actualiza datos del producto.
- **`DELETE /:id`**: Desactiva o elimina un producto del catálogo.

### Stock y Agencias (`/api/v1/stock`)

- **`GET /agencia/:agenciaId`**: Consulta la disponibilidad de stock en una agencia.
- **`GET /vencimientos`**: Reporte de productos próximos a vencer o vencidos en agencias.

### Kardex y Movimientos (`/api/v1/kardex`)

- **`POST /entrada`**: Registra entrada de stock (incrementa `stock_agencias` y genera registro en `kardex`).
- **`POST /salida`**: Registra salida por venta/merma (decrementa `stock_agencias` y genera registro en `kardex`).

---

## Arquitectura y Organización del Proyecto

```text
api-gestion-inventario/
├── src/
│   ├── config/             # Configuración de base de datos y variables de entorno
│   ├── controllers/        # Controladores (HTTP handlers)
│   ├── models/             # Esquemas y modelos de datos
│   │   ├── Agencia.js
│   │   ├── Empresa.js
│   │   ├── Kardex.js
│   │   ├── Producto.js
│   │   ├── Proveedor.js
│   │   ├── StockAgencia.js
│   │   └── Usuario.js
│   ├── routes/             # Definición de rutas (GET, POST, PUT, DELETE)
│   ├── services/           # Lógica de negocio (actualización de stock, validación de vencimientos)
│   └── utils/              # Funciones auxiliares y validadores
├── .env.example
├── .gitignore
├── package.json
└── README.md
```

---

## Models propuestos

---

## src/models/Empresa.js

```text
class Empresa {
    constructor(id, nombre, nit, nrf, direccion, telefono, activa, creadaEn) {
        this.id = id;                     // String
        this.nombre = nombre;             // String
        this.nit = nit;                   // String
        this.nrc = nrc;                   // String
        this.direccion = direccion;       // String
        this.telefono = telefono;         // String
        this.activa = activa ?? true;     // Boolean
        this.creadaEn = creadaEn || new Date();
    }
}

module.exports = Empresa;
```

---

## models/StockAgencia.js

```text
class Agencia {
    constructor(id, empresaId, nombre, direccion, telefono, activa, creadaEn) {
        this.id = id;                     // String
        this.empresaId = empresaId;       // String (Relación con Empresa)
        this.nombre = nombre;             // String
        this.direccion = direccion;       // String
        this.telefono = telefono;         // String
        this.activa = activa ?? true;     // Boolean
        this.creadaEn = creadaEn || new Date();
    }
}

module.exports = Agencia;
```

---

## models/Producto.js

```text
class Producto {
    constructor(id, empresaId, proveedorId, codigo, nombre, descripcion, precio, requiereVencimiento, fechaVencimiento, activo, creadoEn) {
        this.id = id;                           // String
        this.empresaId = empresaId;             // String
        this.proveedorId = proveedorId;         // String
        this.codigo = codigo;                   // String (SKU o código de barras)
        this.nombre = nombre;                   // String
        this.descripcion = descripcion;         // String
        this.precio = precio;                   // Number
        this.requiereVencimiento = requiereVencimiento ?? false; // Boolean (Para perecederos)
        this.activo = activo ?? true;           // Boolean
        this.creadoEn = creadoEn || new Date();
    }
}

module.exports = Producto;
```

---

## src/models/StockAgencia.js

```text
class StockAgencia {
    constructor(id, empresaId, prodId, agencia, cantidad, lote, fechaVencimiento) {
        this.id = id;                           // String
        this.empresaId = empresaId;             // String
        this.prodId = prodId;                   // String
        this.agencia = agencia;                 // String
        this.cantidad = cantidad;               // Number
        this.lote = lote || null;               // String (Opcional)
        this.fechaVencimiento = fechaVencimiento || null; // Date (Opcional para productos que vencen)
    }
}

module.exports = StockAgencia;
```

---

## models/Kardex.js

```text
class Kardex {
    constructor(id, empresaId, prodId, agenciaId, tipoMovimiento, cantidad, stockAnterior, stockNuevo, lote, fechaVencimiento, fecha, usuarioId) {
        this.id = id;                           // String
        this.empresaId = empresaId;             // String
        this.prodId = prodId;                   // String
        this.agenciaId = agenciaId;             // String
        this.tipoMovimiento = tipoMovimiento;   // String ("ENTRADA", "SALIDA", "AJUSTE", "TRASLADO")
        this.cantidad = cantidad;               // Number
        this.stockAnterior = stockAnterior;     // Number
        this.stockNuevo = stockNuevo;           // Number
        this.lote = lote || null;               // String
        this.fechaVencimiento = fechaVencimiento || null; // Date
        this.fecha = fecha || new Date();
        this.usuarioId = usuarioId;             // String
    }
}

module.exports = Kardex;
```

---

## src/models/Proveedor.js

```text
class Proveedor {
    constructor(id, empresaId, nombre, contacto, telefono, email, direccion, activo, creadoEn) {
        this.id = id;                     // String
        this.empresaId = empresaId;       // String
        this.nombre = nombre;             // String
        this.contacto = contacto;         // String
        this.telefono = telefono;         // String
        this.email = email;               // String
        this.direccion = direccion;       // String
        this.activo = activo ?? true;     // Boolean
        this.creadoEn = creadoEn || new Date();
    }
}

module.exports = Proveedor;

```

---

## src/models/Usuario.js

```text

class Usuario {
    constructor(id, empresaId, agenciaId, nombre, email, rol, activo, creadoEn) {
        this.id = id;                     // String
        this.empresaId = empresaId;       // String
        this.agenciaId = agenciaId;       // String
        this.nombre = nombre;             // String
        this.email = email;               // String
        this.rol = rol;                   // String ("ADMIN", "Bodeguero", "SUPERVISOR")
        this.activo = activo ?? true;     // Boolean
        this.creadoEn = creadoEn || new Date();
    }
}

module.exports = Usuario;

```

---
