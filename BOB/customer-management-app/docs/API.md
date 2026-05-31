# Documentación de API REST
## Sistema de Gestión de Clientes

Base URL: `http://localhost:8080/customer-management-api/api`

---

## Tabla de Contenidos

1. [Autenticación](#autenticación)
2. [Endpoints de Países](#endpoints-de-países)
3. [Endpoints de Ciudades](#endpoints-de-ciudades)
4. [Endpoints de Clientes](#endpoints-de-clientes)
5. [Códigos de Estado HTTP](#códigos-de-estado-http)
6. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Autenticación

Actualmente la API no requiere autenticación. Para implementar autenticación en el futuro, se recomienda usar JWT (JSON Web Tokens).

---

## Endpoints de Países

### Listar Todos los Países

**GET** `/paises`

Obtiene la lista completa de países ordenados por nombre.

**Respuesta exitosa (200):**
```json
[
  {
    "idPais": 1,
    "nombre": "Argentina",
    "fechaCreacion": "2026-05-30T00:00:00",
    "fechaModificacion": "2026-05-30T00:00:00"
  },
  {
    "idPais": 2,
    "nombre": "Chile",
    "fechaCreacion": "2026-05-30T00:00:00",
    "fechaModificacion": "2026-05-30T00:00:00"
  }
]
```

### Obtener País por ID

**GET** `/paises/{id}`

**Parámetros:**
- `id` (path, requerido): ID del país

**Respuesta exitosa (200):**
```json
{
  "idPais": 1,
  "nombre": "Argentina",
  "fechaCreacion": "2026-05-30T00:00:00",
  "fechaModificacion": "2026-05-30T00:00:00"
}
```

**Respuesta de error (404):**
```json
{
  "message": "País con ID 999 no encontrado"
}
```

### Crear Nuevo País

**POST** `/paises`

**Body (JSON):**
```json
{
  "nombre": "Colombia"
}
```

**Respuesta exitosa (201):**
```json
{
  "idPais": 6,
  "nombre": "Colombia",
  "fechaCreacion": "2026-05-30T10:30:00",
  "fechaModificacion": "2026-05-30T10:30:00"
}
```

**Respuesta de error (400):**
```json
{
  "message": "Ya existe un país con el nombre: Colombia"
}
```

### Actualizar País

**PUT** `/paises/{id}`

**Parámetros:**
- `id` (path, requerido): ID del país

**Body (JSON):**
```json
{
  "nombre": "República Argentina"
}
```

**Respuesta exitosa (200):**
```json
{
  "idPais": 1,
  "nombre": "República Argentina",
  "fechaCreacion": "2026-05-30T00:00:00",
  "fechaModificacion": "2026-05-30T10:35:00"
}
```

### Eliminar País

**DELETE** `/paises/{id}`

**Parámetros:**
- `id` (path, requerido): ID del país

**Respuesta exitosa (204):** Sin contenido

**Respuesta de error (409):**
```json
{
  "message": "No se puede eliminar el país porque tiene ciudades asociadas"
}
```

### Contar Países

**GET** `/paises/count`

**Respuesta exitosa (200):**
```json
{
  "count": 5
}
```

---

## Endpoints de Ciudades

### Listar Todas las Ciudades

**GET** `/ciudades`

Obtiene la lista completa de ciudades ordenadas por país y nombre.

**Respuesta exitosa (200):**
```json
[
  {
    "idCiudad": 1,
    "ciudad": "Buenos Aires",
    "pais": {
      "idPais": 1,
      "nombre": "Argentina"
    },
    "fechaCreacion": "2026-05-30T00:00:00",
    "fechaModificacion": "2026-05-30T00:00:00"
  }
]
```

### Obtener Ciudad por ID

**GET** `/ciudades/{id}`

**Parámetros:**
- `id` (path, requerido): ID de la ciudad

**Respuesta exitosa (200):**
```json
{
  "idCiudad": 1,
  "ciudad": "Buenos Aires",
  "pais": {
    "idPais": 1,
    "nombre": "Argentina"
  },
  "fechaCreacion": "2026-05-30T00:00:00",
  "fechaModificacion": "2026-05-30T00:00:00"
}
```

### Listar Ciudades por País

**GET** `/ciudades/pais/{idPais}`

**Parámetros:**
- `idPais` (path, requerido): ID del país

**Respuesta exitosa (200):**
```json
[
  {
    "idCiudad": 1,
    "ciudad": "Buenos Aires",
    "pais": {
      "idPais": 1,
      "nombre": "Argentina"
    }
  },
  {
    "idCiudad": 2,
    "ciudad": "Córdoba",
    "pais": {
      "idPais": 1,
      "nombre": "Argentina"
    }
  }
]
```

### Buscar Ciudades

**GET** `/ciudades/buscar?q={searchTerm}`

**Parámetros:**
- `q` (query, requerido): Término de búsqueda

**Ejemplo:** `/ciudades/buscar?q=buenos`

**Respuesta exitosa (200):**
```json
[
  {
    "idCiudad": 1,
    "ciudad": "Buenos Aires",
    "pais": {
      "idPais": 1,
      "nombre": "Argentina"
    }
  }
]
```

### Crear Nueva Ciudad

**POST** `/ciudades`

**Body (JSON):**
```json
{
  "ciudad": "Mendoza",
  "pais": {
    "idPais": 1
  }
}
```

**Respuesta exitosa (201):**
```json
{
  "idCiudad": 51,
  "ciudad": "Mendoza",
  "pais": {
    "idPais": 1,
    "nombre": "Argentina"
  },
  "fechaCreacion": "2026-05-30T10:40:00",
  "fechaModificacion": "2026-05-30T10:40:00"
}
```

### Actualizar Ciudad

**PUT** `/ciudades/{id}`

**Parámetros:**
- `id` (path, requerido): ID de la ciudad

**Body (JSON):**
```json
{
  "ciudad": "Ciudad de Buenos Aires",
  "pais": {
    "idPais": 1
  }
}
```

**Respuesta exitosa (200):**
```json
{
  "idCiudad": 1,
  "ciudad": "Ciudad de Buenos Aires",
  "pais": {
    "idPais": 1,
    "nombre": "Argentina"
  },
  "fechaCreacion": "2026-05-30T00:00:00",
  "fechaModificacion": "2026-05-30T10:45:00"
}
```

### Eliminar Ciudad

**DELETE** `/ciudades/{id}`

**Parámetros:**
- `id` (path, requerido): ID de la ciudad

**Respuesta exitosa (204):** Sin contenido

**Respuesta de error (409):**
```json
{
  "message": "No se puede eliminar la ciudad porque tiene 5 cliente(s) asociado(s)"
}
```

### Contar Ciudades

**GET** `/ciudades/count`

**Respuesta exitosa (200):**
```json
{
  "count": 50
}
```

### Contar Ciudades por País

**GET** `/ciudades/count/pais/{idPais}`

**Parámetros:**
- `idPais` (path, requerido): ID del país

**Respuesta exitosa (200):**
```json
{
  "count": 10
}
```

---

## Endpoints de Clientes

### Listar Todos los Clientes

**GET** `/clientes`

Obtiene la lista completa de clientes ordenados por nombre.

**Respuesta exitosa (200):**
```json
[
  {
    "idCliente": 1,
    "nombre": "Juan Pérez",
    "direccion": "Av. Corrientes 1234",
    "ciudad": {
      "idCiudad": 1,
      "ciudad": "Buenos Aires"
    },
    "pais": {
      "idPais": 1,
      "nombre": "Argentina"
    },
    "telefono": "+54 11 1234-5678",
    "correoElectronico": "juan.perez@email.com",
    "fechaCreacion": "2026-05-30T00:00:00",
    "fechaModificacion": "2026-05-30T00:00:00"
  }
]
```

### Obtener Cliente por ID

**GET** `/clientes/{id}`

**Parámetros:**
- `id` (path, requerido): ID del cliente

**Respuesta exitosa (200):**
```json
{
  "idCliente": 1,
  "nombre": "Juan Pérez",
  "direccion": "Av. Corrientes 1234",
  "ciudad": {
    "idCiudad": 1,
    "ciudad": "Buenos Aires"
  },
  "pais": {
    "idPais": 1,
    "nombre": "Argentina"
  },
  "telefono": "+54 11 1234-5678",
  "correoElectronico": "juan.perez@email.com",
  "fechaCreacion": "2026-05-30T00:00:00",
  "fechaModificacion": "2026-05-30T00:00:00"
}
```

### Buscar Clientes

**GET** `/clientes/buscar?q={searchTerm}`

Busca clientes por nombre, email, teléfono o dirección.

**Parámetros:**
- `q` (query, opcional): Término de búsqueda

**Ejemplo:** `/clientes/buscar?q=juan`

**Respuesta exitosa (200):**
```json
[
  {
    "idCliente": 1,
    "nombre": "Juan Pérez",
    "correoElectronico": "juan.perez@email.com"
  }
]
```

### Listar Clientes por País

**GET** `/clientes/pais/{idPais}`

**Parámetros:**
- `idPais` (path, requerido): ID del país

**Respuesta exitosa (200):** Array de clientes

### Listar Clientes por Ciudad

**GET** `/clientes/ciudad/{idCiudad}`

**Parámetros:**
- `idCiudad` (path, requerido): ID de la ciudad

**Respuesta exitosa (200):** Array de clientes

### Buscar Cliente por Email

**GET** `/clientes/email/{email}`

**Parámetros:**
- `email` (path, requerido): Correo electrónico del cliente

**Respuesta exitosa (200):**
```json
{
  "idCliente": 1,
  "nombre": "Juan Pérez",
  "correoElectronico": "juan.perez@email.com"
}
```

### Crear Nuevo Cliente

**POST** `/clientes`

**Body (JSON):**
```json
{
  "nombre": "María García",
  "direccion": "Calle Falsa 123",
  "ciudad": {
    "idCiudad": 2
  },
  "pais": {
    "idPais": 1
  },
  "telefono": "+54 11 9876-5432",
  "correoElectronico": "maria.garcia@email.com"
}
```

**Respuesta exitosa (201):**
```json
{
  "idCliente": 2,
  "nombre": "María García",
  "direccion": "Calle Falsa 123",
  "ciudad": {
    "idCiudad": 2,
    "ciudad": "Córdoba"
  },
  "pais": {
    "idPais": 1,
    "nombre": "Argentina"
  },
  "telefono": "+54 11 9876-5432",
  "correoElectronico": "maria.garcia@email.com",
  "fechaCreacion": "2026-05-30T11:00:00",
  "fechaModificacion": "2026-05-30T11:00:00"
}
```

**Respuesta de error (400):**
```json
{
  "message": "Ya existe un cliente con el correo electrónico: maria.garcia@email.com"
}
```

### Actualizar Cliente

**PUT** `/clientes/{id}`

**Parámetros:**
- `id` (path, requerido): ID del cliente

**Body (JSON):**
```json
{
  "nombre": "Juan Carlos Pérez",
  "direccion": "Av. Corrientes 5678",
  "ciudad": {
    "idCiudad": 1
  },
  "pais": {
    "idPais": 1
  },
  "telefono": "+54 11 1234-5678",
  "correoElectronico": "juancarlos.perez@email.com"
}
```

**Respuesta exitosa (200):** Cliente actualizado

### Eliminar Cliente

**DELETE** `/clientes/{id}`

**Parámetros:**
- `id` (path, requerido): ID del cliente

**Respuesta exitosa (204):** Sin contenido

### Contar Clientes

**GET** `/clientes/count`

**Respuesta exitosa (200):**
```json
{
  "count": 150
}
```

### Contar Clientes por País

**GET** `/clientes/count/pais/{idPais}`

**Respuesta exitosa (200):**
```json
{
  "count": 45
}
```

### Contar Clientes por Ciudad

**GET** `/clientes/count/ciudad/{idCiudad}`

**Respuesta exitosa (200):**
```json
{
  "count": 12
}
```

---

## Códigos de Estado HTTP

| Código | Descripción |
|--------|-------------|
| 200 | OK - Solicitud exitosa |
| 201 | Created - Recurso creado exitosamente |
| 204 | No Content - Eliminación exitosa |
| 400 | Bad Request - Datos inválidos o duplicados |
| 404 | Not Found - Recurso no encontrado |
| 409 | Conflict - No se puede eliminar por dependencias |
| 500 | Internal Server Error - Error del servidor |

---

## Ejemplos de Uso

### Usando cURL

#### Crear un país:
```bash
curl -X POST http://localhost:8080/customer-management-api/api/paises \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Colombia"}'
```

#### Obtener todos los clientes:
```bash
curl http://localhost:8080/customer-management-api/api/clientes
```

#### Buscar clientes:
```bash
curl "http://localhost:8080/customer-management-api/api/clientes/buscar?q=juan"
```

#### Actualizar un cliente:
```bash
curl -X PUT http://localhost:8080/customer-management-api/api/clientes/1 \
  -H "Content-Type: application/json" \
  -d '{
    "nombre":"Juan Pérez Actualizado",
    "direccion":"Nueva Dirección 123",
    "ciudad":{"idCiudad":1},
    "pais":{"idPais":1},
    "telefono":"+54 11 1111-2222",
    "correoElectronico":"juan.nuevo@email.com"
  }'
```

#### Eliminar una ciudad:
```bash
curl -X DELETE http://localhost:8080/customer-management-api/api/ciudades/51
```

### Usando JavaScript (Axios)

```javascript
import axios from 'axios';

const API_URL = 'http://localhost:8080/customer-management-api/api';

// Obtener todos los países
const getPaises = async () => {
  const response = await axios.get(`${API_URL}/paises`);
  return response.data;
};

// Crear un cliente
const createCliente = async (cliente) => {
  const response = await axios.post(`${API_URL}/clientes`, cliente);
  return response.data;
};

// Buscar clientes
const searchClientes = async (searchTerm) => {
  const response = await axios.get(`${API_URL}/clientes/buscar`, {
    params: { q: searchTerm }
  });
  return response.data;
};
```

---

## Swagger UI

La documentación interactiva de la API está disponible en:

**URL:** `http://localhost:8080/customer-management-api/swagger-ui/`

Desde Swagger UI puedes:
- Ver todos los endpoints disponibles
- Probar las peticiones directamente
- Ver los esquemas de datos
- Descargar la especificación OpenAPI

---

## Notas Adicionales

1. **Validaciones:**
   - Los campos marcados como requeridos deben estar presentes
   - Los emails deben tener formato válido
   - Los nombres no pueden exceder 50 caracteres

2. **Relaciones:**
   - Al crear una ciudad, el país debe existir
   - Al crear un cliente, la ciudad y país deben existir
   - No se pueden eliminar países o ciudades con dependencias

3. **Búsquedas:**
   - Las búsquedas son case-insensitive
   - Se busca en múltiples campos simultáneamente
   - Los resultados están ordenados por relevancia

4. **Performance:**
   - Las consultas están optimizadas con índices
   - Se usa eager loading para relaciones frecuentes
   - El pool de conexiones está configurado para 20 conexiones máximas
