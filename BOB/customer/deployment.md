# Guía Completa de Despliegue
## Sistema de Gestión de Clientes - Arquitectura de 3 Capas

Esta guía proporciona instrucciones paso a paso para desplegar el sistema completo en Red Hat Enterprise Linux 9.

---

## 📋 Requisitos Previos

### Hardware Mínimo Recomendado

**Backend (10.241.0.15)**:
- CPU: 2 cores
- RAM: 4 GB
- Disco: 20 GB
- Red Hat Enterprise Linux 9

**Middleware (10.241.0.14)**:
- CPU: 2 cores
- RAM: 4 GB
- Disco: 20 GB
- Red Hat Enterprise Linux 9

**Frontend (10.241.0.13)**:
- CPU: 2 cores
- RAM: 2 GB
- Disco: 20 GB
- Red Hat Enterprise Linux 9

### Acceso Requerido
- Acceso root o sudo en las tres máquinas
- Conectividad de red entre las máquinas
- Acceso a repositorios de Red Hat

---

## 🔧 FASE 1: Preparación del Entorno

### En todas las máquinas (10.241.0.13, 10.241.0.14, 10.241.0.15)

```bash
# 1. Actualizar el sistema
sudo dnf update -y

# 2. Configurar hostname (opcional pero recomendado)
# En 10.241.0.15:
sudo hostnamectl set-hostname backend-db

# En 10.241.0.14:
sudo hostnamectl set-hostname middleware-api

# En 10.241.0.13:
sudo hostnamectl set-hostname frontend-web

# 3. Configurar /etc/hosts en todas las máquinas
sudo tee -a /etc/hosts << EOF
10.241.0.15 backend-db
10.241.0.14 middleware-api
10.241.0.13 frontend-web
EOF

# 4. Verificar conectividad
ping -c 3 10.241.0.15
ping -c 3 10.241.0.14
ping -c 3 10.241.0.13

# 5. Deshabilitar SELinux temporalmente (para pruebas)
# NOTA: En producción, configurar políticas SELinux apropiadas
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config
```

---

## 🗄️ FASE 2: Instalación y Configuración del Backend

### En la máquina 10.241.0.15

#### Paso 1: Instalar MySQL

```bash
# Instalar MySQL Server
sudo dnf install mysql-server -y

# Iniciar y habilitar el servicio
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Verificar estado
sudo systemctl status mysqld
```

#### Paso 2: Configurar Seguridad de MySQL

```bash
# Ejecutar script de seguridad
sudo mysql_secure_installation

# Responder a las preguntas:
# - Set root password: YES (usar contraseña segura)
# - Remove anonymous users: YES
# - Disallow root login remotely: NO (necesitamos acceso desde middleware)
# - Remove test database: YES
# - Reload privilege tables: YES
```

#### Paso 3: Configurar MySQL para Acceso Remoto

```bash
# Editar configuración de MySQL
sudo vi /etc/my.cnf.d/mysql-server.cnf

# Agregar en la sección [mysqld]:
[mysqld]
bind-address = 0.0.0.0
max_connections = 200
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# Guardar y salir (:wq)

# Reiniciar MySQL
sudo systemctl restart mysqld
```

#### Paso 4: Crear Base de Datos y Tablas

```bash
# Copiar scripts SQL al servidor
# (Transferir archivos desde el directorio backend/)

# Conectarse a MySQL
sudo mysql -u root -p

# Ejecutar scripts
source /ruta/a/01-create-database.sql
source /ruta/a/02-insert-initial-data.sql

# Verificar creación
USE gestion_clientes;
SHOW TABLES;
SELECT COUNT(*) FROM Pais;
SELECT COUNT(*) FROM Ciudades;

# Salir
exit;
```

#### Paso 5: Configurar Firewall

```bash
# Permitir conexión desde middleware (10.241.0.14)
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.241.0.14" port protocol="tcp" port="3306" accept'

# Recargar firewall
sudo firewall-cmd --reload

# Verificar reglas
sudo firewall-cmd --list-all
```

#### Paso 6: Verificar Instalación

```bash
# Desde la máquina middleware (10.241.0.14), probar conexión:
mysql -h 10.241.0.15 -u app_user -p gestion_clientes
# Contraseña: App_P@ssw0rd_2024!

# Si conecta exitosamente, el backend está listo
```

