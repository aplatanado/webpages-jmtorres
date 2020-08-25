---
title: "Qt Network"
author: "Jesús Torres"
date: 2013-03-04T00:00:00.000Z

description: ""

subtitle: ""

image: "/posts/2013-03-04_qt-network/images/1.jpg" 
images:
 - "/posts/2013-03-04_qt-network/images/1.jpg" 


aliases:
    - "/qt-network-b405a20edd4"
---

[Qt](https://jmtorres.webs.ull.es/me/2013/01/proyecto-qt-framework-de-desarrollo-de-aplicaciones/) proporciona un conjunto de API de comunicaciones a través del módulo [Qt Network](http://qt-project.org/doc/qt-5.0/qtnetwork/).
Este ofrece tanto clases de bajo nivel para comunicación mediante protocolos de transporte, como TCP y UDP, como clases de alto nivel que implementan operaciones usando los protocolos de nivel de aplicación más communes, como HTTP o FTP.
[Qt](https://jmtorres.webs.ull.es/me/2013/01/proyecto-qt-framework-de-desarrollo-de-aplicaciones/) incorpora este módulo por dos motivos fundamentales:

1.  Aunque la mayor parte de los sistemas operativos modernos proporcionan un API de acceso a red basado en [BSD sockets](https://jmtorres.webs.ull.es/me/2013/03/bsd-sockets/), algunos introducen funcionalidades adicionales para adaptarlo al modelo de programación preferente en el sistema en cuestión.
Este, por ejemplo, es el caso de Windows y la librería [Winsock](https://jmtorres.webs.ull.es/me/2013/03/bsd-sockets/) que incorpora.
Por tanto [Qt](https://jmtorres.webs.ull.es/me/2013/01/proyecto-qt-framework-de-desarrollo-de-aplicaciones/), gracias a su módulo [Qt Network](http://qt-project.org/doc/qt-5.0/qtnetwork/), nos proporciona una forma portable de acceder a los servicios de red de los sistemas operativos soportados sin tener que preocuparnos por las particularidades de cada uno.
2.  Las operaciones de red suelen requerir mucho tiempo antes de completarse, por lo que usarlas en un _slot_ del hilo principal implica el bloqueo del bucle de mensajes de la aplicación.
Por fortuna el API [BSD sockets](https://jmtorres.webs.ull.es/me/2013/03/bsd-sockets/) permite tanto el uso síncrono como asíncrono de la interfaz.
Esta última forma de utilizarla evita el bloqueo del bucle de mensajes al impedir que las llamadas a la interfaz se bloqueen hasta que las operaciones de red son completadas.
Sin embargo implican una forma de programar mucho más compleja para el desarrollador.



{{< figure src="/posts/2013-03-04_qt-network/images/1.jpg" caption="Cable Ethernet --- [Raysonho @ Open Grid Scheduler / Grid Engine](https://commons.wikimedia.org/wiki/User:Raysonho)" >}}


El API de bajo nivel de [Qt Network](http://qt-project.org/doc/qt-5.0/qtnetwork/) es asíncrono por diseño, ocultando toda esa complejidad detrás del bucle de mensajes y del mecanismo de señales y _slots_ común a todo el _framework_ [Qt](https://jmtorres.webs.ull.es/me/2013/01/proyecto-qt-framework-de-desarrollo-de-aplicaciones/).
Aunque el API de bajo nivel ofrece métodos para comunicación síncrona --- los métodos `waitFor*`, por ejemplo--- en cuyo caso no necesitamos bucle de mensajes, su uso no está recomendado si no es en un hilo diferente al hilo principal.
Además las clases del API de alto nivel no ofrecen métodos síncronos, por lo que si se usan es obligatorio disponer de un bucle de mensajes.
Debemos tener en cuenta que las API del módulo [Qt Network](http://qt-project.org/doc/qt-5.0/qtnetwork/) se pueden usar desde hilos diferentes al hilo principal ya que cada hilo gestionado por la clase [QThread](https://jmtorres.webs.ull.es/me/2013/02/hilos-de-trabajo-usando-senales-y-slots-en-qt/) tiene su propio bucle de mensajes, que se inicia automáticamente en el método `QThread::run()` por defecto.

## Funcionalidad

El módulo [Qt Network](http://qt-project.org/doc/qt-5.0/qtnetwork/) ofrece las siguientes interfaces:

### Clases QNetwork*


`QNetworkRequest`, `QNetworkReply` y `QNetworkAccessManager` son una colección de clases que proporcionan una abstracción de alto nivel sobre operaciones y protocolos de comunicación comunes, por ejemplo HTTP y FTP.
Cada instancia de la clase `QNetworkRequest` representa una petición u operación contra un servicio en red, mientras que las instancias de la clase `QNetworkReply` representan la respuesta a esas peticiones.
La coordinación de toda esta actividad es responsabilidad de `QNetworkAccessManager`, que se encarga de entregar a la red las peticiones una vez han sido creadas y de emitir señales con la respuesta a las mismas cuando son recibidas.
También coordina el uso de _cookies_, peticiones de autenticación y el uso de servidores _proxy_.

### QTcpSocket y QTcpServer

TCP es el protocolo de comunicación utilizado por diversos protocolos de Internet, como HTTP o FTP.
`QTcpSocket` proporciona acceso de bajo nivel a dicho protocolo.
Permite establecer una conexión con una dirección IP y puerto determinados, así como enviar y recibir datos.
Estas operaciones son asíncronas, por lo que no bloquean la ejecución del hilo, siendo la clase la que notifica mediante señales tanto las condiciones de error como de cuándo se dispone de nuevos datos para leer.
Si lo que se desea es crear una aplicación que reciba conexiones TCP entrantes (como un servidor), lo conveniente es utilizar la clase `QTcpServer`.
Ésta nos permite escuchar en una dirección IP y puerto concretos y aceptar las conexiones entrantes por parte de los clientes.
Cada vez que se acepta el intento de conexión de un cliente, se obtiene una instancia de la clase `QTcpSocket` con la que podemos comunicarnos con él.

### QUdpSocket

UDP es un protocolo de comunicación que también es utilizado por diversos protocolos de Internet.
`QUdpSocket` proporciona acceso de bajo nivel a dicho protocolo.
Permite enviar y recibir paquetes de datos a una dirección IP y puerto concretos, ya que a diferencia de TCP, UDP no provee un flujo continuo de datos sino que opera mediante el envío de paquetes.
Tampoco se provee una clase específica para escuchar por conexiones entrantes ---al estilo de `QTcpServer`---, ya que UDP no es un protocolo orientado a conexión.
La interfaz de `QUdpSocket` simplemente permite preparar el _socket_ para aceptar la recepción de paquetes.

### QLocalSocket y QLocalServer

Proporcionan una abstracción similar a la de `QTcpSocket` y `QTcpServer` pero para _sockets_ locales.
En Windows se implementa haciendo uso de tuberías con nombre, mientras que en UNIX, Linux y otro sistemas POSIX se utilizan [sockets de dominio UNIX](http://es.wikipedia.org/wiki/Socket_Unix).

### QHostInfo

Se utiliza para obtener la dirección IP asignada a un nombre de máquina concreto a través del servicio DNS (Domain Name Service).
Tanto la clase `QTcpSocket` como `QUdpSocket` hacen esto automáticamente cuando se indica un nombre de máquina y no una dirección IP.
Sin embargo la clase `QHostInfo` nos permite hacerlo manualmente, si por cualquier motivo fuera necesario.

### QNetworkProxy

Cada instancia de `QNetworkProxy` se usa para describir y configurar la conexión a un servidor _proxy_, que pueden dirigir o filtrar el tráfico entre ambos extremos de una conexión.
Un servidor _proxy_ puede ser activado para un socket concreto, a través del método `QAbstractSocket::setProxy()`, antes de conectarlo, o a nivel de toda la aplicación a través de la función `QNetworkProxy::setApplicationProxy()`.

### QNetworkConfigurationManager y QNetworkConfiguration

Estas clases constituyen la interfaz del gestor de portabilidad, que controla el estado de conectividad del dispositivo, permitiendo iniciar y detener las interfaces de red así como migrar transparentemente entre puntos de acceso.
La clase `QNetworkConfigurationManager` gestiona la lista de configuraciones de red conocidas por el dispositivo. Una configuración de red describe el conjunto de parámetros usados al iniciar la interfaz de red y es representada por instancias de la clase `QNetworkConfiguration`.

## Utilizar Qt Network

Para incorpora el módulo en nuestra aplicación, sólo tenemos que incluir la declaración de sus clases añadiendo la siguiente directiva en aquellos archivos donde las vayamos a utilizar:
``#include <QtNetwork>``

Mientras que para enlazar el módulo con el ejecutable de la aplicación, sólo hay que añadir la siguiente línea al archivo `.pro` del proyecto:
``QT += network``

Hecho esto, la mejor forma de aprender a utilizar los recursos del módulo [Qt Network](http://qt-project.org/doc/qt-5.0/qtnetwork/) es siguiendo los ejemplos disponibles en su documentación:

### Ejemplos de [QTcpSocket](http://qt-project.org/doc/qt-5.0/qtnetwork/qtcpsocket.html) y [QTcpServer](http://qt-project.org/doc/qt-5.0/qtnetwork/qtcpserver.html):

*   [Fortune Client](http://doc.qt.io/qt-5/qtnetwork-fortuneclient-example.html) y [Fortune Server](http://doc.qt.io/qt-5/qtnetwork-fortuneserver-example.html) muestran como desarrollar una aplicación cliente-servidor sobre TCP.
*   [Blocking Fortune Client](http://doc.qt.io/qt-5/qtnetwork-blockingfortuneclient-example.html) muestra como usar la interfaz síncrona de [QTcpSocket](http://qt-project.org/doc/qt-5.0/qtnetwork/qtcpsocket.html).
*   [Threaded Fortune Server](http://doc.qt.io/qt-5/qtnetwork-threadedfortuneserver-example.html) muestra como desarrollar un servidor multihilo donde cada hilo sirve a un cliente.

### Ejemplos de [QUdpSocket](http://qt-project.org/doc/qt-5.0/qtnetwork/qudpsocket.html):

*   [Broadcast Sender](http://doc.qt.io/qt-5/qtnetwork-broadcastsender-example.html) y [Broadcast Receiver](http://doc.qt.io/qt-5/qtnetwork-broadcastreceiver-example.html) muestran como desarrollar un emisor y un receptor sobre UDP.
*   [Multicast Sender](http://doc.qt.io/qt-5/qtnetwork-multicastsender-example.html) y [Multicast Receiver](http://doc.qt.io/qt-5/qtnetwork-multicastreceiver-example.html) muestran como desarrollar aplicaciones que hagan uso de la [multidifusión](http://es.wikipedia.org/wiki/Multidifusi%C3%B3n) (o _multicast_).

## Referencias

1.  [Qt Network](http://qt-project.org/doc/qt-5.0/qtnetwork/)
