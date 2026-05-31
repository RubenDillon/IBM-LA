# Sistema de Gestión de Clientes
## Aplicación de 3 Capas

Sistema completo para la gestión de clientes, ciudades y países, desarrollado con arquitectura de 3 capas.

---

## 📋 Descripción

Aplicación web empresarial que permite:
- ✅ Crear, modificar, eliminar y buscar clientes
- ✅ Gestionar ciudades por país
- ✅ Administrar países del sistema
- ✅ Búsqueda avanzada de clientes
- ✅ Relaciones entre entidades (País → Ciudades → Clientes)

---

## 🏗️ Arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│                    CAPA DE PRESENTACIÓN                      │
│  ┌────────────────────────────────────────────────────┐     │
│  │  React 18 + React Router + Axios                   │     │
│  │  Servido por Apache HTTP Server 2.4                │     │
│  └────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            ↓ HTTP/REST
┌─────────────────────────────────────────────────────────────┐
│                  CAPA DE LÓGICA DE NEGOCIO                   │
│  ┌────────────────────────────────────────────────────┐     │
│  │  Quarkus 3.x + Java 21 LTS                         │     │
│  │  API REST + Hibernate ORM + Panache                │     │
│  │  Desplegado en Apache Tomcat 10.x                  │     │
│  └────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            ↓ JDBC
┌─────────────────────────────────────────────────────────────┐
│                      CAPA DE DATOS                           │
│  ┌────────────────────────────────────────────────────┐     │
│  │  MySQL 8.4 LTS                                     │     │
│  │  Red Hat Enterprise Linux 8/9                      │     │
│  │  3 Tablas: PAIS, CIUDADES, CLIENTES               │     │
│  └────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Stack Tecnológico

### Backend (Base de Datos)
- **MySQL 8.4 LTS** - Base de datos relacional
- **Red Hat Enterprise Linux** - Sistema operativo

### Middleware (API)
- **Java 21 LTS** - Lenguaje de programación
- **Quarkus 3.15.1** - Framework de microservicios
- **Hibernate ORM con Panache** - ORM para persistencia
- **RESTEasy Reactive** - Framework REST
- **Apache Tomcat 10.1.28** - Servidor de aplicaciones
- **Maven 3.8+** - Gestión de dependencias

### Frontend (Interfaz Web)
- **React 18.3.1** - Biblioteca de UI
- **React Router 6.26.2** - Enrutamiento
- **Axios 1.7.7** - Cliente HTTP
- **Apache HTTP Server 2.4** - Servidor web
- **Node.js 20 LTS** - Entorno de desarrollo

---

## 📁 Estructura del Proyecto

```
customer-management-app/
├── backend/                    # Capa de datos
│   ├── sql/                   # Scripts SQL
│   │   ├── 01-create-tables.sql
│   │   ├── 02-insert-paises.sql
│   │   └── 03-insert-ciudades.sql
│   └── scripts/               # Scripts de instalación
│       └── install-mysql-redhat.sh
│
├── middleware/                 # Capa de lógica de negocio
│   ├── src/
│   │   └── main/
│   │       ├── java/com/customerapp/
│   │       │   ├── entity/    # Entidades JPA
│   │       │   ├── service/   # Lógica de negocio
│   │       │   └── resource/  # Endpoints REST
│   │       └── resources/
│   │           └── application.properties
│   ├── scripts/
│   │   └── install-tomcat.sh
│   ├── pom.xml
│   └── README.md
│
├── frontend/                   # Capa de presentación
│   ├── public/
│   │   └── index.html
│   ├── src/
│   │   ├── components/        # Componentes React
│   │   │   ├── clientes/
│   │   │   ├── ciudades/
│   │   │   └── paises/
│   │   ├── services/          # Servicios HTTP
│   │   │   ├── api.js
│   │   │   ├── clienteService.js
│   │   │   ├── ciudadService.js
│   │   │   └── paisService.js
│   │   ├── App.js
│   │   └── index.js
│   ├── scripts/
│   │   └── install-apache.sh
│   └── package.json
│
├── docs/                       # Documentación
│   ├── INSTALLATION.md        # Guía de instalación
│   └── API.md                 # Documentación de API
│
├── PLAN.md                    # Plan del proyecto
└── README.md                  # Este archivo
```

