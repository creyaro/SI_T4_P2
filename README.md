
# Práctica 2: Exploración y Análisis de Certificados HTTPS en Linux

Los certificados digitales son piezas esenciales en la comunicación segura en internet. Garantizan que un servidor web es legítimo y que los datos transmitidos están protegidos. En esta práctica vamos a realizar el análisis de un certificado HTTPS en Linux usando OpenSSL, herramienta que ya hemos utilizado en prácticas anteriores.

##  ✅ Entrega y evaluación

Para la entrega deberás generar un documento PDF con capturas de todos los pasos realizados, explicándolo brevemente. Además, debes responder a las preguntas que se plantean a lo largo de la práctica e incluir un informe con la siguiente información:

1. **Dominio Analizado:**  
   _(Por ejemplo: www.google.com)_

2. **Detalles del Certificado:**
   - **Emisor:**  
     _(Ejemplo: Google Trust Services LLC)_
   - **Propietario:**  
     _(Ejemplo: *.google.com)_
   - **Fechas de Validez:**  
     - `Not Before`: _(Ejemplo: Jan  5 08:17:03 2025 GMT)_  
     - `Not After`: _(Ejemplo: Apr  4 08:17:02 2025 GMT)_  
   - **Algoritmo de Firma:**  
     _(Ejemplo: sha256WithRSAEncryption)_
   - **Propósito:**  
     _(Ejemplo: Autenticación del servidor)_

3. **Verificación del Certificado:**
   - Resultado del comando `verify`: _(OK/No OK y por qué)_

4. **Huella Digital:**
   - SHA256: _(Copiar la huella obtenida)_

5. **Conclusión Final:**  
   - ¿El certificado es seguro y confiable?  


## 🖥️ Conexión al servidor HTTPS y obtener su certificado

En primer lugar, escoge la página de la cual extraerás el certificado. Asegúrate de que tiene HTTPS habilitado. Para esta práctica, usaremos Google como ejemplo. Tras ello, sigue los siguientes pasos:

1. En tu terminal, ejecuta el siguiente comando para conectarte al servidor HTTPS de Google:

   ```bash
   openssl s_client -connect www.google.com:443 -showcerts
   ```

> **❗Nota:** Indicamos el puerto 443 porque es el puerto estándar para HTTPS.

2. Este comando realiza varias tareas:
   - Establece una conexión segura con el servidor (www.google.com)
   - Muestra los certificados enviados por el servidor, que suelen incluir el certificado del servidor y, a veces, otros intermedios de la cadena de confianza.

3. Cuando ejecutes este comando, verás una salida extensa. Dentro de ella, encontrarás bloques que comienzan con:

   ```
   -----BEGIN CERTIFICATE-----
   ```

   y terminan con:

   ```
   -----END CERTIFICATE-----
   ```

4. Copia uno de estos bloques (generalmente el primero, correspondiente al certificado principal) y guárdalo en un archivo llamado `google_cert.pem`. Para hacerlo, puedes usar el editor `nano`:

   ```bash
   nano google_cert.pem
   ```

   Pega el contenido del certificado en este archivo, guarda y cierra.


También puedes ejecutar este comando para guardarlo directamente (aunque será todo el contenido)

```bash
openssl s_client -connect www.google.com:443 -showcerts > google_cert.pem
   ```

> **❗Nota:** Este archivo contendrá el certificado en formato PEM, que es el estándar para certificados X.509.

---

### 🔎 Análisis el certificado digital

Ahora que tienes el certificado guardado, lo analizaremos con OpenSSL para entender sus componentes clave.

1. Ejecuta el siguiente comando para analizar el archivo:

   ```bash
   openssl x509 -in google_cert.pem -text -noout
   ```

