# 🎯 Lab: MyBreakSSH

## 📊 Resumen
* **Plataforma:** DockerLabs
* **OS:** Linux
* **Nivel:** Muy Fácil
* **Cadena de Ataque:** Enumeración de usuarios (CVE-2018-15473) ➔ Fuerza Bruta SSH (Hydra) ➔ Acceso Root directo

---

## 📝 Ejecución

### 1. Reconocimiento y enumeración.
Realizamos un escaneo inicial con nmap, para mirar con qué nos encontramos.
```bash
sudo nmap -p- -sS -sC -sV -T5 -n -vvv -Pn 172.17.0.2
```
Nos encuentra el puerto 22 abierto:

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260404194631.png]*

### 2. Búsqueda de vectores de ataque.
Realizamos un escaneo con nmap dirigido al puerto 22.
```bash
sudo nmap -p22 --script=vuln 172.17.0.2
```
No se encuentra ninguna vulnerabilidad inicial con nmap.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260404204040.png]*

* Realizamos una búsqueda de la versión actual de SSH instalada, en busca de vulnerabilidades. En este caso es la OpenSSH 7.7.
* Encontramos que esta versión es propensa a permitir enumeración de usuarios, CVE-2018-15473.

### 3. Explotación de la vulnerabilidad
Abrimos Metasploit, este se encargará de explotar el anterior vector de ataque encontrado.

**Paso a paso dentro de metasploit:**
```bash
msfconsole
use auxiliary/scanner/ssh/ssh_enumusers
set RHOSTS 172.17.0.2
set USER_FILE /usr/share/wordlists/rockyou.txt
set CHECK_FALSE true
run
```
Nos encuentra el siguiente usuario: "lovely"

También probamos con el usuario "root"

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260404234006.png]* Ya con esa pista nos vamos directamente a Hydra.

Ejecutamos el siguiente comando:
```bash
hydra -l lovely -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4 -vV
```
Bingo! hemos encontrado la contraseña del usuario "lovely".

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260404235851.png]*

Nos conectamos por SSH y vemos que no tiene un vector de ataque inicial claro.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260405003109.png]*

Probamos Hydra con el usuario "root" 
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4 -vV
```
Vemos que nos encuentra la contraseña bingo!!!

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260405003403.png]*

Probamos esas credenciales y sucede la magia:

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260405003634.png]*

---

## 🧠 Lecciones Aprendidas & Blue Team

**Concepto Nuevo:** *Explotación de la vulnerabilidad **CVE-2018-15473** en OpenSSH < 7.7 utilizando el módulo auxiliar `ssh_enumusers` de Metasploit para descubrir usuarios válidos en el sistema sin poseer credenciales previas.
* Verificación de que comprometer a un usuario de bajos privilegios ("lovely") puede ser un distractor cuando la cuenta de máxima autoridad (`root`) carece de políticas de contraseñas robustas y es susceptible a fuerza bruta directa con listas conocidas (RockYou).

**Cómo Parcharlo (Fix):** 1. **Actualización de Software:** Actualizar el servicio OpenSSH a una versión superior a la 7.7 para mitigar la vulnerabilidad de enumeración de usuarios (CVE-2018-15473).
2. **Restricción de Privilegios por Red:** Modificar el archivo de configuración `/etc/ssh/sshd_config` y establecer el parámetro `PermitRootLogin no`. La cuenta root jamás debe tener acceso directo de inicio de sesión a través del protocolo SSH.
3. **Autenticación Fuerte:** En el mismo archivo de configuración, establecer `PasswordAuthentication no` para forzar el uso exclusivo de pares de llaves criptográficas (SSH Keys), lo que anula por completo la efectividad de los ataques de fuerza bruta o diccionario realizados con Hydra.
4. **Respuesta Activa (IPS):** Implementar herramientas como Fail2Ban para monitorear los logs del sistema (`/var/log/auth.log`) y bloquear automáticamente a nivel de firewall (iptables/nftables) las direcciones IP que generen múltiples intentos de inicio de sesión fallidos.
