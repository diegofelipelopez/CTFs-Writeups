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

<img width="959" height="236" alt="Pasted image 20260404194631" src="https://github.com/user-attachments/assets/4ab5850b-7f29-4c50-beaa-48c633def6d3" />

### 2. Búsqueda de vectores de ataque.
Realizamos un escaneo con nmap dirigido al puerto 22.
```bash
sudo nmap -p22 --script=vuln 172.17.0.2
```
No se encuentra ninguna vulnerabilidad inicial con nmap.

<img width="583" height="194" alt="Pasted image 20260404204040" src="https://github.com/user-attachments/assets/34d6a28d-d7ce-40d1-b042-e7726f5d672a" />

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

<img width="759" height="293" alt="Pasted image 20260404234006" src="https://github.com/user-attachments/assets/7b36a442-6a23-400b-a48b-16a1786130aa" />

* Ya con esa pista nos vamos directamente a Hydra.

Ejecutamos el siguiente comando:
```bash
hydra -l lovely -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4 -vV
```
Bingo! hemos encontrado la contraseña del usuario "lovely".

<img width="950" height="440" alt="Pasted image 20260404235851" src="https://github.com/user-attachments/assets/44e0d5fb-3db4-4173-8c5b-bddfb583654a" />

Nos conectamos por SSH y vemos que no tiene un vector de ataque inicial claro.

<img width="580" height="586" alt="Pasted image 20260405003109" src="https://github.com/user-attachments/assets/ffc0354b-3c29-4bee-9365-966cc8e2aad4" />

Probamos Hydra con el usuario "root" 
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4 -vV
```
Vemos que nos encuentra la contraseña bingo!!!

<img width="793" height="89" alt="Pasted image 20260405003403" src="https://github.com/user-attachments/assets/11634305-137e-4ff6-a387-ab9dec308147" />

Probamos esas credenciales y sucede la magia:

<img width="621" height="264" alt="Pasted image 20260405003634" src="https://github.com/user-attachments/assets/1daca37a-2210-4264-9f0c-d8182e83b1be" />

---

## 🧠 Lecciones Aprendidas & Blue Team

**Concepto Nuevo:** 
* Explotación de la vulnerabilidad **CVE-2018-15473** en OpenSSH < 7.7 utilizando el módulo auxiliar `ssh_enumusers` de Metasploit para descubrir usuarios válidos en el sistema sin poseer credenciales previas.
* Verificación de que comprometer a un usuario de bajos privilegios ("lovely") puede ser un distractor cuando la cuenta de máxima autoridad (`root`) carece de políticas de contraseñas robustas y es susceptible a fuerza bruta directa con listas conocidas (RockYou).

**Cómo Parcharlo (Fix):** 1. **Actualización de Software:** Actualizar el servicio OpenSSH a una versión superior a la 7.7 para mitigar la vulnerabilidad de enumeración de usuarios (CVE-2018-15473).
2. **Restricción de Privilegios por Red:** Modificar el archivo de configuración `/etc/ssh/sshd_config` y establecer el parámetro `PermitRootLogin no`. La cuenta root jamás debe tener acceso directo de inicio de sesión a través del protocolo SSH.
3. **Autenticación Fuerte:** En el mismo archivo de configuración, establecer `PasswordAuthentication no` para forzar el uso exclusivo de pares de llaves criptográficas (SSH Keys), lo que anula por completo la efectividad de los ataques de fuerza bruta o diccionario realizados con Hydra.
4. **Respuesta Activa (IPS):** Implementar herramientas como Fail2Ban para monitorear los logs del sistema (`/var/log/auth.log`) y bloquear automáticamente a nivel de firewall (iptables/nftables) las direcciones IP que generen múltiples intentos de inicio de sesión fallidos.