---

## ⚙️ FASE 3: Instalación y Configuración del Middleware

### En la máquina 10.241.0.14

#### Paso 1: Instalar Java y Maven

```bash
# Instalar OpenJDK 17
sudo dnf install java-17-openjdk java-17-openjdk-devel -y

# Verificar instalación
java -version
javac -version

# Configurar JAVA_HOME
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk' | sudo tee -a /etc/profile.d/java.sh
echo 'export PATH=$JAVA_HOME/bin:$PATH' | sudo tee -a /etc/profile.d/java.sh
source /etc/profile.d/java.sh

# Instalar Maven
sudo dnf install maven -y

# Verificar instalación
mvn -version
```

#### Paso 2: Instalar Apache Tomcat (Opcional)

```bash
# Instalar Tomcat
sudo dnf install tomcat -y

# Iniciar y habilitar Tomcat
sudo systemctl start tomcat
sudo systemctl enable tomcat

# Verificar estado
sudo systemctl status tomcat
```

#### Paso 3: Compilar la Aplicación

```bash
# Copiar el directorio middleware al servidor
# (Transferir archivos desde el directorio middleware/)

# Navegar al directorio
cd /ruta/a/middleware

# Compilar el proyecto
./mvnw clean package -DskipTests

# Verificar que se generó el JAR
ls -lh target/quarkus-app/quarkus-run.jar
```

#### Paso 4: Configurar la Aplicación

```bash
# Editar application.properties si es necesario
vi src/main/resources/application.properties

# Verificar configuración de base de datos:
# quarkus.datasource.jdbc.url=jdbc:mysql://10.241.0.15:3306/gestion_clientes
# quarkus.datasource.username=app_user
# quarkus.datasource.password=App_P@ssw0rd_2024!

# Verificar configuración CORS:
# quarkus.http.cors.origins=http://10.241.0.13,http://10.241.0.13:80
```

#### Paso 5: Ejecutar la Aplicación

**Opción A: Ejecutar como aplicación standalone**

```bash
# Ejecutar directamente
java -jar target/quarkus-app/quarkus-run.jar

# O crear un servicio systemd
sudo vi /etc/systemd/system/gestion-clientes-api.service

# Contenido del archivo:
[Unit]
Description=Gestion Clientes API
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/ruta/a/middleware
ExecStart=/usr/bin/java -jar /ruta/a/middleware/target/quarkus-app/quarkus-run.jar
Restart=on-failure

[Install]
WantedBy=multi-user.target

# Guardar y salir

# Recargar systemd
sudo systemctl daemon-reload

# Iniciar servicio
sudo systemctl start gestion-clientes-api
sudo systemctl enable gestion-clientes-api

# Verificar estado
sudo systemctl status gestion-clientes-api
```

**Opción B: Desplegar en Tomcat**

```bash
# Copiar el JAR a Tomcat
sudo cp target/quarkus-app/quarkus-run.jar /opt/tomcat/webapps/api.war

# Reiniciar Tomcat
sudo systemctl restart tomcat
```

#### Paso 6: Configurar Firewall

```bash
# Permitir tráfico en puerto 8080
sudo firewall-cmd --permanent --add-port=8080/tcp

# Permitir acceso desde frontend (10.241.0.13)
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.241.0.13" port protocol="tcp" port="8080" accept'

# Recargar firewall
sudo firewall-cmd --reload

# Verificar reglas
sudo firewall-cmd --list-all
```

#### Paso 7: Verificar Instalación

```bash
# Probar endpoints
curl http://localhost:8080/api/paises
curl http://localhost:8080/health

# Desde otra máquina
curl http://10.241.0.14:8080/api/paises

# Acceder a Swagger UI
# Abrir en navegador: http://10.241.0.14:8080/swagger-ui
```

---

## 🌐 FASE 4: Instalación y Configuración del Frontend

### En la máquina 10.241.0.13

#### Paso 1: Instalar Node.js

```bash
# Listar módulos disponibles
sudo dnf module list nodejs

# Habilitar Node.js 18
sudo dnf module enable nodejs:18 -y

# Instalar Node.js
sudo dnf install nodejs -y

# Verificar instalación
node --version
npm --version
```