---

## 🗄️ Modelo de Datos

### Tabla: PAIS
```sql
IdPais (PK, AUTO_INCREMENT)
Nombre (VARCHAR 50, UNIQUE)
FechaCreacion (TIMESTAMP)
FechaModificacion (TIMESTAMP)
```

### Tabla: CIUDADES
```sql
IdCiudad (PK, AUTO_INCREMENT)
IdPais (FK → PAIS)
Ciudad (VARCHAR 50)
FechaCreacion (TIMESTAMP)
FechaModificacion (TIMESTAMP)
UNIQUE(Ciudad, IdPais)
```

### Tabla: CLIENTES
```sql
IdCliente (PK, AUTO_INCREMENT)
Nombre (VARCHAR 50)
Direccion (VARCHAR 50)
IdCiudad (FK → CIUDADES)
IdPais (FK → PAIS)
Telefono (VARCHAR 14)
CorreoElectronico (VARCHAR 50)
FechaCreacion (TIMESTAMP)
FechaModificacion (TIMESTAMP)
```

**Datos Iniciales:**
- 5 países: Argentina, Chile, Uruguay, Paraguay, Brasil
- 50 ciudades: 10 ciudades principales por país

---

## 🚀 Inicio Rápido

### Prerrequisitos

- Red Hat Enterprise Linux 8/9 (para backend)
- Java 21 LTS
- Maven 3.8+
- Node.js 20 LTS
- MySQL 8.4 LTS
- Apache Tomcat 10.x
- Apache HTTP Server 2.4

### Instalación

#### 1. Backend (MySQL)

```bash
cd backend/scripts
sudo ./install-mysql-redhat.sh

# Ejecutar scripts SQL
mysql -u root -p < ../sql/01-create-tables.sql
mysql -u root -p < ../sql/02-insert-paises.sql
mysql -u root -p < ../sql/03-insert-ciudades.sql
```

#### 2. Middleware (Quarkus/Tomcat)

```bash
cd middleware/scripts
sudo ./install-tomcat.sh

# Compilar y desplegar
cd ..
mvn clean package
sudo cp target/customer-management-api.war /opt/tomcat/webapps/
sudo systemctl restart tomcat
```

#### 3. Frontend (React/Apache)

```bash
cd frontend
npm install
npm run build

# Desplegar en Apache
sudo cp -r build/* /var/www/html/
sudo systemctl restart httpd
```

### Verificación

1. **Backend**: `mysql -u customerapp -p -e "USE customer_management; SHOW TABLES;"`
2. **Middleware**: http://localhost:8080/customer-management-api/swagger-ui/
3. **Frontend**: http://localhost/

---

## 📚 Documentación

- **[Guía de Instalación Completa](docs/INSTALLATION.md)** - Instrucciones detalladas paso a paso
- **[Documentación de API](docs/API.md)** - Referencia completa de endpoints REST
- **[Plan del Proyecto](PLAN.md)** - Arquitectura y especificaciones técnicas
- **[README del Middleware](middleware/README.md)** - Documentación específica de la API

---

## 🔌 API Endpoints

### Países
- `GET /api/paises` - Listar todos
- `POST /api/paises` - Crear nuevo
- `PUT /api/paises/{id}` - Actualizar
- `DELETE /api/paises/{id}` - Eliminar

### Ciudades
- `GET /api/ciudades` - Listar todas
- `GET /api/ciudades/pais/{idPais}` - Por país
- `GET /api/ciudades/buscar?q={term}` - Buscar
- `POST /api/ciudades` - Crear nueva
- `PUT /api/ciudades/{id}` - Actualizar
- `DELETE /api/ciudades/{id}` - Eliminar

