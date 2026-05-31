# Guía de Instalación y Configuración
## Sistema de Gestión de Clientes - Aplicación de 3 Capas

Esta guía proporciona instrucciones paso a paso para instalar y configurar el sistema completo.

---

## Tabla de Contenidos

1. [Requisitos del Sistema](#requisitos-del-sistema)
2. [Instalación del Backend (MySQL)](#instalación-del-backend-mysql)
3. [Instalación del Middleware (Quarkus/Tomcat)](#instalación-del-middleware-quarkustomcat)
4. [Instalación del Frontend (React/Apache)](#instalación-del-frontend-reactapache)
5. [Verificación del Sistema](#verificación-del-sistema)
6. [Troubleshooting](#troubleshooting)

---

## Requisitos del Sistema

### Backend Server (Base de Datos)
- **Sistema Operativo**: Red Hat Enterprise Linux 8/9
- **RAM**: Mínimo 2GB
- **Disco**: 20GB disponibles
- **Software**: MySQL 8.4 LTS

### Middleware Server (API)
- **Sistema Operativo**: Linux/Windows Server
- **RAM**: Mínimo 4GB
- **Disco**: 10GB disponibles
- **Software**: 
  - Java 21 LTS (OpenJDK)
  - Apache Maven 3.8+
  - Apache Tomcat 10.x

### Frontend Server (Interfaz Web)
- **Sistema Operativo**: Linux/Windows Server
- **RAM**: Mínimo 2GB
- **Disco**: 5GB disponibles
- **Software**:
  - Node.js 20 LTS
  - Apache HTTP Server 2.4

---

## Instalación del Backend (MySQL)

### Paso 1: Preparar el Servidor

```bash
# Actualizar el sistema
sudo dnf update -y

# Instalar herramientas necesarias
sudo dnf install -y wget curl vim
```

### Paso 2: Ejecutar Script de Instalación

```bash
# Navegar al directorio de scripts
cd customer-management-app/backend/scripts

# Dar permisos de ejecución
chmod +x install-mysql-redhat.sh

# Ejecutar el script
sudo ./install-mysql-redhat.sh
```

El script realizará:
- Instalación de MySQL 8.4 LTS
- Configuración de seguridad
- Creación de usuario `customerapp`
- Configuración del firewall

### Paso 3: Crear Base de Datos y Tablas

```bash
# Conectarse a MySQL
mysql -u root -p

# Ejecutar scripts SQL en orden
mysql -u root -p < ../sql/01-create-tables.sql
mysql -u root -p < ../sql/02-insert-paises.sql
mysql -u root -p < ../sql/03-insert-ciudades.sql
```

### Paso 4: Verificar Instalación

```bash
# Verificar que MySQL está ejecutándose
sudo systemctl status mysqld

# Verificar tablas creadas
mysql -u customerapp -p -e "USE customer_management; SHOW TABLES;"

# Verificar datos iniciales
mysql -u customerapp -p -e "USE customer_management; SELECT COUNT(*) FROM PAIS;"
mysql -u customerapp -p -e "USE customer_management; SELECT COUNT(*) FROM CIUDADES;"
```

**Resultado esperado:**
- 5 países
- 50 ciudades (10 por país)

---

## Instalación del Middleware (Quarkus/Tomcat)

### Paso 1: Instalar Java 21 y Tomcat

```bash
# Navegar al directorio de scripts
cd customer-management-app/middleware/scripts

# Dar permisos de ejecución
chmod +x install-tomcat.sh

# Ejecutar el script
sudo ./install-tomcat.sh
```

El script instalará:
- Java 21 OpenJDK
- Apache Maven
- Apache Tomcat 10.1.28
- Configuración como servicio systemd

### Paso 2: Configurar Conexión a Base de Datos

Editar el archivo de configuración:

```bash
cd customer-management-app/middleware
vim src/main/resources/application.properties
```

Actualizar las siguientes propiedades:

```properties
# URL de la base de datos (cambiar localhost si es necesario)
quarkus.datasource.jdbc.url=jdbc:mysql://IP_DEL_SERVIDOR_BD:3306/customer_management

# Credenciales (usar las configuradas en el backend)
quarkus.datasource.username=customerapp
quarkus.datasource.password=TU_CONTRASEÑA_AQUI
```

### Paso 3: Compilar la Aplicación

```bash
# Compilar el proyecto
mvn clean package

# Verificar que se generó el WAR
ls -lh target/customer-management-api.war
```

### Paso 4: Desplegar en Tomcat

```bash
# Copiar el WAR a Tomcat
sudo cp target/customer-management-api.war /opt/tomcat/webapps/

# Reiniciar Tomcat
sudo systemctl restart tomcat

# Verificar logs
tail -f /opt/tomcat/logs/catalina.out
```

### Paso 5: Verificar Despliegue

```bash
# Verificar que la aplicación está desplegada
curl http://localhost:8080/customer-management-api/api/paises

# Verificar Swagger UI
# Abrir en navegador: http://IP_SERVIDOR:8080/customer-management-api/swagger-ui/
```

---

## Instalación del Frontend (React/Apache)

### Paso 1: Instalar Node.js

```bash
# Instalar Node.js 20 LTS
curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
sudo dnf install -y nodejs

# Verificar instalación
node --version
npm --version
```

### Paso 2: Instalar Dependencias del Proyecto

```bash
# Navegar al directorio del frontend
cd customer-management-app/frontend

# Instalar dependencias
npm install
```

### Paso 3: Configurar URL de la API

Crear archivo `.env` en el directorio frontend:

```bash
cat > .env << EOF
REACT_APP_API_URL=http://IP_DEL_MIDDLEWARE:8080/customer-management-api/api
EOF
```

### Paso 4: Compilar para Producción

```bash
# Compilar la aplicación
npm run build

# Verificar que se generó el directorio build
ls -lh build/
```

### Paso 5: Instalar y Configurar Apache

```bash
# Instalar Apache HTTP Server
sudo dnf install -y httpd

# Copiar archivos compilados
sudo cp -r build/* /var/www/html/

# Configurar Apache para React Router
sudo tee /etc/httpd/conf.d/customer-app.conf > /dev/null << 'EOF'
<VirtualHost *:80>
    ServerName customer-app.local
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
        
        # Rewrite para React Router
        RewriteEngine On
        RewriteBase /
        RewriteRule ^index\.html$ - [L]
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule . /index.html [L]
    </Directory>

    ErrorLog /var/log/httpd/customer-app-error.log
    CustomLog /var/log/httpd/customer-app-access.log combined
</VirtualHost>
EOF

# Habilitar mod_rewrite
sudo sed -i 's/^#LoadModule rewrite_module/LoadModule rewrite_module/' /etc/httpd/conf.modules.d/00-base.conf

# Configurar firewall
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload

# Iniciar Apache
sudo systemctl start httpd
sudo systemctl enable httpd
```

### Paso 6: Verificar Instalación

```bash
# Verificar que Apache está ejecutándose
sudo systemctl status httpd

# Probar acceso
curl http://localhost/

# Abrir en navegador
# http://IP_DEL_SERVIDOR/
```

---

## Verificación del Sistema

### 1. Verificar Backend (MySQL)

```bash
# Conectarse y verificar datos
mysql -u customerapp -p customer_management -e "
SELECT 
    (SELECT COUNT(*) FROM PAIS) as Paises,
    (SELECT COUNT(*) FROM CIUDADES) as Ciudades,
    (SELECT COUNT(*) FROM CLIENTES) as Clientes;
"
```

### 2. Verificar Middleware (API)

```bash
# Probar endpoints
curl http://localhost:8080/customer-management-api/api/paises
curl http://localhost:8080/customer-management-api/api/ciudades
curl http://localhost:8080/customer-management-api/api/clientes

# Verificar health check
curl http://localhost:8080/customer-management-api/health
```

### 3. Verificar Frontend

Abrir en navegador:
- http://IP_DEL_SERVIDOR/
- Verificar que carga la página principal
- Verificar que el menú de navegación funciona

### 4. Prueba de Integración Completa

1. Acceder al frontend
2. Navegar a "Clientes"
3. Crear un nuevo cliente
4. Verificar que aparece en la lista
5. Editar el cliente
6. Eliminar el cliente

---

## Troubleshooting

### Problema: MySQL no inicia

```bash
# Verificar logs
sudo tail -f /var/log/mysqld.log

# Verificar permisos
sudo chown -R mysql:mysql /var/lib/mysql

# Reiniciar servicio
sudo systemctl restart mysqld
```

### Problema: Tomcat no despliega la aplicación

```bash
# Verificar logs
tail -f /opt/tomcat/logs/catalina.out

# Verificar que Java 21 está configurado
java -version

# Verificar permisos del WAR
ls -l /opt/tomcat/webapps/customer-management-api.war

# Limpiar y redesplegar
sudo rm -rf /opt/tomcat/webapps/customer-management-api*
sudo cp target/customer-management-api.war /opt/tomcat/webapps/
sudo systemctl restart tomcat
```

### Problema: Error de conexión entre Middleware y Backend

```bash
# Verificar conectividad
telnet IP_MYSQL 3306

# Verificar firewall en servidor MySQL
sudo firewall-cmd --list-all

# Verificar usuario y permisos en MySQL
mysql -u root -p -e "SELECT user, host FROM mysql.user WHERE user='customerapp';"
```

### Problema: Frontend no se conecta al Middleware

1. Verificar archivo `.env`:
```bash
cat .env
```

2. Verificar CORS en el middleware:
```bash
grep "quarkus.http.cors" src/main/resources/application.properties
```

3. Verificar en consola del navegador (F12) si hay errores de CORS

### Problema: Apache no sirve la aplicación React

```bash
# Verificar configuración
sudo httpd -t

# Verificar logs
sudo tail -f /var/log/httpd/error_log

# Verificar permisos
sudo ls -la /var/www/html/

# Reiniciar Apache
sudo systemctl restart httpd
```

---

## Comandos Útiles

### MySQL
```bash
sudo systemctl start mysqld
sudo systemctl stop mysqld
sudo systemctl restart mysqld
sudo systemctl status mysqld
```

### Tomcat
```bash
sudo systemctl start tomcat
sudo systemctl stop tomcat
sudo systemctl restart tomcat
sudo systemctl status tomcat
tail -f /opt/tomcat/logs/catalina.out
```

### Apache
```bash
sudo systemctl start httpd
sudo systemctl stop httpd
sudo systemctl restart httpd
sudo systemctl status httpd
tail -f /var/log/httpd/error_log
```

---

## Próximos Pasos

Una vez completada la instalación:

1. Cambiar contraseñas por defecto
2. Configurar backups automáticos de la base de datos
3. Configurar SSL/HTTPS para producción
4. Implementar monitoreo y logs centralizados
5. Configurar autenticación y autorización

---

## Soporte

Para más información, consultar:
- [Documentación de la API](API.md)
- [Plan del Proyecto](../PLAN.md)
- [README del Middleware](../middleware/README.md)
