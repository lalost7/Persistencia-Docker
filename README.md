Persistencia de Datos y Volúmenes en Docker

Introducción

Docker permite ejecutar aplicaciones dentro de contenedores ligeros y aislados. Sin embargo, los contenedores son efímeros, lo que significa que cuando un contenedor es eliminado también desaparece toda la información almacenada en él.

En esta práctica se analizó el problema de pérdida de datos y la solución mediante volúmenes persistentes utilizando MariaDB.

Primero se trabajó con un contenedor sin persistencia para demostrar cómo la información desaparece al eliminar el contenedor. Posteriormente se implementó un Named Volume para almacenar los datos fuera del contenedor y comprobar que la información permanece incluso después de destruir y recrear el servicio.


Objetivos

* Implementar estrategias de persistencia utilizando Docker.
* Diferenciar contenedores efímeros y persistentes.
* Utilizar Named Volumes para conservar información.
* Validar la persistencia de datos en MariaDB.
* Comprender la importancia de separar los datos del ciclo de vida del contenedor.
Desarrollo de la Práctica
 Parte 1 — Simulación del Error (Sin Persistencia)
Descargar imagen de MariaDB

bash
docker pull mariadb


Crear contenedor efímero

bash
docker run -d --name db-efimera -e MYSQL_ROOT_PASSWORD=123 mariadb
 Verificar contenedor
`bash
docker ps



 Entrar al contenedor

bash
docker exec -it db-efimera mariadb -u root -p

Contraseña:

bash
123

 Crear base de datos y tabla

```sql
CREATE DATABASE prueba;

USE prueba;

CREATE TABLE usuarios(
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(50)
);

INSERT INTO usuarios(nombre) VALUES ('Eduardo');

SELECT * FROM usuarios;
 Salir de MariaDB

```bash
exit;
```
Eliminar el contenedor

```bash
docker rm -f db-efimera
```

---

Resultado Observado

Al eliminar el contenedor, toda la información desapareció completamente debido a que los datos estaban almacenados únicamente dentro del contenedor.

Esto demuestra que los contenedores son efímeros y que, sin persistencia, los datos no sobreviven al ciclo de vida del contenedor.

---

 Parte 2 — Persistencia con Named Volumes

Crear volumen persistente

```bash
docker volume create mi-data-db
```

---

Verificar volumen

```bash
docker volume ls
```

---

Crear contenedor persistente

```bash
docker run -d \
--name db-persistente \
-v mi-data-db:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123 \
mariadb
```

---

 Entrar al contenedor

```bash
docker exec -it db-persistente mariadb -u root -p
```

Contraseña:

```bash
123
```

---

## Crear datos persistentes

```sql
CREATE DATABASE empresa;

USE empresa;

CREATE TABLE empleados(
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(50)
);

INSERT INTO empleados(nombre) VALUES ('Carlos');

SELECT * FROM empleados;
```

---
Salir de MariaDB

```bash
exit;
```

---

Eliminar contenedor persistente

```bash
docker rm -f db-persistente
```
Recrear el contenedor utilizando el mismo volumen

```bash
docker run -d \
--name db-persistente2 \
-v mi-data-db:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123 \
mariadb
```

---

 Entrar nuevamente

```bash
docker exec -it db-persistente2 mariadb -u root -p
```

Contraseña:

```bash
123


 Verificar persistencia

```sql
USE empresa;

SELECT * FROM empleados;
```

---

 Resultado

Los datos permanecieron almacenados incluso después de eliminar el contenedor original.

Esto demuestra que los Named Volumes permiten conservar la información de forma independiente al ciclo de vida del contenedor.

---

Inspección del Volumen

```bash
docker volume inspect mi-data-db
```

Este comando permite visualizar la información detallada del volumen creado, incluyendo su nombre, ruta de almacenamiento y configuración.

Conclusión

En esta práctica se comprobó la importancia de la persistencia de datos en Docker.

Los contenedores sin volúmenes pierden toda la información al ser eliminados, lo que representa un riesgo para aplicaciones reales como bases de datos.

Mediante el uso de Named Volumes se logró desacoplar los datos del contenedor, permitiendo conservar la información incluso después de destruir y recrear el servicio.

La persistencia es fundamental en entornos de producción porque garantiza la integridad de los datos y facilita tareas como respaldos, migraciones y recuperación de servicios.

Referencias Bibliográficas

Docker. (2024). Docker documentation: Manage data in Docker.
[https://docs.docker.com/storage/](https://docs.docker.com/storage/)

Pelado Nos. (2022). Docker en 12 minutos [Archivo de video]. YouTube.
[https://www.youtube.com/watch?v=4Dko5W96WHg](https://www.youtube.com/watch?v=4Dko5W96WHg)

Tecnocrática. Alta disponibilidad y gestión de clusters en Proxmox.