### Clientes
- `GET /api/clientes` - Listar todos
- `GET /api/clientes/buscar?q={term}` - Buscar
- `GET /api/clientes/pais/{idPais}` - Por país
- `GET /api/clientes/ciudad/{idCiudad}` - Por ciudad
- `POST /api/clientes` - Crear nuevo
- `PUT /api/clientes/{id}` - Actualizar
- `DELETE /api/clientes/{id}` - Eliminar

**Swagger UI**: http://localhost:8080/customer-management-api/swagger-ui/

---

## 🧪 Testing

### Backend
```bash
# Verificar conexión
mysql -u customerapp -p customer_management -e "SELECT COUNT(*) FROM CLIENTES;"
```

### Middleware
```bash
# Probar endpoints
curl http://localhost:8080/customer-management-api/api/paises
curl http://localhost:8080/customer-management-api/health
```

### Frontend
```bash
# Modo desarrollo
cd frontend
npm start

# Tests
npm test
```

---

## 🔧 Configuración

### Variables de Entorno

**Middleware** (`middleware/src/main/resources/application.properties`):
```properties
quarkus.datasource.jdbc.url=jdbc:mysql://localhost:3306/customer_management
quarkus.datasource.username=customerapp
quarkus.datasource.password=changeme
```

**Frontend** (`frontend/.env`):
```env
REACT_APP_API_URL=http://localhost:8080/customer-management-api/api
```

---

## 📊 Características

### Funcionalidades Implementadas

✅ **CRUD Completo**
- Crear, leer, actualizar y eliminar para todas las entidades
- Validaciones de datos
- Manejo de errores

✅ **Búsqueda Avanzada**
- Búsqueda de clientes por múltiples campos
- Filtrado por país y ciudad
- Búsqueda case-insensitive

✅ **Relaciones**
- Integridad referencial
- Cascada de operaciones
- Validación de dependencias

✅ **API REST**
- Documentación OpenAPI/Swagger
- CORS configurado
- Health checks

✅ **Seguridad**
- Validación de entrada
- Sanitización de datos
- Manejo de excepciones

---

## 🛡️ Seguridad

- Validación de entrada en todos los endpoints
- Protección contra SQL injection (JPA/Hibernate)
- CORS configurado para orígenes específicos
- Contraseñas seguras para base de datos
- Usuarios con permisos limitados

---

## 📈 Performance

- Pool de conexiones configurado (5-20 conexiones)
- Índices en campos de búsqueda frecuente
- Eager loading para relaciones comunes
- Caché de segundo nivel (Hibernate)
- Compresión de respuestas HTTP

---

## 🐛 Troubleshooting

Ver la [Guía de Instalación](docs/INSTALLATION.md#troubleshooting) para soluciones a problemas comunes.

---

## 📝 Licencia

Este proyecto es de código abierto y está disponible bajo la licencia Apache 2.0.

---

## 👥 Autores

- Sistema desarrollado como proyecto de arquitectura de 3 capas
- Tecnologías: MySQL, Java/Quarkus, React

---

## 🔄 Versiones

- **v1.0.0** (2026-05-30) - Versión inicial
  - Backend con MySQL 8.4 LTS
  - Middleware con Quarkus 3.x y Java 21
  - Frontend con React 18
  - CRUD completo para todas las entidades
  - Búsqueda avanzada de clientes
  - Documentación completa

---

## 📞 Soporte

Para reportar problemas o solicitar características:
1. Revisar la documentación en `/docs`
2. Verificar los logs de cada capa
3. Consultar la sección de Troubleshooting

---

## 🎯 Próximas Mejoras

- [ ] Autenticación y autorización (JWT)
- [ ] Paginación en listados
- [ ] Exportación de datos (CSV, PDF)
- [ ] Dashboard con estadísticas
- [ ] Notificaciones en tiempo real
- [ ] Modo offline (PWA)
- [ ] Tests automatizados completos
- [ ] CI/CD pipeline
- [ ] Monitoreo y métricas
- [ ] Backup automático

---

**¡Gracias por usar el Sistema de Gestión de Clientes!** 🚀
