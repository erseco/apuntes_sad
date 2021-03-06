Infraestructura de clave pública o PKI
--------------------------------------------------------------------------------

En administración de sistemas suele ser necesario la configuración de mecanismos seguros que impliquen cifrado y confianza. Aunque el proceso que se va a mencionar ahora suele requerir que se pida la intervención de una autoridad de certificación externa en otros casos no va a ser necesario. A continuación se muestra como crear nuestra propia autoridad certificadora que pueda firmar certificados de otros. El objetivo de estos puntos es conseguir lo siguiente:

* Una clave raíz (que estará en un fichero llamado ``ca.key``)
* Un certificado raíz que podremos pasar a todos nuestros clientes (fichero ``ca.crt`` ).
* Una clave privada para un servidor (fichero ``servidor.key``)
* Un certificado emitido para un cierto servicio o servidor (fichero ``servidor.crt``)
* Una clave privada para un servidor (fichero ``cliente.key``)
* Un certificado emitido para un cierto servicio o servidor (fichero ``cliente.crt``)
* Un fichero con números primos precalculados (``Precalculados.pem`` )

Paso 0: instalar Easy-RSA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Usando ``sudo apt-get install easy-rsa`` podremos instalar un software que nos automatizará el proceso de crear una autoridad de certificación con su propio certificado y un certificado de servidor firmador por esa autoridad propia.

Paso 1: Crear una autoridad
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Usando ``make-cadir <directorio>`` podremos crear un directorio con los ficheros de configuración necesarios para crear nuestra propia autoridad de certificacion. Despues entramos en él y editamos el fichero ``vars`` para indicar los siguientes datos:

* País.
* Estado/Provincia.
* Ciudad.
* Organización.
* Email.
* Unidad organizativa.

Paso 2: crear la infraestructura de claves
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Toda autoridad de certificación tiene como mínimo una clave raíz que se necesitará en todos los procesos. Podemos usar el comando ``./easyrsa init-pki`` para construir los ficheros necesarios (pero aún no se generarán las claves).

Paso 3: construir los ficheros de la CA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Con los datos rellenados en el paso 1 y la clave privada del paso 2 se puede crear el certificado raíz de nuestra CA usando ``./easyrsa build-ca`` . Se nos pedirá una clave de acceso para custodiar la clave raíz que se va a generar y se nos pedirá un nombre de usuario o de servidor para incorporar al certificado raíz.

Paso 4: generar un certificado para un servidor
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Usando ``./easyrsa build-server-full <nombre_de_servidor_o_servicio>`` se generarán dos cosas:

* Una clave privada para el servicio (estará en ``pki/private/<nombre>.key`` 
* Un certificado para ese servidor que irá firmado por nuestra CA (estará en ``pki/issued/<nombre>.crt`` 


Paso 5: generar un certificado para un cliente
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Usando ``./easyrsa build-cliente-full <nombre_de_fichero_cliente>`` se generarán otra vez dos cosas:

* Una clave privada para el cliente (estará en ``pki/private/<nombre_fichero_cliente>.key`` 
* Un certificado para ese cliente que irá firmado por nuestra CA (estará en ``pki/issued/<cliente>.crt`` 



Paso 5: precalcular parámetros de claves
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cuando se establezca una conexión se van a utilizar algunos números para cifrar los datos. Estos valores pueden tenerse precalculados en un fichero para acelerar el inicio de las conexiones. Esto puede hacerse con el comando ``openssl dhparam -dsaparam 2048 -out Parametros.pem`` 

Este comando genera números primos aceptables para el establecimiento de una conexión, usando 2048 bits como longitud de clave pero evitando (con el parámetro ``dsaparam``  una serie de números que no aportan más seguridad). 

Paso 6: configurar el servidor y arrancarlos
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

En el servidor podemos crear un fichero como este:

.. code-block:: bash

    proto udp
    port 1194
    dev tun
    server 10.100.0.0 255.255.255.0
    topology subnet
    persist-key
    persist-tun
    keepalive 10 60
    dh    /home/usuario/autoridad/ParametrosDH.pem
    cert  /home/usuario/autoridad/pki/issued/ServidorOpenVPN.crt
    key   /home/usuario/autoridad/pki/private/ServidorOpenVPN.key
    ca    /home/usuario/autoridad/pki/ca.crt

    log-append /var/log/openvpn.log

Y arrancar OpenVPN con ``sudo openvpn --config servidor.conf`` 

Paso 7: configurar el cliente y arrancarlo.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


