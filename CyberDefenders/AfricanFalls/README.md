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

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260227211355.png]*

Usamos la herramienta FTK Imager para analizar el archivo .AD1 y abrimos el txt para ver que encontramos dentro.

Encontramos unos directorios en donde podremos investigar que estaba haciendo el presunto sospechoso.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260227212902.png]*

Y encontramos los esto dentro del reporte de adquisición:

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260227213433.png]*

si bajamos un poco en el txt, encontramos el MD5 de la partición a analizar. Y con eso la respuesta a la primera bandera.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260227213607.png]*

> **Respuesta:** `9471e69c95d8909ae60ddff30d50ffa1`

### 2. Verificación del historial de navegación.
Dentro de la adquision en FTK Imager, nos dirigimos a la siguiente ruta para obtener el archivo con el historial.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260228215533.png]*

Descargamos el archivo para analizarlo con DB Browser:

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260228220112.png]*

Realizamos la conversión de la marca de tiempo 2021-04-29 18:17:38 a WebKit con Dcode:

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260228220717.png]*

Nos genera el siguiente web kit: 13264193858, con esto filtramos dentro de DB Browser. Y encontramos la respuesta de la segunda bandera.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260228220849.png]*

> **Respuesta:** `password cracking lists`

### 3. Verificación de conexiones por FTP
Buscamos dentro de la ruta del usuario posibles programas para realizar algún tipo de conexión.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260228222637.png]*

Dentro del contenido del archivo filezilla.xml encontramos la dirección ip a la que se conecto el sospechoso, y con ello la respuesta a la tercera bandera.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260228222958.png]*

> **Respuesta:** `192.168.1.20`

### 4. Validamos en que fecha y hora se elimino una lista de contraseñas
Después de buscar por diferentes directorios del usuario encontramos una wordlist de contraseñas en la papelera.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301154157.png]*

Exportamos el archivo con los metadatos, en este caso el que inicia con una I en su nombre y usamos la herramienta Windows File Analyzer para ver detalles de su fecha de eliminación, y con esto la respuesta a la cuarta bandera.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301154422.png]*

> **Respuesta:** `2021-04-29 18:22`

### 5. Numero de ejecuciones del navegador Tor
Desde FTK Imager observamos que solo aparece el instalador de Tor, mas no se encuentra ningún rastro de ejecución, de Tor o Firefox, lo que nos quiere decir que el sospechoso, instalo el navegador mas no lo utilizo. Por lo tanto la respuesta seria:     

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301174803.png]*

> **Respuesta:** `0`

### 6. Verificación de la dirección de correo electrónico del sospechoso
Desde FTK Imager nos dirigimos a los archivos del navegador que normalmente almacenan inicios de sesión o los autocompletados, y los abrimos con DB Browser.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301193111.png]*

Observamos lo siguiente del análisis de esos archivos:

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301193608.png]*

Nos sale el el proveedor de correos electrónicos que utiliza el sospechoso, y también 2 posibles usuarios, probamos y funciona con el segundo nombre de usuario, sin embargo no tenemos evidencia solida al respecto.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301195323.png]*

Con BrowsingHistoryView observamos claramente la dirección de correo electrónico del sospechoso y con ello la respuesta de la sexta bandera.

> **Respuesta:** `dreammaker82@protonmail.com`

### 7. Verificamos cual fue el dominio al que el sospechoso le realizo un escaneo
Dentro de FTK Imager nos dirigimos al apartado en donde powershell almacena el historial de comandos utilizados.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301200759.png]*

Dentro del historial de comandos utilizados encontramos un escaneo hacia un dominio, y con ello encontramos la respuesta a la séptima bandera.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301200947.png]*

> **Respuesta:** `dfir.science`

### 8. Ubicación de donde se tomo la fotografía "20210429_152043.jpg" 
Nos dirigimos a la ruta común en donde se almacenan las imágenes y encontramos la imagen solicitada, la descargamos y utilizamos la herramienta para acceder a sus metadatos.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301201612.png]*

Utilizamos por medio de wsl.exe la herramienta de exiftool y sacamos la latitud y longitud de la ubicación

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301212739.png]*

Luego realizamos una búsqueda en Google con esas coordenadas y nos encuentra el sitio de donde fue tomada la foto. 

> **Respuesta:** `Zambia`

### 9. Investigamos cual es la ubicación principal anterior en donde se encontraba la imagen 20210429_151535.jpg
Para realizar esta búsqueda debemos ir a FTK Imager y ubicar el ShellBag, posteriormente abriremos este archivo con ShellBag Explorer.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301225248.png]*

Aquí podemos observar diferentes dispositivos que fueron conectados al equipo y que se navego en ellos y teniendo en cuenta el nombre de la imagen que estamos buscando podemos relacionar que fue tomada desde un dispositivo móvil, por su tipo de nombre y que del teléfono se arrastro o movió a la carpeta actual.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301225554.png]*

Vemos que hay un dispositivo móvil, y que en su interior hay una carpeta con el nombre de la novena bandera. 

> **Respuesta:** `Camera`

### 10. Descubrimos cual es la contraseña del usuario con hash:
`Anon:1001:aad3b435b51404eeaad3b435b51404ee:3DE1A36F6DDB8E036DFD75E8E20C4AF4:::`

Utilizaremos la herramienta web Hashes.com, después de separar el Hash NTLM, y con esto obtenemos la contraseña de la decima bandera.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301231928.png]*

> **Respuesta:** `AFR1CA!`

### 11. Descubrimos la contraseña del usuario Jhon Doe
Los primero que haremos será ir a FTK Imager, luego exportaremos el archivo SYSTEM Y SAM, con esos archivos podremos obtener el Hash NTLM del usuario de Windows.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260301233814.png]*

Luego usando wsl.exe y con la herramienta Impacket, ejecutamos el siguiente comando para obtener el hash:

```bash
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260302002324.png]*

Ahora usaremos la herramienta web de Hashes.com para obtener esa contraseña de inicio de Windows. y con ello la ultima bandera de este ctf.

*[AQUÍ ARRASTRA TU IMAGEN: Pasted image 20260302002448.png]*

> **Respuesta:** `ctf2021`

---

## 🧠 Lecciones Aprendidas & Blue Team
**Concepto Nuevo:** Herramientas antiguas (como samdump2) fallan al extraer contraseñas modernas de Windows y te engañan dándote un hash falso de "contraseña vacía" (31d6cfe0...). Hay que usar herramientas actualizadas como Impacket. También aprendí que los ShellBags registran qué carpetas abrí (incluso en un celular que ya desconecté), y que PowerShell guarda absolutamente todo lo que tecleo en un archivo de texto plano oculto.

**Cómo Parcharlo (Fix):** * **Hashes:** Usar LAPS de Microsoft para rotar claves locales y un EDR para bloquear el acceso directo a los archivos SAM/SYSTEM.
* **ShellBags:** Monitorear el acceso al archivo UsrClass.dat en el registro.
* **PowerShell:** Activar el registro centralizado (Event ID 4104) para que el historial se envíe a un servidor seguro. Así, si el atacante borra su rastro local, la evidencia sobrevive.
