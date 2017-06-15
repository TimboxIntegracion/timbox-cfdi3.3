# timbox-cfdi3.3
Documentación para la integración del anexo 20 versión 3.3 (CFDI).

Ejemplo de como utilizar el servicio con [SOAP UI](https://www.soapui.org/downloads/soapui.html)

Para este ejemplo utilizamos la versión open source.

Se deberá hacer uso de la URL que hace referencia al WSDL, en cada petición realizada:

- [Timbox Pruebas](https://staging.ws.timbox.com.mx/timbrado_cfdi33/wsdl)

## Creación del proyecto en SOAP UI
Para iniciar con el ejemplo del timbrado es necesario crear el proyecto con el URL Servicio.

1. El primer paso es crear el proyecto.

    ![](http://i.imgur.com/0ar7zY0.png)
    
2. Lo siguiente es introducir los datos para generar el servicio, en initial WSDL ponemos el URL que utilizaremos en este caso staging, debemos de asegurarnos de que este seleccionado los siguientes puntos:
    * **Create sample requests for all operations?**
    * **Stores all file paths in project relatively to project file (requires save)**
