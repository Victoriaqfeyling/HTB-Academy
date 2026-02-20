# Skills Assessment

Para completar la evaluación de habilidades, responda las preguntas a continuación. Deberá aplicar una variedad de habilidades aprendidas en este módulo, incluyendo:

Usando `whois`
Analizando `robots.txt`
Realizando ataques de fuerza bruta a subdominios
Rastreando y analizando resultados
Demuestre su competencia utilizando estas técnicas eficazmente. Recuerde agregar subdominios a su `/etc/hosts` a medida que los encuentre.

---


- vHosts necesarios para estas preguntas: `inlanefreight.htb`

Agregamos la ip `154.57.164.81` a `/etc/hosts`.

#### ¿Cuál es el ID de IANA del registrador del dominio inlanefreight.com?

<img width="1908" height="963" alt="image" src="https://github.com/user-attachments/assets/a4f50e04-8584-4f0c-b407-b17108c82f3b" />



#### ¿Qué software de servidor HTTP alimenta el sitio inlanefreight.htb en el sistema de destino? Responda con el nombre del software, no con la versión; por ejemplo, Apache.


<img width="931" height="384" alt="image" src="https://github.com/user-attachments/assets/43467930-e5f1-4d2c-add7-da88343c4a4e" />


#### ¿Cuál es la clave API en el directorio de administración oculto que ha descubierto en el sistema de destino?

vHost bruteforcing:
<img width="1907" height="683" alt="image" src="https://github.com/user-attachments/assets/9a4402cb-ff45-4a5c-b84a-c84cd7dc665c" />

Agregamos `web1337.inlanefreight.htb` a `/etc/hosts`.

<img width="1232" height="216" alt="image" src="https://github.com/user-attachments/assets/2750fda6-154f-448b-8bd2-d604699087ac" />

Inmediatamente hacemos las pruebas de reconocimiento de inicio a fin, probamos `robots.txt`:

<img width="1273" height="346" alt="image" src="https://github.com/user-attachments/assets/885b6ef6-3e2e-4340-ad35-1609c3dec460" />

Encontramos el panel administrativo en `/admin_h1dd3n`, luego de probar, nos damos cuenta que hay que agregarle la barra `/` al final:

<img width="1869" height="729" alt="image" src="https://github.com/user-attachments/assets/84c0432c-381e-449e-beec-91bc52f81410" />

Encontramos `e963d863ee0e82ba7080fbf558ca0d3f`


#### Después de rastrear el dominio inlanefreight.htb en el sistema de destino, ¿ cuál es la dirección de correo electrónico que encontró? Responda con el correo electrónico completo, por ejemplo, mail@inlanefreight.htb

Ejecutamos:

```bash
python3 ReconSpider.py http://web1337.inlanefreight.htb:<port>
```

Sin embargo no encuentra nada. Si hacemos la búsqueda sobre `http://inlanefreight.htb:<port>` tampoco.

Volvemos a realizar un vHost bruteforce ahora sobre `http://web1337.inlanefreight.htb:<port>`:

<img width="1869" height="729" alt="image" src="https://github.com/user-attachments/assets/3a83da85-8f10-423b-90f1-4a849dfaa8df" />

Encontramos un vHost `dev`, antes de acceder via navegador, hay que agregarlo al `/etc/hosts`. Accedemos:
<img width="925" height="328" alt="image" src="https://github.com/user-attachments/assets/3fb73af3-95cd-4208-b65e-c45a47746477" />


Ahora sí realizamos crawling y ejecutamos:
```bash
python3 ReconSpider.py http://dev.web1337.inlanefreight.htb:<port>
```

Accedemos al archivo `results.json` y obtenemos el mail:
<img width="1744" height="955" alt="image" src="https://github.com/user-attachments/assets/72ea447f-c502-47bc-8cba-0f2d5a262228" />


#### ¿Cuál es la clave API que los desarrolladores de inlanefreight.htb cambiarán?


En el mismo archivo, si vemos al final en la sección de comentarios, encontraremos la clave api:
<img width="1744" height="955" alt="image" src="https://github.com/user-attachments/assets/58b8b227-1075-4093-8c11-69ae5c77920c" />
