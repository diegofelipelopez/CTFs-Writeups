# 🎯 Lab: AfricanFalls

## 📊 Contexto
* **Plataforma:** CyberDefenders 
* **OS:** Linux / Windows 
* **Nivel:** Medio
* **Adquision de memoria no volatil:** Análisis de partición lógica de disco

---

## 📝 Lista de Tareas

### 1. Verificamos todo el contenido disponible en el ctf
Encontramos 2 archivos dentro de la carpeta de descargas.

<img width="690" height="107" alt="Pasted image 20260227211355" src="https://github.com/user-attachments/assets/75e097e8-5f39-43b5-af16-51af7591c3f4" />


Usamos la herramienta FTK Imager para analizar el archivo .AD1 y abrimos el txt para ver que encontramos dentro.

Encontramos unos directorios en donde podremos investigar que estaba haciendo el presunto sospechoso.

<img width="405" height="478" alt="Pasted image 20260227212902" src="https://github.com/user-attachments/assets/1eb313ff-ff1f-4ef3-ab47-2f9ad7548f35" />


Y encontramos los esto dentro del reporte de adquisición:

<img width="1920" height="1040" alt="Pasted image 20260227213433" src="https://github.com/user-attachments/assets/245b61a5-38b6-4773-8ba3-7108329e11cc" />


si bajamos un poco en el txt, encontramos el MD5 de la partición a analizar. Y con eso la respuesta a la primera bandera.

<img width="499" height="69" alt="Pasted image 20260227213607" src="https://github.com/user-attachments/assets/aaddb5eb-32d5-40ca-9827-0c54cf5c2c39" />


> **Respuesta:** `9471e69c95d8909ae60ddff30d50ffa1`

### 2. Verificación del historial de navegación.
Dentro de la adquision en FTK Imager, nos dirigimos a la siguiente ruta para obtener el archivo con el historial.

<img width="1377" height="430" alt="Pasted image 20260228215533" src="https://github.com/user-attachments/assets/f0df1cee-018f-440a-9106-6c98d9bcb0f5" />


Descargamos el archivo para analizarlo con DB Browser:

<img width="1808" height="674" alt="Pasted image 20260228220112" src="https://github.com/user-attachments/assets/7371c47e-55e8-4fff-8a82-7c6247d7eef3" />


Realizamos la conversión de la marca de tiempo 2021-04-29 18:17:38 a WebKit con Dcode:

<img width="1091" height="650" alt="Pasted image 20260228220717" src="https://github.com/user-attachments/assets/b9ed4adb-c314-4fa0-891a-b773cdc666cf" />


Nos genera el siguiente web kit: 13264193858, con esto filtramos dentro de DB Browser. Y encontramos la respuesta de la segunda bandera.

<img width="1807" height="673" alt="Pasted image 20260228220849" src="https://github.com/user-attachments/assets/4fa41353-7801-4099-b421-ea1f2f2c28bd" />


> **Respuesta:** `password cracking lists`

### 3. Verificación de conexiones por FTP
Buscamos dentro de la ruta del usuario posibles programas para realizar algún tipo de conexión.

<img width="1393" height="312" alt="Pasted image 20260228222637" src="https://github.com/user-attachments/assets/0dbcd0bb-aafb-44a6-92cf-68e56f067ddc" />


Dentro del contenido del archivo filezilla.xml encontramos la dirección ip a la que se conecto el sospechoso, y con ello la respuesta a la tercera bandera.

<img width="590" height="626" alt="Pasted image 20260228222958" src="https://github.com/user-attachments/assets/5336b4b1-6bb3-406d-9a8a-f8df0f440e3f" />


> **Respuesta:** `192.168.1.20`

### 4. Validamos en que fecha y hora se elimino una lista de contraseñas
Después de buscar por diferentes directorios del usuario encontramos una wordlist de contraseñas en la papelera.

