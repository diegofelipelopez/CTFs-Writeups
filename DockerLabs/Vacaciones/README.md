# 🎯 Vacaciones

## 📊 Resumen
**Plataforma:** [Dockerlabs]
**OS:** [Linux]
**Nivel:** [Muy Fácil]
**Cadena de Ataque:** [Information Disclosure (Código Fuente) ➔ SSH Brute Force ➔ Movimiento Lateral (Lectura de txt) ➔ Sudo Misconfiguration (Ruby) ➔ Root]

---

## 📝 Ejecución y Explotación

### 1. Reconocimiento y Enumeración
Realizamos un escaneo inicial sobre la máquina.
```bash
sudo nmap -p- -sS -sC -sV -T5 -n -vvv -Pn 172.17.0.2
```

Vemos que el puerto 22 y el puerto 80 están abiertos, este último nos indica que hay un servidor web corriendo.
<img width="941" height="324" alt="Pasted image 20260419173941" src="https://github.com/user-attachments/assets/a9ef8321-f71c-4fc6-8c05-5d02a4ed3e4b" />


Ahora realizamos un escaneo dirigido a los puertos abiertos encontrados. 
```bash
sudo nmap -p22 --script=vuln 172.17.0.2
sudo nmap -p80 --script=vuln 172.17.0.2
```
Nmap no encuentra inicialmente vulnerabilidades sobre los puertos.
<img width="539" height="431" alt="Pasted image 20260419174724" src="https://github.com/user-attachments/assets/111de9b1-2a23-4fe6-bb70-9aa1afeab0dd" />


### 2. Buscando más vectores vulnerables
Abrimos el navegador y ponemos la url de la máquina.
<img width="959" height="919" alt="Pasted image 20260419175326" src="https://github.com/user-attachments/assets/4037ff63-5f70-423c-8144-2f7086d4370d" />


Nos encontramos con el típico panel de Apache.

Utilizamos gobuster para buscar directorios ocultos.
```bash
gobuster dir -u [http://172.17.0.2/](http://172.17.0.2/) -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,html,php,bak -t 50
```
<img width="942" height="416" alt="Pasted image 20260419191720" src="https://github.com/user-attachments/assets/8af2ee9a-0383-4322-b025-c5e4ebbb3858" />


Seguimos la ruta javascript, que nos lleva hacia 2 archivos js a los que tenemos acceso.
<img width="939" height="680" alt="Pasted image 20260419185634" src="https://github.com/user-attachments/assets/51bad9f6-7a12-40fe-9d12-8d6201e2bb22" />


Y nos muestra la versión de JavaScript y el código.
<img width="942" height="920" alt="Pasted image 20260419185612" src="https://github.com/user-attachments/assets/6d88d0ee-6223-4b97-8e80-41eedb4143fd" />


Revisamos el código de la página en el index.html y encontramos esto:
<img width="959" height="941" alt="Pasted image 20260419191829" src="https://github.com/user-attachments/assets/b0c9bc97-4564-40f4-a4ae-0e71554a8acb" />


Vemos 2 posibles usuarios con los cuales podemos intentar un ataque de fuerza bruta por SSH, con estas pistas encontradas nos vamos a probar:
<img width="793" height="230" alt="Pasted image 20260419195647" src="https://github.com/user-attachments/assets/be2df693-6bef-4e69-8501-92e443e340b6" />


Vemos que el ataque funciona y ya tenemos acceso al servidor por SSH.
<img width="628" height="157" alt="Pasted image 20260419203313" src="https://github.com/user-attachments/assets/41143402-a5ef-46cb-b209-7791693d3097" />


### 3. Escalada de Privilegios

Ya estamos dentro del servidor, ahora empezaremos a buscar opciones de escalada de privilegios. Ejecutamos comandos para ver si hay binarios diferentes, o mirar si tenemos permiso de ejecutar sudo -l
```bash
sudo -l
find / -perm -4000 2>/dev/null
```
<img width="664" height="492" alt="Pasted image 20260419203812" src="https://github.com/user-attachments/assets/ba610507-dc41-464d-8761-43778d361f41" />


Ejecutamos una bash para tener una mejor vista, luego nos vamos atrás en los directorios y revisamos a ver qué podemos encontrar:
<img width="948" height="755" alt="Pasted image 20260419204510" src="https://github.com/user-attachments/assets/cc6803e8-3ad4-4ff1-ab86-0713a9c81f29" />


Recordando el mensaje anterior encontrado en el código fuente de la página, esta contraseña es del usuario juan, salimos y validamos:
<img width="643" height="147" alt="Pasted image 20260419204856" src="https://github.com/user-attachments/assets/c5eaca0b-3bcf-4d58-ac8b-cd300778c224" />


Efectivamente ingresamos al usuario juan, ahora vamos a buscar posibilidades de escalada de privilegios.
Ejecutamos los mismos comandos que antes:
```bash
ls -all
sudo -l
find / -perm -4000 2>/dev/null
```
Vemos que tenemos permisos de ejecución del binario ruby, entonces vamos a probar payload para escalar privilegios, vamos a la página de GTFObins:
<img width="950" height="382" alt="Pasted image 20260419210155" src="https://github.com/user-attachments/assets/0433d3d6-125d-45b8-992f-1defcac4b808" />


### 4. Validación Final

Lanzamos el payload anterior y vemos la magia:
```bash
sudo ruby -e 'exec "/bin/bash"'
```
<img width="453" height="70" alt="Pasted image 20260419210926" src="https://github.com/user-attachments/assets/914411dd-2098-42af-bc2e-7aff86d82aac" />


---

## 🧠 Lecciones Aprendidas & Blue Team

**Concepto Nuevo:** * **Enumeración pasiva:** Extracción de credenciales y vectores de ataque a partir de la revisión exhaustiva de código fuente público (HTML/JS).
* **GTFOBins / Sudo Abuse:** Uso de binarios de programación legítimos (como Ruby) para saltar restricciones y spawnear shells con privilegios elevados cuando existe una mala configuración en `sudoers`.

**Cómo Parcharlo (Fix):** 1. **Fuga de Información (Information Disclosure):** Implementar prácticas de desarrollo seguro. Bajo ninguna circunstancia se deben dejar nombres de usuario, comentarios internos, ni posibles contraseñas en el código fuente del lado del cliente (frontend).
2. **Endurecimiento de SSH (Hardening):** * Deshabilitar la autenticación por contraseña en `/etc/ssh/sshd_config` (`PasswordAuthentication no`) y utilizar autenticación exclusiva mediante claves criptográficas (PKI).
   * Implementar herramientas como **Fail2Ban** o políticas restrictivas en el Firewall/IPS para bloquear direcciones IP tras múltiples intentos fallidos de inicio de sesión, mitigando la fuerza bruta.
3. **Gestión de Permisos Internos:** Realizar auditorías en el sistema de archivos. Evitar almacenar credenciales en archivos `.txt` dentro de los directorios de los usuarios. Establecer permisos estrictos (ej. `chmod 700`) para que un usuario no pueda leer los directorios *home* de otros.
4. **Restricción de Sudo (Privilege Escalation Fix):** Eliminar o modificar la línea en `/etc/sudoers` que permite al usuario ejecutar `ruby` como root (generalmente configurado con la etiqueta `NOPASSWD`). Si por requerimientos de la aplicación el usuario necesita ejecutar un script de Ruby con privilegios, se debe referenciar el path exacto del script en el sudoers (`/usr/bin/ruby /ruta/al/script.rb`) y nunca permitir la ejecución del binario base en solitario.
