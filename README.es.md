# Laboratorio de NAT: masquerade y redirección de puertos con nftables en una VM Ubuntu

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md)

[![check](https://github.com/mitsurutech/lab-nat-gateway/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-nat-gateway/actions/workflows/check.yml)

Un **laboratorio de NAT** listo para usar en clase, de la [plataforma de aprendizaje MitsuruTech](https://mitsurutech.co.uk/es/learning), donde los centros dan a cada alumno sus propias máquinas y un terminal en el navegador. Cada alumno recibe una VM Ubuntu con una pequeña red construida dentro: un equipo de una LAN privada, una **pasarela** y un servidor exterior. Configura en la pasarela el **masquerade (NAT de origen)** y una **redirección de puertos (NAT de destino)** con **nftables**, guarda las reglas para que sobrevivan a un reinicio y ve con sus propios ojos lo que ve el otro extremo. No se instala nada en el ordenador del alumno.

Una VM es una sola máquina, así que cada uno de los tres es un **espacio de nombres de red** (network namespace) y cada enlace, un par de cables virtuales. Funcionan los comandos de verdad, que es lo importante: `nft`, `conntrack -L`, `tcpdump`, `curl`, `ip route`.

## El laboratorio

| Laboratorio | Archivo | Qué hace el alumno | Página de ayuda |
| --- | --- | --- | --- |
| Pasarela NAT | [`nat-gateway.lab.yaml`](nat-gateway.lab.yaml) | Averiguar por qué un equipo privado no recibe respuesta desde el exterior, traducir sus direcciones en la pasarela, redirigir un puerto desde el exterior a una página web de la LAN, permitir ese flujo y nada más amplio, y guardar el conjunto de reglas | [Laboratorio de pasarela NAT](https://mitsurutech.co.uk/es/help/lab-nat-gateway) |

La red: un equipo de la LAN en `10.0.0.0/24`, la pasarela entre él y una red exterior, `203.0.113.0/24` (un rango reservado para documentación), y un servidor exterior que responde a cada petición con la dirección desde la que la vio llegar. El exterior no tiene ruta de vuelta a la LAN, como en internet. El laboratorio deja en la carpeta personal del alumno un README que dice qué hacer y no da ninguna de las respuestas.

Tiene una **comprobación** (check), un script que califica una VM entregada sin que el profesor tenga que leerla. Se ejecuta sobre una copia de la máquina guardada tras un arranque en limpio, así que solo ve lo que el alumno guardó en `/etc/nftables.conf`, y **califica lo que hace la red y no cómo están escritas las reglas**: una petición del equipo de la LAN debe llegar al servidor exterior, que debe haber visto la dirección exterior de la pasarela; una petición desde el exterior al puerto redirigido debe llegar a la página de la LAN, que debe haber visto la dirección del propio cliente (así que un proxy en la pasarela no aprueba); la otra página de la LAN y todos los demás puertos deben seguir cerrados; y la cadena forward de la pasarela debe seguir descartando lo que no se le indicó que permitiera.

## Qué hay en el archivo

El `.lab.yaml` es una **definición de laboratorio de VM** completa, exactamente lo que ejecuta la plataforma:

| Clave | Qué es |
| --- | --- |
| `cpus`, `memory`, `disk` | el tamaño de la VM de cada alumno: 1 CPU, 1024 MB y 10 GB |
| `open` | los puertos que se abren en el navegador: ninguno, el laboratorio es el terminal |
| `setup` | un script que se ejecuta una vez, como root, la primera vez que arranca la VM de cada alumno: instala `nftables`, `conntrack`, `curl` y algunas herramientas del archivo de Ubuntu en unos diez segundos, y escribe el comando `netlab`, la topología, el `/etc/nftables.conf` inicial y el README del laboratorio |
| `check` | el script de calificación, que se ejecuta como root sobre una copia del disco guardado tras un arranque en limpio; su estado de salida es el resultado y `0` aprueba |

El comando `netlab` que instala: `netlab status` lista las máquinas, `netlab shell gw` abre un shell dentro de una de ellas, y `netlab reset` restablece las reglas iniciales y vuelve a arrancar el laboratorio.

## Uso en la plataforma

**Para asignarlo como trabajo a una clase:**

1. En tu módulo, añade una tarea y elige la tarjeta **Laboratorio de VM Ubuntu**.
2. Pulsa el ejemplo de Pasarela NAT, o pega el contenido del archivo de aquí. Usar un ejemplo sustituye el texto, y Deshacer recupera el tuyo.
3. Publica y prueba primero tu propia copia. Cada alumno recibe una VM propia, que se conserva entre sesiones.
4. Asígnalo como cualquier tarea. Cuando se entregue, ejecuta la comprobación en la fila de un alumno en la lista de la clase: arranca una copia de su máquina guardada que no se vuelve a guardar, y muestra si aprobó y qué imprimió.

La [guía de laboratorios de VM Ubuntu](https://mitsurutech.co.uk/es/help/labs) del manual y su [página de laboratorios de ejemplo](https://mitsurutech.co.uk/es/help/sample-labs) cuentan el resto.

¿Aún no tienes una cuenta para tu centro? [Tu primer año es gratis](https://mitsurutech.co.uk/es/pricing) para hasta 50 alumnos, y [crear una cuenta](https://mitsurutech.co.uk/es/signup) solo requiere una dirección de correo electrónico.

## Comprobación del archivo

En cada push, `.github/workflows/check.yml` comprueba que el archivo es un YAML válido con las cinco claves anteriores y que `setup` y `check` caben cada uno en el límite de 16 KB de la plataforma. No arranca ninguna VM: eso se hace en las máquinas de la propia plataforma.

## Qué no funciona en un laboratorio

La VM llega al archivo de paquetes de Ubuntu, PyPI, npm y GitHub, y a nada más, así que un laboratorio no puede apuntarse a un equipo de internet. Es una sola VM, por lo que el «exterior» es un espacio de nombres dentro de ella y no la internet real.

## Acerca de

Creado por [Mitsuru Technologies](https://mitsurutech.co.uk/es) para la [plataforma de aprendizaje MitsuruTech](https://mitsurutech.co.uk/es/learning): Python, Java, Go, Node, SQL y Linux en el navegador para escuelas y centros, con laboratorios de contenedores, laboratorios de VM, aplicaciones web desplegadas desde GitHub y cuestionarios que se califican solos. Prueba código sin cuenta en el [entorno de pruebas](https://mitsurutech.co.uk/es/playground).

Más laboratorios del mismo autor: [lab-netlab](https://github.com/mitsurutech/lab-netlab) (enrutamiento estático, OSPF y BGP), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea) y [lab-sample](https://github.com/mitsurutech/lab-sample).

## Licencia

Esta obra está bajo licencia [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Puedes compartirla y adaptarla para aprender y enseñar, si citas a Mitsuru Technologies Ltd, no la usas comercialmente y compartes lo que hagas con ella bajo la misma licencia. El texto completo está en el archivo `LICENSE`.

nftables y conntrack-tools tienen licencia GPL-2.0 y se instalan desde el archivo de Ubuntu cuando arranca una VM; nada de ellos se distribuye aquí.