2. Este comando genera una descripción legible del certificado. A continuación, se explican algunos de los campos importantes que verás en la salida:

   - **Issuer (Emisor):** Indica quién emitió el certificado. Normalmente, es una Autoridad de Certificación (CA) confiable, como "Google Trust Services LLC".
     - **❓Pregunta para reflexionar:** ¿Reconoces al emisor? ¿Es una entidad confiable?

   - **Subject (Propietario):** Describe a quién pertenece el certificado. Por ejemplo, en el caso de Google, dirá algo como:  
     ```
     Subject: C = US, ST = California, L = Mountain View, O = Google LLC, CN = *.google.com
     ``` 
     - `C` (Country): Corresponde al país donde se encuentra la entidad que solicitó el certificado. 
      Ejemplo: C = US → Indica que la entidad está en Estados Unidos.
     - `ST` (State or Province): Corresponde al estado o provincia dentro del país.
      Ejemplo: ST = California → Indica que la entidad está en el estado de California.

     - `L` (Locality): Corresponde a la localidad o ciudad donde se encuentra la entidad.
      Ejemplo: L = Mountain View → Indica que está en la ciudad de Mountain View, California.

     - `O` (Organization): Corresponde al nombre de la organización que solicitó el certificado.
      Ejemplo: O = Google LLC → La organización propietaria del certificado es Google LLC.

     - `CN` (Common Name): Corresponde al dominio principal para el que es válido el certificado.
      Ejemplo: CN = *.google.com → Indica que el certificado es válido para todos los subdominios de google.com, como mail.google.com o drive.google.com.
   
     - **❓Pregunta para reflexionar:** ¿El dominio del certificado coincide con el dominio del sitio web? ¿Por qué es importante?

   - **Validity (Validez):** Muestra las fechas de inicio y expiración del certificado. Por ejemplo:
     ```
     Not Before: Jan  5 08:17:03 2025 GMT
     Not After : Apr  4 08:17:02 2025 GMT
     ```
     - **❓Pregunta para reflexionar:** ¿El certificado está dentro del período válido? ¿Qué implicaciones tendría que estuviera expirado?

   - **Public-Key Info:** Detalla el algoritmo de la clave pública y su tamaño. Por ejemplo:
     ```
     Public-Key: (2048 bit)
     ```
     - Un tamaño de clave de al menos 2048 bits es estándar para garantizar la seguridad.

   - **Signature Algorithm:** Indica el algoritmo utilizado para firmar el certificado, como `sha256WithRSAEncryption`.
     - **❓Pregunta para reflexionar:** ¿Es este algoritmo seguro según los estándares actuales?

   - **Extensions:** Indica los propósitos del certificado, como autenticación de servidores o cifrado.


---

### 🔐 Verificación de la validez del certificado

Una vez que entiendes el contenido del certificado, es importante verificar si es confiable.

1. Usa el siguiente comando para comprobar si el certificado es válido:

   ```bash
   openssl verify -CAfile google_cert.pem google_cert.pem
   ```

   - Este comando compara el certificado con la cadena de confianza definida en tu sistema.  
   - Si el certificado es válido, deberías ver algo como:
     ```
     google_cert.pem: OK
     ```

2. Si el certificado no es válido, OpenSSL indicará el motivo, como problemas con la cadena de confianza o fechas expiradas.

**❓Preguntas para reflexionar:**
   - ¿Qué verificaciones ha superado el certificado?
   - ¿Qué pasaría si no fuera válido?

---

### 🐾 Generación la huella digital del certificado

Las huellas digitales de los certificados son hashes únicos que identifican un certificado. Pueden usarse para verificar su integridad.

1. Genera la huella digital del certificado con el siguiente comando:

   ```bash
   openssl x509 -noout -fingerprint -sha256 -in google_cert.pem
   ```

   Este comando produce la siguiente salida:
     ```
     SHA256 Fingerprint=9A:E9:B6:6D:74:20:87:09:53:03:18:32:43:F9:28:AA:6B:58:35:EF:45:9F:3E:82:84:44:0D:52:59:63:3E
     ```

**❓Preguntas para reflexionar:**
   - ¿Qué garantiza la huella digital?
   - ¿Por qué es importante usar algoritmos seguros como SHA256?

