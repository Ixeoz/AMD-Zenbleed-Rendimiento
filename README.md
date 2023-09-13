# **AMD Zen 2 Zenbleed - Rendimiento**

Hace unos pocos días se descubrió una vulnerabilidad en **Zen 2**. Básicamente, esto permite exfiltrar información de procesos **hermanos** sin ningún privilegio especial.

- [**CVE-2023-20593.**](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-20593)

Por lo tanto, se ha probado un parche por software sobre está vulnerabilidad **Zenbleed** y se encontró que algunas cargas de trabajo, como decodificadores y renderizadores, pueden sufrir pérdidas de rendimiento hasta un **~15%.** [**Esto según lo descrito por Paul Alcorn en TomsHardware**](https://www.tomshardware.com/news/amd-zenbleed-fix-tested), pero que algunas otras cargas no sufren pérdidas de rendimiento.

>[!WARNING]
> Cabe resaltar que ni deshabilitando SMT (Multithreading Simultáneo) puedes mitigar la vulnerabilidad.

- [**Lectura para una mayor comprensión sobre Zenbleed**](https://www.xda-developers.com/zenbleed/)

- **AMD** confirmó que es un medio viable hacer el parche mediante software para solucionar el problema. 

    - Estos parches saldrán en Noviembre o Diciembre de este año.

- Se está tardando ya que se supone que es una creación mediante un parche de microcódigo que es mucho más eficiente ya que es implementado por firmware.

## **¿Cuáles son las CPUs afectadas?**

| Procesadores AMD |
| --- |
| AMD Ryzen 3000 |
| AMD Ryzen 4000 |
| AMD Ryzen 5000 |
| AMD Ryzen 7020 |
| AMD Ryzen Pro 3000 |
| AMD Ryzen Pro 4000 |
| Procesadores de centro de datos AMD EPYC "Roma" |

**Más información:**

- [**Zenbleed: new hardware vulnerability in AMD CPUs.**](https://www.kaspersky.com/blog/zenbleed-vulnerability/48836/)

- [**AMD ‘Zenbleed’ exploit can leak passwords and encryption keys from Ryzen CPUs.**](https://www.theverge.com/2023/7/25/23806705/amd-ryzen-cpu-processor-zenbleed-vulnerability-exploit-bug)

- [**Zenbleed: Everything you need to know about this AMD security bug.**](https://www.xda-developers.com/zenbleed/)

En caso tal de que quieran probarlo en **Windows 11**, deben aplicar esté registro para su uso:

```cpp
Reg.exe add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CI\Config" /v "VulnerableDriverBlocklistEnable" /t REG_DWORD /d "0" /f
```

- Esto es una función de seguridad llamada **Vulnerable Driver Blocklist**. Esta función está diseñada para evitar que los controladores que se sabe que son vulnerables se carguen en el sistema.

## Instrucciones del parche.

> [!WARNING]
> Antes de explicar los pasos a continuación. Debemos saber que se debe replicar cada vez que se inicie el sistema.
## Manual:

**1.** Debemos tener descargado [**RWEverything.**](http://rweverything.com/download/)

**1.1.** Una vez descargado lo ejecutamos.

**1.2.** Presionamos la opción **CPU MSR Register**.

<p align="center">
  <img src="https://i.imgur.com/SDgeKgC.png" />
</p>

**2.** Se abrirá el menú **CPU MSR Register**.

- Presionamos la opción arriba posteriormente de **User**.
<p align="center">
  <img src="https://i.imgur.com/QqnNC02.png" />
</p>

**2.1.** Se abrirá el apartado de **CPU MSR Register User List**.

**2.2.** Podemos seguir el siguiente formato: ``Name = Address``.

**2.3.** En el apartado ``Name`` podemos escribir lo que queramos.

**2.4.** En el apartado ``Address``, debemos escribir la siguiente dirección ``0xC0011029``.

<p align="center">
  <img src="https://i.imgur.com/zsetkIm.png" />
</p>

**3.** Presionamos en la parte inferior izquierda ``Done``.

**3.1.** Estaremos en otro menú ``User`` y tendremos dos nombres de registro: **MTTRR_DEF_TYPE** y la dirección con el nombre que ingresamos anteriormente.

**3.2.** Presionamos en la segunda fila ``CPU1``.

<p align="center">
  <img src="https://i.imgur.com/uRwp6sQ.png" />
</p>

**3.3.** Abrirá un menú llamado ``Edit CPU1 MSR 0xC0011029``. Lo que demos hacer es buscar el BIT ``9``, el valor estará en ``0``.

<p align="center">
  <img src="https://i.imgur.com/efjFpGR.png" />
</p>

**3.4.** Le damos doble clic al número ``0``, y esté se convertirá en ``1`` de color verde.

> [!NOTE]
> Debemos verificar que en el apartado derecho del medio que la opción ``To all CPUs`` esté marcada.

<p align="center">
  <img src="https://i.imgur.com/6JCuyl8.png" />
</p>

**3.5.** Podemos verificar que antes de hacer el cambio tendremos una asignación ``0003000310E08002`` y al cambiar el BIT 9 se cambiará la asignación a ``0003000310E08202``

<p align="center">
  <img src="https://i.imgur.com/p0x7ist.png" />
</p>

**3.6.** Una vez hecho todos los pasos anteriores podemos cerrar el programa.

> [!WARNING]
> He de recordar que deben hacer esto cada vez que se inicie el sistema, como se mencionó anteriormente.

**Si quieren que se establezca cada vez que se reinicie el PC:**

- Este es un código de batch para que en el momento que se reinicie el sistema pueda volver a ``0003000310E08202`` y no repetir todo el proceso. 

>[!NOTE]
> Como condición se deben tener los pasos [**2.3**] y [**2.4**] hechos, de lo contrario no funcionará el script.

**Instrucciones:**

- Crear un archivo .bat llamado **rw**.

- Copiar y pegar el siguiente código.

```cpp
@echo off
set "RW_PATH=ruta de RW.exe"
set REGISTER_NUMBER=0xC0011029
set EDX_VAL=0x0
set EAX_VAL=0x300310E08002

for /f "tokens=2 delims==" %%a in ('wmic cpu get NumberOfLogicalProcessors /value') do set cores=%%a

for /L %%i in (1,1,%cores%) do (
    echo puto2 %%i
    "%RW_PATH%" /Min /NoLogo /Stdout /Command="RDMSR %REGISTER_NUMBER%"
    "%RW_PATH%" /Min /NoLogo /Stdout /Command="WRMSR %REGISTER_NUMBER% %EDX_VAL% %EAX_VAL%"
)

start "" "%~dp0\rw.bat"
```
- En el lugar de ``set "RW_PATH=ruta de RW.exe"``, poner la carpeta en donde se encuentra **RW.exe**. Normalemnte es aquí: ``C:\Program Files\RW-Everything\Rw.exe``.

- Guardar el cambio hecho en el archivo .bat, asignarle el nombre **rw**, pero con la extensión al final .bat.

- ``Win + R`` -> shell:startup.

- En esa ruta mover el .bat.

- Esto lo que hará es que cada vez que se inicié el PC, ejecutará el .bat para modificar el acceso a las CPUs ``0003000310E08002`` a ``0003000310E08202``. Y con eso tendríamos el parche contra la vulnerabilidad Zenbleed por software.

## **¿Afecta al rendimiento?**

Dentro de lo descrito con anterioridad por [**Paul Alcorn**](https://www.tomshardware.com/news/amd-zenbleed-fix-tested), el rendimiento en juegos no deberías verse afectado de forma tan "**negativa**" por lo menos una mínima diferencia o nula. Pero, a lo que a mí respecta he tenido una regresión en rendimiento y un aumento de la latencia en memoria.

- Esta es una prueba con [**MicrobenchmarkingGUI**](https://github.com/clamchowder/MicrobenchmarksGui) se puede observar que aumentó un poco más la latencia de la memoria, pero esto puede ser una margen de error.

<p align="center">
  <img src="https://i.imgur.com/5wZcdci.png" />
</p>

- Esto es una prueba de **PC Latency**, se está usando **PresentMon 1.9.0**.

```msInputLatency = msBetweenPresents + msUntilDisplayed - previous(msInPresentAPI)```

<p align="center">
  <img src="https://i.imgur.com/WJL7dei.png" />
</p>

- Esto es una prueba de rendimiento en **CSGO** -> 50 (.csv).

  - **Se observan:** Min, 1%ile, 1%low, 0.1%low y STDEV.

<p align="center">
  <img src="https://i.imgur.com/uWgPSMu.png" />
</p>

- Pero en otros casos puede ser lo contrario, gracias a [**Perfomatters**](https://twitter.com/perfomatters) por el banchmark.

<p align="center">
  <img src="https://i.imgur.com/qdTo9OW.png" />
</p>

**Hardware Specs:**

- Ryzen 5 3600 @ 4.2 GHZ.

- XPG D41 3200 Mbps @ 3600 Mpbs (16-21-17-21-50) - (tCL-tRCDRD-tRP-tRAS-tRC).

- ASUS TUF GAMING B450M-PLUS II | AGESA ComboV2PI 1.2.0.A.

- MSI GTX 1050 Ti OC/AERO ITX.

- KernelOS11 v1.4 (22621.2215).