#### Paso 2: Instalar Apache HTTP Server

```bash
# Instalar Apache
sudo dnf install httpd -y

# Instalar módulos adicionales
sudo dnf install mod_ssl -y

# Iniciar y habilitar Apache
sudo systemctl start httpd
sudo systemctl enable httpd

# Verificar estado
sudo systemctl status httpd
```

#### Paso 3: Compilar la Aplicación React

```bash
# Copiar el directorio frontend al servidor
# (Transferir archivos desde el directorio frontend/)

# Navegar al directorio
cd /ruta/a/frontend

# Instalar dependencias
npm install

# Compilar para producción
npm run build

# Verificar que se generó el directorio build
ls -lh build/
```

#### Paso 4: Desplegar en Apache

```bash
# Copiar archivos compilados a Apache
sudo cp -r build/* /var/www/html/

# Configurar permisos
sudo chown -R apache:apache /var/www/html/
sudo chmod -R 755 /var/www/html/

# Verificar archivos
ls -la /var/www/html/
```

#### Paso 5: Configurar Apache para React Router

```bash
# Crear archivo de configuración
sudo vi /etc/httpd/conf.d/gestion-clientes.conf

# Agregar contenido:
<VirtualHost *:80>
    ServerName 10.241.0.13
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
        
        # Configuración para React Router
        RewriteEngine On
        RewriteBase /
        RewriteRule ^index\.html$ - [L]
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule . /index.html [L]
    </Directory>

    # Proxy para el API (opcional)
    ProxyPreserveHost On
    ProxyPass /api http://10.241.0.14:8080/api
    ProxyPassReverse /api http://10.241.0.14:8080/api

    ErrorLog /var/log/httpd/gestion-clientes-error.log
    CustomLog /var/log/httpd/gestion-clientes-access.log combined
</VirtualHost>

# Guardar y salir

# Verificar configuración
sudo apachectl configtest

# Si muestra "Syntax OK", reiniciar Apache
sudo systemctl restart httpd
```

#### Paso 6: Configurar Firewall

```bash
# Permitir tráfico HTTP
sudo firewall-cmd --permanent --add-service=http

# Permitir tráfico HTTPS (opcional)
sudo firewall-cmd --permanent --add-service=https

# Recargar firewall
sudo firewall-cmd --reload

# Verificar reglas
sudo firewall-cmd --list-all
```

#### Paso 7: Verificar Instalación

```bash
# Probar localmente
curl http://localhost

# Desde otra máquina
curl http://10.241.0.13

# Abrir en navegador
# http://10.241.0.13
```

---

## ✅ FASE 5: Verificación del Sistema Completo

### Pruebas de Conectividad

```bash
# Desde Frontend (10.241.0.13)
curl http://10.241.0.14:8080/api/paises

# Desde Middleware (10.241.0.14)
mysql -h 10.241.0.15 -u app_user -p gestion_clientes
```

### Pruebas Funcionales

1. **Abrir navegador**: `http://10.241.0.13`
2. **Verificar navegación**: Clientes, Ciudades, Países
3. **Probar CRUD de Países**:
   - Crear un nuevo país
   - Editar un país existente
   - Intentar eliminar un país con ciudades (debe fallar)
4. **Probar CRUD de Ciudades**:
   - Crear una nueva ciudad
   - Asociarla a un país
   - Intentar eliminar una ciudad con clientes (debe fallar)
5. **Probar CRUD de Clientes**:
   - Crear un nuevo cliente
   - Verificar validación de email
   - Verificar que ciudad pertenezca al país
   - Buscar clientes por nombre
   - Filtrar por país y ciudad

### Verificar Logs

```bash
# Backend
sudo tail -f /var/log/mysql/mysql.log

# Middleware
sudo journalctl -u gestion-clientes-api -f
# O si usa Tomcat:
sudo tail -f /var/log/tomcat/catalina.out

# Frontend
sudo tail -f /var/log/httpd/access_log
sudo tail -f /var/log/httpd/error_log
```

---

## 🔒 FASE 6: Seguridad y Hardening

### Cambiar Contraseñas por Defecto

