# 🚩 CTF Writeup: Tproot

## 📊 Resumen 
* **Plataforma:** Dockerlabs 
* **OS:** Linux 
* **Nivel:** Muy Fácil
* **Cadena de Ataque:** Escaneo de puertos ➔ Detección de vsftpd 2.3.4 vulnerable ➔ Explotación de backdoor (CVE-2011-2523) �accede como root

---

## 📝 Ejecución y Explotación

- [ ] **1. Reconocimiento y enumeración**
    Realizamos el escaneo inicial con nmap con el comando:
    ```bash
    sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -vvv -Pn 172.17.0.2
    ```
    Nos da como resultado lo siguiente:
<img width="785" height="817" alt="Pasted image 20260318223429" src="https://github.com/user-attachments/assets/45fc418c-927c-4ba9-bc4a-a4cd87b8cf5f" />

    Nos muestra que hay 2 servicios corriendo en donde podemos empezar a buscar cositas: el primero es FTP (puerto 21) y el segundo es HTTP (puerto 80). El segundo nos indica que puede haber alojada una página web.

- [ ] **2. Reconocimiento y escaneo dirigido**
    Ahora vamos a realizar escaneos dirigidos a esos puertos asociados a los servicios activos. Utilizamos el siguiente comando para el puerto 80:
    ```bash
    sudo nmap -p80 --script=vuln 172.17.0.2 
    ```
    El resultado fue:
<img width="581" height="293" alt="Pasted image 20260318225140" src="https://github.com/user-attachments/assets/b00fdb6c-6acd-4574-8417-8640dce65c49" />

    No encontró nada inicialmente por el puerto 80. Vamos con el escaneo al otro puerto:
       
        ```bash
        sudo nmap -p21 --script=vuln 172.17.0.2 
        ```
        
    Vemos que ha encontrado algo interesante:
<img width="948" height="446" alt="Pasted image 20260318230149" src="https://github.com/user-attachments/assets/bf6dde46-9461-4c1e-8bec-a9bdaa96a37c" />

    Tenemos un CVE para investigar y explotar. ¡Vamos a ello!

- [ ] **3. CVE encontrado**
    Investigamos la vulnerabilidad: La versión vsftpd 2.3.4 descargada entre el 30/06/2011 y el 03/07/2011 contiene una puerta trasera que abre una shell en el puerto 6200/tcp.
    
    Encontramos un script en GitHub que podemos utilizar para explotar esta vulnerabilidad. Para utilizarlo debemos hacer lo siguiente:
    
    ```bash
    sudo apt update
    sudo apt install python3
    git clone https://github.com/BolivarJ/CVE-2011-2523.git
    cd CVE-2011-2523
    chmod +x exploit.py
    python3 exploit.py 172.17.0.2
    ```
    
    ¡Ha funcionado, bingo! Ya somos root:
<img width="385" height="171" alt="Pasted image 20260318233442" src="https://github.com/user-attachments/assets/d766320d-6ada-44fb-80b3-4e2193c29e37" />


---

## 🧠 Lecciones Aprendidas & Blue Team

- **Concepto Nuevo:** 
  - Aprendí a identificar versiones de servicios vulnerables mediante scripts de Nmap (`--script=vuln`).
  - Descubrí el CVE-2011-2523 que afecta a vsftpd 2.3.4 y cómo explotarlo para obtener una shell root.

- **Cómo Parcharlo (Fix):** 
  - Actualizar vsftpd a una versión superior a la 2.3.4 (por ejemplo, 2.3.5 o posterior) que no contenga la puerta trasera.
  - Si no es posible actualizar, deshabilitar el servicio FTP o restringir el acceso mediante firewall solo a IPs confiables.
  - Utilizar siempre versiones estables y oficiales del software, evitando descargas de fuentes no verificadas.

