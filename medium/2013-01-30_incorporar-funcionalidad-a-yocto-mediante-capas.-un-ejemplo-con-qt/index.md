---
title: "Incorporar funcionalidad a Yocto mediante capas. Un ejemplo con Qt"
author: "Jesús Torres"
date: 2013-01-30T00:00:00.000Z

description: ""

subtitle: ""

image: "/posts/2013-01-30_incorporar-funcionalidad-a-yocto-mediante-capas.-un-ejemplo-con-qt/images/1.png" 
images:
 - "/posts/2013-01-30_incorporar-funcionalidad-a-yocto-mediante-capas.-un-ejemplo-con-qt/images/1.png" 


aliases:
    - "/incorporar-funcionalidad-a-yocto-mediante-capas-un-ejemplo-con-qt-4e65549cebe5"
---

Una vez hemos [construido nuestra distribución](https://jmtorres.webs.ull.es/me/2013/01/crea-tu-propia-distro-de-linux-con-yocto/) podemos incorporarle nuevas funcionalidades a través de **capas**.

Las capas no son más que grupos de recetas, cada una de las cuales contiene información sobre como construir un componente de software concreto, de manera que trabajando juntos se incorpora la funcionalidad deseada
Para nuestro ejemplo añadiremos a nuestra distribución el _framework_ de desarrollo de aplicaciones [Qt](https://jmtorres.webs.ull.es/me/2013/01/proyecto-qt-framework-de-desarrollo-de-aplicaciones/).




{{< figure src="/posts/2013-01-30_incorporar-funcionalidad-a-yocto-mediante-capas.-un-ejemplo-con-qt/images/1.png" caption="La versión actual del proyecto [Yocto](https://jmtorres.webs.ull.es/me/2013/01/yocto-poky-y-bitbake/) ya provee de la capa **meta-qt** que permite incluir [Qt 4.8 Embedded for Linux](http://doc.qt.io/qt-4.8/qt-embedded-linux.html) en nuestra distribución." >}}
Sin embargo en la versión 5 se actualiza y reemplaza [_Qt_](https://jmtorres.webs.ull.es/me/2013/01/proyecto-qt-framework-de-desarrollo-de-aplicaciones/) _for Embedded Linux_ con un nuevo componente denominado [QPA](http://qt-project.org/wiki/Qt-Platform-Abstraction) o _Qt Platform Abstraction_
La nueva arquitectura es más simple, facilita la integración de [Qt](https://jmtorres.webs.ull.es/me/2013/01/proyecto-qt-framework-de-desarrollo-de-aplicaciones/) con cualquier sistema de ventanas --- suprime el gestor de ventanas QWS incluido en [_Qt_](https://jmtorres.webs.ull.es/me/2013/01/proyecto-qt-framework-de-desarrollo-de-aplicaciones/) _for Embedded Linux_ --- y la inclusión de contenidos basados en [OpenGL](http://es.wikipedia.org/wiki/OpenGL)
Por eso la 5 será la versión que utilizaremos en nuestro proyecto.

En [una charla de Thomas Senyk](http://qt-project.org/videos/watch/qpa-the-qt-platform-abstraction) se desarrollan mucho mejor todas estas diferencias y se justifican las ventajas de QPA respecto a su predecesor.

## Incorporar Qt

Incorporar [Qt](https://jmtorres.webs.ull.es/me/2013/01/proyecto-qt-framework-de-desarrollo-de-aplicaciones/) es muy similar a hacerlo para [el soporte de Raspberry Pi](https://jmtorres.webs.ull.es/me/2013/01/crea-tu-propia-distro-de-linux-con-yocto/) a nuestra distribución.

Empezamos clonado localmente el repositorio **meta-qt5** fuera del directorio `raspberry-pi-build`.

{{< highlight >}}
git clone [https://github.com/meta-qt5/meta-qt5.git](https://github.com/meta-qt5/meta-qt5.git)
{{< / highlight >}}

Después buscamos la variable `BBLAYERS` en `raspberry-pi-build/conf/bblayers.conf` y añadir al final la ruta hasta el repositorio de la capa **meta-qt5** para incluirla en el proceso de construcción
Por ejemplo:

{{< highlight >}}
 BBLAYERS ?= "   
 /home/usuario/poky-danny-8.0/meta   
 /home/usuario/poky-danny-8.0/meta-yocto   
 /home/usuario/poky-danny-8.0/meta-yocto-bsp   
 /home/usuario/meta-raspberry-pi   
 /home/usuario/meta-qt5   
 "
{{< / highlight >}}

Posteriormente construimos la imagen del sistema:

{{< highlight >}}
cd raspberry-pi-build  
bitbake rpi-basic-image
{{< / highlight >}}

Recuerda que esta imagen incluye un servidor **SSH** y un _splash_ de Raspberry Pi durante el arranque
Mientras que la imagen alternativa `rpi-hwup-image` no contiene ninguna de las dos cosas.

Finalmente transferimos la imagen construida a la tarjeta SD:

{{< highlight >}}
sudo dd if=tmp/deploy/images/\  
rpi-basic-image-raspberrypi.rpi-sdimg of=/ruta/a/la/sd
{{< / highlight >}}

y ya podemos probarla en el dispositivo.

## Referencias

1.  [Yocto Project Quick Start](http://www.yoctoproject.org/docs/1.0/yocto-quick-start/yocto-project-qs.html).