```bash
# En Backend (10.241.0.15)
mysql -u root -p
ALTER USER 'app_user'@'10.241.0.14' IDENTIFIED BY 'NuevaContraseñaSegura123!';
FLUSH PRIVILEGES;
```

### Configurar SELinux (Producción)

```bash
# Habilitar SELinux
sudo setenforce 1
sudo sed -i 's/^SELINUX=permissive/SELINUX=enforcing/' /etc/selinux/config

# Backend
sudo setsebool -P mysql_connect_any 1

# Middleware
sudo setsebool -P tomcat_can_network_connect_db 1

# Frontend
sudo setsebool -P httpd_can_network_connect 1
```

### Configurar HTTPS (Recomendado)

```bash
# En Frontend (10.241.0.13)
# Generar certificado autofirmado (para desarrollo)
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/pki/tls/private/gestion-clientes.key \
  -out /etc/pki/tls/certs/gestion-clientes.crt

# Configurar SSL en Apache
sudo vi /etc/httpd/conf.d/ssl.conf
# Actualizar rutas de certificados
```

---

## 📦 FASE 7: Backup y Mantenimiento

### Configurar Backups Automáticos

**Backend**:
```bash
# Crear script de backup
sudo vi /usr/local/bin/backup-mysql.sh

#!/bin/bash
BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR
mysqldump -u root -p'PASSWORD' gestion_clientes > $BACKUP_DIR/backup_$DATE.sql
find $BACKUP_DIR -name "backup_*.sql" -mtime +7 -delete

# Dar permisos de ejecución
sudo chmod +x /usr/local/bin/backup-mysql.sh

# Agregar a crontab (diario a las 2 AM)
sudo crontab -e
0 2 * * * /usr/local/bin/backup-mysql.sh
```

---

## 🐛 Troubleshooting Común

### Problema: No se puede conectar a MySQL desde Middleware

```bash
# Verificar firewall en backend
sudo firewall-cmd --list-all

# Verificar que MySQL escuche en todas las interfaces
sudo netstat -tlnp | grep 3306

# Verificar permisos de usuario
mysql -u root -p
SELECT user, host FROM mysql.user WHERE user='app_user';
```

### Problema: Error CORS en Frontend

```bash
# Verificar configuración en middleware
cat src/main/resources/application.properties | grep cors

# Debe incluir: http://10.241.0.13
```

### Problema: Apache no muestra la aplicación React

```bash
# Verificar archivos
ls -la /var/www/html/

# Verificar permisos
sudo chown -R apache:apache /var/www/html/

# Verificar logs
sudo tail -f /var/log/httpd/error_log
```

---

## 📊 Monitoreo

### Comandos Útiles

```bash
# Ver uso de recursos
top
htop

# Ver conexiones de red
sudo netstat -tulpn

# Ver logs en tiempo real
sudo journalctl -f
```

---

## ✅ Checklist Final

- [ ] Backend: MySQL instalado y funcionando
- [ ] Backend: Base de datos creada con datos iniciales
- [ ] Backend: Firewall configurado correctamente
- [ ] Backend: Conexión desde middleware verificada
- [ ] Middleware: Java y Maven instalados
- [ ] Middleware: Aplicación compilada sin errores
- [ ] Middleware: Servicio ejecutándose
- [ ] Middleware: Conexión a base de datos exitosa
- [ ] Middleware: API respondiendo correctamente
- [ ] Frontend: Node.js y Apache instalados
- [ ] Frontend: Aplicación compilada
- [ ] Frontend: Archivos desplegados en Apache
- [ ] Frontend: Aplicación accesible desde navegador
- [ ] Frontend: Conexión a API funcionando
- [ ] Seguridad: Contraseñas cambiadas
- [ ] Seguridad: Firewall configurado en todas las máquinas
- [ ] Seguridad: SELinux configurado (producción)
- [ ] Backup: Scripts de backup configurados
- [ ] Pruebas: CRUD completo verificado
- [ ] Documentación: README revisado

---

**¡Despliegue Completado!**

El sistema debería estar completamente funcional en:
- Frontend: http://10.241.0.13
- API: http://10.241.0.14:8080
- Swagger: http://10.241.0.14:8080/swagger-ui
