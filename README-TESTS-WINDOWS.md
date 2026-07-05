# Tests de requisitos funcionales en Windows

Guía de entorno para correr `test-requisitos-funcionales.ps1`. El script pega a los
mismos endpoints REST que usa el frontend; no hay endpoints de testing. Su gemelo Linux
es `test-requisitos-funcionales.sh` (misma cobertura, curl en vez de
`Invoke-RestMethod`).

## Requisitos previos

- SQL Server 2019 o 2022 corriendo, con la base `HeladosMimoDB` creada y el usuario `sa`
  con contraseña configurada.
- SQL Server Command Line Tools (`sqlcmd`). Se instala con:
  ```powershell
  winget install Microsoft.SQLServerCommandLineUtilities
  ```
  o desde https://learn.microsoft.com/sql/tools/sqlcmd-utility.
- Java 17 o superior y Maven (el wrapper `mvnw.cmd` viene en el repo).

## Configuración inicial

### 1. Base de datos

En SQL Server Management Studio:

```sql
CREATE DATABASE HeladosMimoDB;
GO
```

La contraseña de `sa` que asumen los ejemplos es `YourStrong@Passw0rd`; si usás otra,
cambiala en los dos archivos de los pasos 2 y 3.

### 2. application.properties

Editá `src/main/resources/application.properties`:

```properties
# Conexión SQL Server (Windows)
spring.datasource.url=jdbc:sqlserver://localhost:1433;databaseName=HeladosMimoDB;encrypt=true;trustServerCertificate=true
spring.datasource.username=sa
spring.datasource.password=YourStrong@Passw0rd
spring.datasource.driver-class-name=com.microsoft.sqlserver.jdbc.SQLServerDriver

# JPA/Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.SQLServerDialect

# Server
server.port=8080

# Session
server.servlet.session.timeout=30m
```

### 3. Variables del script

Editá `test-requisitos-funcionales.ps1` si tu entorno difiere:

```powershell
$DBServer = "localhost"           # Cambia si SQL Server está en otro host
$DBName = "HeladosMimoDB"
$DBUser = "sa"
$DBPassword = "YourStrong@Passw0rd"  # Usa tu contraseña real
```

## Ejecución

1. Levantá la aplicación en una ventana de PowerShell:
   ```powershell
   .\mvnw.cmd clean spring-boot:run
   ```
   Esperá la línea `Started HeladosMimosApplication in X.XXX seconds`.

2. En otra ventana, corré los tests:
   ```powershell
   # Solo la primera vez, para permitir scripts locales
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

   .\test-requisitos-funcionales.ps1
   ```

3. El script pregunta si usa SQL Server para crear el usuario `ADMINISTRADOR_VENTAS`:
   con `s` lo crea vía `sqlcmd`; con `N` salta el setup y usa solo la API REST.

El reporte final imprime tests ejecutados, exitosos, fallidos y porcentaje, más el
resumen por RF (RF-01 a RF-05). Los logs quedan en
`.\logs\test-rf-YYYY-MM-DD_HH-mm-ss.log`.

Para verificar los pagos contra entrega, buscá en la consola de Spring Boot líneas como:

```
[PAGO CONTRA ENTREGA] Código de confirmación: 487293 - Pedido: 19
[PAGO DATÁFONO] Código de confirmación: 821701 - Pedido: 20
```

## Solución de problemas

**"No se pudo conectar a SQL Server":** el servicio no corre o TCP/IP está deshabilitado.
En SQL Server Configuration Manager verificá que `SQL Server (MSSQLSERVER)` esté
corriendo, habilitá TCP/IP en Network Configuration y reiniciá el servicio. El firewall
tiene que permitir el puerto 1433.

**"sqlcmd no se reconoce como comando":** faltan las Command Line Tools; instalalas con
el `winget` de arriba.

**"Spring Boot NO está corriendo":** levantá la aplicación con
`.\mvnw.cmd clean spring-boot:run` y esperá a que termine de iniciar. El puerto 8080
tiene que estar libre.

**"Cannot find path '...\logs'":** crea la carpeta:
```powershell
New-Item -ItemType Directory -Path .\logs
```

Si hay errores de permisos, abrí PowerShell como administrador.

## Diferencias con la versión bash

| Aspecto | Bash (Linux) | PowerShell (Windows) |
|---|---|---|
| Contenedores | Podman | No usa (SQL Server nativo) |
| Cliente HTTP | curl | Invoke-RestMethod |
| Cliente DB | podman exec sqlcmd | sqlcmd directo |
| Logs | ./logs/ | .\logs\ |
| Ejecutable | ./test-*.sh | .\test-*.ps1 |