<img width="1362" height="505" alt="Pasted image 20260301154157" src="https://github.com/user-attachments/assets/725e00d4-9f6b-400d-9c44-f49a7b45e360" />


Exportamos el archivo con los metadatos, en este caso el que inicia con una I en su nombre y usamos la herramienta Windows File Analyzer para ver detalles de su fecha de eliminación, y con esto la respuesta a la cuarta bandera.

<img width="902" height="598" alt="Pasted image 20260301154422" src="https://github.com/user-attachments/assets/79197044-1978-47b1-89e9-e6e28cf52f3f" />


> **Respuesta:** `2021-04-29 18:22`

### 5. Numero de ejecuciones del navegador Tor
Desde FTK Imager observamos que solo aparece el instalador de Tor, mas no se encuentra ningún rastro de ejecución, de Tor o Firefox, lo que nos quiere decir que el sospechoso, instalo el navegador mas no lo utilizo. Por lo tanto la respuesta seria:     

<img width="1492" height="475" alt="Pasted image 20260301174803" src="https://github.com/user-attachments/assets/374ad4d8-ad9a-4f2b-9ebc-b59403fe11bd" />


> **Respuesta:** `0`

### 6. Verificación de la dirección de correo electrónico del sospechoso
Desde FTK Imager nos dirigimos a los archivos del navegador que normalmente almacenan inicios de sesión o los autocompletados, y los abrimos con DB Browser.

<img width="1479" height="645" alt="Pasted image 20260301193111" src="https://github.com/user-attachments/assets/c28f59ae-f920-482c-ad83-00916af85db7" />


Observamos lo siguiente del análisis de esos archivos:

<img width="1812" height="561" alt="Pasted image 20260301193608" src="https://github.com/user-attachments/assets/67f89294-cef9-47b3-be9e-e0c5c7c19261" />


Nos sale el el proveedor de correos electrónicos que utiliza el sospechoso, y también 2 posibles usuarios, probamos y funciona con el segundo nombre de usuario, sin embargo no tenemos evidencia solida al respecto.

<img width="1428" height="753" alt="Pasted image 20260301195323" src="https://github.com/user-attachments/assets/8b81b70d-0d27-49e5-af53-fb2e841ea647" />


Con BrowsingHistoryView observamos claramente la dirección de correo electrónico del sospechoso y con ello la respuesta de la sexta bandera.

> **Respuesta:** `dreammaker82@protonmail.com`

### 7. Verificamos cual fue el dominio al que el sospechoso le realizo un escaneo
Dentro de FTK Imager nos dirigimos al apartado en donde powershell almacena el historial de comandos utilizados.

<img width="1476" height="538" alt="Pasted image 20260301200759" src="https://github.com/user-attachments/assets/83d7d2b0-6681-4731-92c9-712bbecf740e" />


Dentro del historial de comandos utilizados encontramos un escaneo hacia un dominio, y con ello encontramos la respuesta a la séptima bandera.

<img width="245" height="360" alt="Pasted image 20260301200947" src="https://github.com/user-attachments/assets/f888dbbf-9813-44a6-9edd-b233dd1717b0" />


> **Respuesta:** `dfir.science`

### 8. Ubicación de donde se tomo la fotografía "20210429_152043.jpg" 
Nos dirigimos a la ruta común en donde se almacenan las imágenes y encontramos la imagen solicitada, la descargamos y utilizamos la herramienta para acceder a sus metadatos.

<img width="1474" height="424" alt="Pasted image 20260301201612" src="https://github.com/user-attachments/assets/ec2f93ba-bf88-4f2b-b14d-c9e505e838af" />


Utilizamos por medio de wsl.exe la herramienta de exiftool y sacamos la latitud y longitud de la ubicación

<img width="717" height="189" alt="Pasted image 20260301212739" src="https://github.com/user-attachments/assets/fd38fdd7-921b-4ed8-90b7-fa6b54c6c037" />


