# Configuración de un servidor DNS autoritativo con BIND9 en Docker

## 1. Descripción del proyecto

Este proyecto implementa un servidor DNS autoritativo utilizando BIND9 y Docker, configurando una zona DNS local denominada `empresa.test`.

Se implementan registros DNS de diferentes tipos y se utilizan herramientas de diagnóstico para comprobar su correcto funcionamiento.

## 2. Objetivos

**Objetivo general**

Implementar y administrar un servidor DNS autoritativo mediante BIND9 en Docker para gestionar una zona de resolución local.

**Objetivos específicos**

- Configurar BIND9 mediante Docker Compose.
- Crear y administrar la zona `empresa.test`.
- Implementar registros SOA, NS, A, MX y CNAME.
- Verificar la resolución DNS mediante `dig`.
- Realizar modificaciones en los registros y actualizar el serial SOA.

## 3. Tecnologías utilizadas

- Docker Desktop.
- Docker Compose.
- BIND9.
- Alpine Linux para herramientas de diagnóstico.
- PowerShell.
- Visual Studio Code.
- Git y GitHub.

## 4. Estructura del proyecto

```text
dns-bind9-docker/
├── config/
│   ├── named.conf
│   ├── named.conf.options
│   └── named.conf.local
├── zones/
│   └── db.empresa.test
├── docker-compose.yml
├── .gitignore
└── README.md
```

## 5. Configuración del servidor DNS

El servidor BIND9 se ejecuta dentro de un contenedor Docker.

Se utiliza el puerto `1053` del equipo anfitrión, redirigido al puerto `53` del contenedor mediante TCP y UDP.

La zona configurada es `empresa.test`, y el servidor funciona de manera autoritativa, sin ofrecer resolución recursiva.

## 6. ¿Qué contiene un archivo de zona DNS?

Un archivo de zona DNS contiene los registros que permiten administrar la resolución de nombres dentro de un dominio.

Los registros implementados son:

| Registro | Descripción |
|---|---|
| SOA | Contiene información administrativa de la zona DNS. |
| NS | Identifica el servidor de nombres autoritativo. |
| A | Relaciona un nombre de dominio con una dirección IPv4. |
| CNAME | Define un alias de otro nombre DNS. |
| MX | Identifica el servidor de correo electrónico. |

En este proyecto, los registros están almacenados en `zones/db.empresa.test`.

## 7. ¿Por qué se utiliza el serial SOA?

El serial SOA representa la versión del archivo de zona DNS.

Permite que los servidores DNS secundarios identifiquen cuándo se realizaron modificaciones en la zona principal.

Se utiliza comúnmente el formato `AAAAMMDDNN`.

Ejemplo:

- `2026112101`: versión inicial.
- `2026112102`: segunda versión después de una modificación.

Cuando se agrega o modifica un registro, se incrementa el serial para identificar la nueva versión.

## 8. ¿Cómo comprobar los registros DNS?

Primero se valida la configuración del servidor:

```powershell
docker exec dns-bind9 named-checkconf /etc/bind/named.conf
```

Después se valida el archivo de zona:

```powershell
docker exec dns-bind9 named-checkzone empresa.test /var/lib/bind/db.empresa.test
```

Para consultar registros DNS se utiliza un contenedor temporal Alpine con `bind-tools`:

```powershell
docker run --rm --network container:dns-bind9 alpine:3.20 sh -c "apk add --no-cache bind-tools >/dev/null && dig @127.0.0.1 www.empresa.test A"
```

También se pueden consultar los registros MX, NS, SOA y CNAME modificando el nombre y tipo de registro en el comando.

La bandera `aa` en la respuesta de `dig` confirma que el servidor proporciona una respuesta autoritativa.

## 9. Administración de registros DNS

Para demostrar la administración del servidor se agregó el registro:

```dns
sistema IN A 192.168.1.50
```

Después se incrementó el serial SOA y se validó nuevamente la zona.

Para aplicar los cambios se reinició el servicio:

```powershell
docker compose restart dns
```

Posteriormente se comprobó la resolución del nuevo registro mediante `dig`.

## 10. ¿Por qué DNS por sí solo no publica una aplicación web?

DNS permite relacionar nombres de dominio con direcciones IP, pero no almacena ni ejecuta aplicaciones web.

Cuando un usuario escribe `www.empresa.test`, el servidor DNS responde con una dirección IP.

Sin embargo, para visualizar una página web es necesario contar con un servidor HTTP, como Nginx o Apache, o con un servidor de aplicaciones.

Por lo tanto, DNS permite localizar el servidor, pero no sustituye al servicio web encargado de entregar el contenido.

## 11. Ejecución del proyecto

Para iniciar el servidor:

```powershell
docker compose up -d
```

Para comprobar su estado:

```powershell
docker compose ps
```

Para detenerlo:

```powershell
docker compose down
```

## 12. Conclusiones

Se implementó un servidor DNS autoritativo utilizando BIND9 y Docker, configurando una zona local con diferentes registros DNS.

Las herramientas de diagnóstico permitieron validar la configuración y comprobar la resolución de nombres.

Además, se comprendió la importancia del serial SOA para la administración de zonas y la diferencia entre un servicio DNS y un servidor web.