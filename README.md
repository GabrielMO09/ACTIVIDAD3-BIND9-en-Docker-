# Servidor DNS autoritativo con BIND9 y Docker

## 1. Descripción del proyecto

Este proyecto implementa un servidor DNS autoritativo utilizando BIND9 dentro de un contenedor Docker. Su objetivo es administrar una zona DNS local denominada `empresa.test`, configurar diferentes registros DNS y verificar su funcionamiento mediante herramientas de diagnóstico.

## 2. Tecnologías utilizadas

- Docker y Docker Compose.
- BIND9 como servidor DNS.
- Windows y PowerShell.
- Herramientas de diagnóstico `dig` y `nslookup`.
- Git y GitHub para el control de versiones.

## 3. ¿Qué contiene un archivo de zona DNS?

Un archivo de zona DNS contiene los registros que permiten asociar nombres de dominio con direcciones IP y otros servicios de una red.

Los registros principales son:

- **SOA (Start of Authority):** contiene la información administrativa de la zona.
- **NS (Name Server):** identifica los servidores DNS autoritativos.
- **A:** relaciona un nombre de dominio con una dirección IPv4.
- **CNAME:** permite establecer alias de otros nombres DNS.
- **MX (Mail Exchange):** identifica los servidores encargados de recibir correo electrónico.
- **TTL (Time to Live):** indica cuánto tiempo pueden almacenarse en caché los registros.

En este proyecto, el archivo `db.empresa.test` contiene los registros de la zona local.

## 4. ¿Por qué se utiliza el serial SOA?

El serial SOA es un número de versión que identifica el estado del archivo de zona DNS.

Su principal función es permitir que los servidores DNS secundarios detecten modificaciones realizadas en el servidor principal y actualicen sus copias de la zona.

Generalmente se utiliza el formato `AAAAMMDDNN`, donde:

- AAAA: año.
- MM: mes.
- DD: día.
- NN: número de modificación.

Por ejemplo, `2026112101` representa la primera versión del día 21 de noviembre de 2026.

Cuando se modifica la zona, se incrementa el serial para identificar la nueva versión.

## 5. ¿Cómo comprobar los registros DNS?

Para verificar los registros se utiliza la herramienta `dig`, que permite consultar directamente al servidor DNS.

Ejemplos:

```bash
dig @127.0.0.1 empresa.test SOA
dig @127.0.0.1 empresa.test NS
dig @127.0.0.1 www.empresa.test A
dig @127.0.0.1 ftp.empresa.test CNAME
dig @127.0.0.1 empresa.test MX
```

Estos comandos se ejecutan dentro del contenedor BIND9 mediante `docker exec dns-bind9`.

También se utilizan las herramientas `named-checkconf` y `named-checkzone` para validar los archivos de configuración y la zona DNS.

La bandera `aa` en una respuesta de `dig` permite comprobar que la respuesta es autoritativa.

## 6. ¿Por qué DNS por sí solo no publica una aplicación web?

El sistema DNS se encarga de traducir nombres de dominio en direcciones IP, pero no proporciona los archivos, las páginas ni los servicios necesarios para ejecutar una aplicación web.

Para publicar una aplicación web se necesita un servidor HTTP, como Nginx o Apache, o un servidor de aplicaciones que atienda las solicitudes.

Por ejemplo:

1. El usuario introduce `www.empresa.test` en el navegador.
2. El servidor DNS devuelve la dirección IP `192.168.1.20`.
3. El navegador intenta conectarse a esa dirección.
4. Un servidor web debe estar funcionando en dicha dirección para entregar la página.

Si no existe un servidor web accesible, la resolución DNS puede funcionar correctamente, pero la página no se mostrará.

## 7. Administración de la zona

Para administrar la zona se realizan los siguientes procedimientos:

1. Editar el archivo `db.empresa.test`.
2. Agregar o modificar registros DNS.
3. Incrementar el serial SOA.
4. Validar la zona mediante `named-checkzone`.
5. Recargar la zona o reiniciar el servicio BIND9.
6. Verificar los cambios mediante `dig`.

## 8. Conclusión

La práctica permitió comprender la implementación y administración de un servidor DNS autoritativo mediante BIND9 y Docker.

Se configuró una zona local, se definieron registros DNS y se estableció un procedimiento para validar y administrar sus modificaciones.

Además, se comprendió la diferencia entre resolver un dominio y publicar una aplicación web, identificando la necesidad de utilizar servicios adicionales como un servidor HTTP.