Luego realizamos una búsqueda en Google con esas coordenadas y nos encuentra el sitio de donde fue tomada la foto. 

> **Respuesta:** `Zambia`

### 9. Investigamos cual es la ubicación principal anterior en donde se encontraba la imagen 20210429_151535.jpg
Para realizar esta búsqueda debemos ir a FTK Imager y ubicar el ShellBag, posteriormente abriremos este archivo con ShellBag Explorer.

<img width="1395" height="432" alt="Pasted image 20260301225248" src="https://github.com/user-attachments/assets/0ef60306-2776-46f4-926c-e7407dd3e292" />


Aquí podemos observar diferentes dispositivos que fueron conectados al equipo y que se navego en ellos y teniendo en cuenta el nombre de la imagen que estamos buscando podemos relacionar que fue tomada desde un dispositivo móvil, por su tipo de nombre y que del teléfono se arrastro o movió a la carpeta actual.

<img width="1205" height="804" alt="Pasted image 20260301225554" src="https://github.com/user-attachments/assets/d721cf78-aaa4-470f-9e3c-5b88c7ef2540" />


Vemos que hay un dispositivo móvil, y que en su interior hay una carpeta con el nombre de la novena bandera. 

> **Respuesta:** `Camera`

### 10. Descubrimos cual es la contraseña del usuario con hash:
`Anon:1001:aad3b435b51404eeaad3b435b51404ee:3DE1A36F6DDB8E036DFD75E8E20C4AF4:::`

Utilizaremos la herramienta web Hashes.com, después de separar el Hash NTLM, y con esto obtenemos la contraseña de la decima bandera.

<img width="1602" height="342" alt="Pasted image 20260301231928" src="https://github.com/user-attachments/assets/2bffba82-02fd-4388-a95e-154db3391f27" />


> **Respuesta:** `AFR1CA!`

### 11. Descubrimos la contraseña del usuario Jhon Doe
Los primero que haremos será ir a FTK Imager, luego exportaremos el archivo SYSTEM Y SAM, con esos archivos podremos obtener el Hash NTLM del usuario de Windows.

<img width="1387" height="586" alt="Pasted image 20260301233814" src="https://github.com/user-attachments/assets/acbd46d9-13ec-4b5e-a7e1-0d46d0ee50b1" />


Luego usando wsl.exe y con la herramienta Impacket, ejecutamos el siguiente comando para obtener el hash:

```bash
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

<img width="752" height="189" alt="Pasted image 20260302002324" src="https://github.com/user-attachments/assets/ed13e280-854a-413f-a8b6-166b639f1816" />


Ahora usaremos la herramienta web de Hashes.com para obtener esa contraseña de inicio de Windows. y con ello la ultima bandera de este ctf.

<img width="704" height="248" alt="Pasted image 20260302002448" src="https://github.com/user-attachments/assets/8fecf194-db41-4e19-ae57-31c8a32c5bf8" />


> **Respuesta:** `ctf2021`

---

## 🧠 Lecciones Aprendidas & Blue Team
**Concepto Nuevo:** Herramientas antiguas (como samdump2) fallan al extraer contraseñas modernas de Windows y te engañan dándote un hash falso de "contraseña vacía" (31d6cfe0...). Hay que usar herramientas actualizadas como Impacket. También aprendí que los ShellBags registran qué carpetas abrí (incluso en un celular que ya desconecté), y que PowerShell guarda absolutamente todo lo que tecleo en un archivo de texto plano oculto.

**Cómo Parcharlo (Fix):** * **Hashes:** Usar LAPS de Microsoft para rotar claves locales y un EDR para bloquear el acceso directo a los archivos SAM/SYSTEM.
* **ShellBags:** Monitorear el acceso al archivo UsrClass.dat en el registro.
* **PowerShell:** Activar el registro centralizado (Event ID 4104) para que el historial se envíe a un servidor seguro. Así, si el atacante borra su rastro local, la evidencia sobrevive.
