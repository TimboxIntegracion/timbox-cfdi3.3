# CFDI 3.3

Documentación para la integración del anexo 20 versión 3.3 (CFDI).

Aviso:

El 1 de julio de 2017 entra en vigor la versión 3.3 de la factura, no obstante podrás continuar emitiendo facturas en la versión 3.2 hasta el 30 de noviembre; a partir del 1 de diciembre, la única versión válida para emitir las facturas será la versión 3.3

[Documentación del SAT para emitir CFDI con la nueva versión 3.3](http://www.sat.gob.mx/informacion_fiscal/factura_electronica/Paginas/Anexo_20_version3.3.aspx)

Ejemplo de comó utilizar el servicio con [SOAP UI](https://www.soapui.org/downloads/soapui.html). Utilizamos la versión open source.

Se deberá hacer uso de la URL que hace referencia al WSDL, en cada petición realizada:

Timbrado:

- [Timbox Pruebas](https://staging.ws.timbox.com.mx/timbrado_cfdi33/wsdl)
- [Timbox Producción](https://sistema.timbox.com.mx/timbrado_cfdi33/wsdl)

Cancelacion:
- [Timbox Pruebas](https://staging.ws.timbox.com.mx/cancelacion/wsdl)
- [Timbox Producción](https://sistema.timbox.com.mx/cancelacion/wsdl)

## Pasos a considerar antes de Timbrar CFDI
Para poder timbrar el CFDI, hay que considerado los siguientes pasos:
   
**<b>Paso 1</b>**
   
   Construir el XML en base al Anexo 20 de acuerdo al estándar definido por el SAT:
  
- [Esquema XSD](http://www.sat.gob.mx/sitio_internet/cfd/3/cfdv33.xsd) 
   
- [Estandar PDF](http://www.sat.gob.mx/informacion_fiscal/factura_electronica/Documents/cfdv33.pdf)
   
**<b>Paso 1.1 Obtener número de certificado</b>**
Si no se cuenta con el número de certificado se puede obtener utilizando un comando de OpenSSL:
  ```
openssl x509 -inform DER -in "CSD01_AAA010101AAA.cer" -noout -serial
  ```
Ejemplo de output:
  ```
serial=3330303031303030303030333030303233373038
  ```
Es importante mencionar que este no es el número de certificado, para obtenerlo se debe eliminar el '3' en cada posición impar de la cadena de 40 caracteres. Ejemplo (hay espacio entre los pares para poder visualizar facilmente):
  ```
33 30 30 30 31 30 30 30 30 30 30 33 30 30 30 32 33 37 30 38
 3  0  0  0  1  0  0  0  0  0  0  3  0  0  0  2  3  7  0  8
30001000000300023708
  ```
En este ejemplo el número de certificado sería: 30001000000300023708

**<b>Paso 2</b>** 
   
   Obtener la cadena Original basandose en el estándar XSLT (Secuencia de cadena Original), para realizar este paso utilizamos la siguiente herramienta  [XSL Test Tool](http://xslttest.appspot.com/) donde subiremos el archivo Estándar de tranformación y el XML.
   
   [Estándar de transformación](http://www.sat.gob.mx/sitio_internet/cfd/3/cadenaoriginal_3_3/cadenaoriginal_3_3.xslt)
 
 Ejemplo de una Cadena Original
 
  ```
||3.3|2017-07-03T12:51:09|01|30001000000300023708|1510.00|MXN|1|1751.60|I|PUE|06300|AAA010101AAA|SENTIENT SA DE CV|601|IAD121214B34|IT & SW Development Solutions de Mexico S de RL de CV|P01|10122100|5|M74|Kilo|Prueba Catalogos Nuevos|250.00|1250.00|1250.00|002|Tasa|0.160000|200.00|24111500|1|KGM|kg|traslucida 90x90 cm. cal. 200|22.00|22.00|22.00|002|Tasa|0.160000|3.52|13101712|10|KGM|KG|POLIETILENO DE BAJA DENSIDAD|23.80|238.00|238.00|002|Tasa|0.160000|38.08|002|Tasa|0.160000|241.60|241.60||
  ```
**<b>Paso 3</b>**
   
Generar sello digital para los CFDIs
Tal como lo específica el anexo 20 en el inciso I Sección B "Generación de sellos digitales para comprobantes fiscales digitales a través de Internet". Para esto usamos la siguiente herramienta, el programa [OPENSSL](https://www.openssl.org/) en el cual utilizamos los siguientes comandos de consola:

1. Obtener el PEM del certificado y el contenido sin los encabezados agregarlo al atributo Certificado en XML.
```
openssl x509 -in 'CSD01_AAA010101AAA.cer' -inform DER -out 'CSD01_AAA010101AAA.cer.pem' -outform PEM
```
2. Obtener el PEM de la llave, con el cual se realizara la firma digital de la cadena original
```
openssl pkcs8 -inform DER -in 'CSD01_AAA010101AAA.key' -passin pass:12345678a -out 'CSD01_AAA010101AAA.key.pem'
```
3. Generación de la digestión o hash.
```
openssl dgst -sha256 -sign 'CSD01_AAA010101AAA.key.pem' -out 'digest.txt' 'cadena_original.txt'
```

4. Creación del archivo PEM de la llave privada.
```
openssl enc -in 'digest.txt' -out 'sello.txt' -base64 -A -K 'CSD01_AAA010101AAA.key.pem' 
```
Ejemplo de la Generación del Sello
```
Vve+KIMdhPjSiPoA+oFPOI1+DHhbIZpAfjHDjdvuDpN9ga4g76DS90JDlY1mwXAOSOwTlA3YUSwFSt23piTUz9fd+e79xhEzLis6Tiarir0EwADu5tHtZezVMzkD4q4hf+qnpFwx9/F8pUd8eU0T6+fvchQyDE8JhTsTAVdKeD7UGjEwr8lbQ0QVVqXf0i3LWLkkrw0IGt4+NKMgp2WcmDmMkcf+fLYBFJmtrb2KQEgG6nc3IG5Bjik2t34BtYrGWfH9FQR9weBitJRMLfq4Lsmv++j9HlehnCdTlHAzEHpUCvSRw8HPQhhMNBg3zYMAWgM9FpPaUuTKFlkjHJbT4w==
```

## Creación del proyecto en SOAP UI
Para iniciar con el ejemplo del timbrado es necesario crear el proyecto con el URL Servicio.

1. El primer paso es crear el proyecto.

    ![](http://i.imgur.com/0ar7zY0.png)
    
2. Lo siguiente es introducir los datos para generar el servicio, en Initial WSDL colocamos el URL que utilizaremos en este caso staging, debemos de asegurarnos de que este seleccionado los siguientes puntos:
    * **Create sample requests for all operations?**
    * **Stores all file paths in project relatively to project file (requires save)**
    
    Después presionaremos el botón de OK

     ![](http://i.imgur.com/fn6qM7N.png)
     
3. El siguiente paso es aceptar el directorio donde se guardará el proyecto.

     ![](http://i.imgur.com/UCq1NwS.png)
     
4. A continuación, nos mostrara el proyecto creado con las peticiones para cada uno de los métodos del servicio.

     ![](http://i.imgur.com/250CyFV.png)
     

## Timbrar CFDI

Para hacer una petición de timbrado de un CFDI, deberá enviar las credenciales asignadas, así como el XML que desea timbrar convertido a una cadena base64, para ello recomendamos utilizar la página [https://www.base64encode.org/](https://www.base64encode.org/) en ella se puede pegar el XML deseado y se obtiene la cadena en base64:

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **timbrar_cfdi**:

     ![](http://i.imgur.com/wxkGZ25.png)
     
Después de dar click nos aparecerá la siguiente ventana, debemos modificar la petición usando nuestros datos, como el siguiente código:
![](http://i.imgur.com/YeoGMB6.png)
```    
 <soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:timbrar_cfdi soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">AAA010101000</username>
         <password xsi:type="xsd:string">h6584D56fVdBbSmmnB</password>
         <sxml xsi:type="xsd:string">PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48Y2ZkaTpDb21wcm9iYW50ZSB4bWxuczp4c2k9Imh0dHA6Ly93d3cudzMub3JnLzIwMDEvWE1MU2NoZW1hLWluc3RhbmNlIiB4bWxuczpjZmRpPSJodHRwOi8vd3d3LnNhdC5nb2IubXgvY2ZkLzMiIFZlcnNpb249IjMuMyIgRmVjaGE9IjIwMTctMDYtMTRUMTE6NTg6NDUiIFNlbGxvPSJOZVh1and1QXBQVTljQnZCaTZkQVNXc1N3RkkwODVJdllINHZmTGlIaCtYeUF1bGgrWXpLQlNIZ2t6MWVKZ3hRSlplbVE2TmxkcEY1b3JZVkFPODBWd1ZDaHcyakh4eFZPV0ZJTUxYT3BTQlNYUkR1ZWtUbmd3anNoNnYzTDlja3RXV01URHhnUVhoN3U0eS9OV09RT1RDUXpuYjZEV3N0WG9laDFNeUhBMmVkemE3dlRucUlSbUdKd3lveFc5dllWK0NqV3FHemViSVpXVURHekc0M09MU1NrWTBVN2RTSG5qZHB6dGwvblBmV0VDdm12emdlaFB5TFN2ZEtSSkFMcUNGWkVIQ3lhRy9FQW83WUhTWjJIS3VMbTRsaXNjSjRmQzlqUjBUV3p6aTNteEpVdGZiempCRUtyS1NXaGxwanNhTG1vZXZGN0hyd1R2RUV3blFCYkE9PSIgRm9ybWFQYWdvPSIwMSIgTm9DZXJ0aWZpY2Fkbz0iMjAwMDEwMDAwMDAzMDAwMjI3NjciIENlcnRpZmljYWRvPSJNSUlGeGpDQ0E2NmdBd0lCQWdJVU1qQXdNREV3TURBd01EQXpNREF3TWpJM05qY3dEUVlKS29aSWh2Y05BUUVMQlFBd2dnRm1NU0F3SGdZRFZRUUREQmRCTGtNdUlESWdaR1VnY0hKMVpXSmhjeWcwTURrMktURXZNQzBHQTFVRUNnd21VMlZ5ZG1samFXOGdaR1VnUVdSdGFXNXBjM1J5WVdOcHc3TnVJRlJ5YVdKMWRHRnlhV0V4T0RBMkJnTlZCQXNNTDBGa2JXbHVhWE4wY21GamFjT3piaUJrWlNCVFpXZDFjbWxrWVdRZ1pHVWdiR0VnU1c1bWIzSnRZV05wdzdOdU1Ta3dKd1lKS29aSWh2Y05BUWtCRmhwaGMybHpibVYwUUhCeWRXVmlZWE11YzJGMExtZHZZaTV0ZURFbU1DUUdBMVVFQ1F3ZFFYWXVJRWhwWkdGc1oyOGdOemNzSUVOdmJDNGdSM1ZsY25KbGNtOHhEakFNQmdOVkJCRU1CVEEyTXpBd01Rc3dDUVlEVlFRR0V3Sk5XREVaTUJjR0ExVUVDQXdRUkdsemRISnBkRzhnUm1Wa1pYSmhiREVTTUJBR0ExVUVCd3dKUTI5NWIyRmp3NkZ1TVJVd0V3WURWUVF0RXd4VFFWUTVOekEzTURGT1RqTXhJVEFmQmdrcWhraUc5dzBCQ1FJTUVsSmxjM0J2Ym5OaFlteGxPaUJCUTBSTlFUQWVGdzB4TmpFd01qRXlNVEUyTXpSYUZ3MHlNREV3TWpFeU1URTJNelJhTUlHeU1Sb3dHQVlEVlFRREV4RlRSVTVVU1VWT1ZDQlRRU0JFUlNCRFZqRWFNQmdHQTFVRUtSTVJVMFZPVkVsRlRsUWdVMEVnUkVVZ1ExWXhHakFZQmdOVkJBb1RFVk5GVGxSSlJVNVVJRk5CSUVSRklFTldNU1V3SXdZRFZRUXRFeHhWVEVNd05URXhNamxIUXpBZ0x5QklSVWRVTnpZeE1EQXpORk15TVI0d0hBWURWUVFGRXhVZ0x5QklSVWRVTnpZeE1EQXpUVVJHVWs1T01Ea3hGVEFUQmdOVkJBc1VERkJ5ZFdWaVlYTmZRMFpFU1RDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTEt5UWxjZjFFb2lQalZBYm5JcW9Pb1c2QzRzendjakRubU4xWU0wNEFrN3NyUEljbjRCTCtpUHVOM3FGRktacnpHYnc0eVpWVWUyRUtUaUhnSy9nTlZiVXdPcGlVR25ZWTY4dERheVBvbExGc2RhdVh2ZExiWm0reVZObkh6bGJnWG4rLzdXRHZNYWU5cjNsNUlUWGRpQW1HdUJiREg2ZllwRTVQSGh4cnlhVzdsb2FWZjNTaTdrT0pjZ1hhRGRpeXVtdm5pUERUN3hWME1FcDB2bmNla3R3amdFalJXRWcvOVJBS3laazcreDBTTGt3ZUhQYzE4bWU4VzFhSGdZZ0tQSmZvZmZTYlFVM1ZUWXJvWnZoWUUwYS8vRkRIZUl6aVZFeXNOMHBqcm5WS0kxRlZ2eVl6bDQ3OFhJeXRTYnBRcUVtbHBTQXFxaE9pU29oUHhuVWFjQ0F3RUFBYU1kTUJzd0RBWURWUjBUQVFIL0JBSXdBREFMQmdOVkhROEVCQU1DQnNBd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dJQkFFOWtkcTdydDRoWDJNTDg1QmkrTXpDUkd6NWNmUFRxcUhDZ2hSdUtRS3lGeEpncUthWDVzT3dua25zVTRlYm1JK1ZuR1dHRDJncWJNVWRVenJzckhGeE5HTmU0K3VUbVhaQW9FOVVGa1JxTHM1TG0wblNpK1FmaGQ5MW01RHhRb1A4TWxpaTMycnFiQ1NVeHloVmFZOFQwS0xnd3d6RkxzYXF5OXZrN3U2TzQ3em1GeU4wQmxXTlFqZ1dQRkRFLzhPWWhiaFlUTERGa01jcmE5OGJVL2JnWFlIM1g3cmZYWWFic3ZmbUx2anJBWkdqS1pSNUovT1ZTQ0RMbGUwa0ZHY1FEMktLcTgrTnQzUkk0cUVoQU9ucExVL29iMlFhZ2JhM3M4aXYrd0NRaXU5aE9Lc0Z2WUpnNkJPZjhQbHFZeG5zUk9TODhpUnlyaWtKYTFvSW9zeTdjekFMZFBiNk9HWDg5Uk5ya1lNWkhWSjdmUEJFbzZ4M09xQ2xRditkTXAzZDVNM2QwVlVhTzVEU1dIY01nb3pob2xaMm5zemRibGViK29rdzAzUkJVM3h3YkhnU2hSODh2UWd3STBUcGxiVGpBUGNpMDYveEZHVzRQY0gxaXV3THNjOGE4K2JGTDBLK2k3OXVYWU9Ia0MwV29oNjVBcFY3MEVKeWN1TWo3SjNyNTkrSXk1Qm1GaCtJd2tTRS9ZdW00V0FvUUxjeHlrcGF2UTFZRUFvVEpieldIaDZlaWNtV0JsNW92ZDZGOFlsQ0tiakQrNlRvNnFkeVQ4MHpNOHRFMXVRRC9FaUdSNDlTeWtSWHFUem9Zb3Q4MjVubEZpb0w1dXd4cFJxYVcwUVMzSDdOd0E4YVZCY2VQUGRYK0dJbm11bXBIWUtvcVMwWS8yNDJ5R2d2TSIgU3ViVG90YWw9IjE1MTAuMDAiIE1vbmVkYT0iTVhOIiBUaXBvQ2FtYmlvPSIxIiBUb3RhbD0iMTc1MS42MCIgVGlwb0RlQ29tcHJvYmFudGU9IkkiIE1ldG9kb1BhZ289IlBVRSIgTHVnYXJFeHBlZGljaW9uPSIwNjMwMCIgeHNpOnNjaGVtYUxvY2F0aW9uPSJodHRwOi8vd3d3LnNhdC5nb2IubXgvY2ZkLzMgaHR0cDovL3d3dy5zYXQuZ29iLm14L3NpdGlvX2ludGVybmV0L2NmZC8zL2NmZHYzMy54c2QgIj48Y2ZkaTpFbWlzb3IgUmZjPSJVTEMwNTExMjlHQzAiIE5vbWJyZT0iU0VOVElFTlQgU0EgREUgQ1YiIFJlZ2ltZW5GaXNjYWw9IjYwMSIvPjxjZmRpOlJlY2VwdG9yIFJmYz0iSUFEMTIxMjE0QjM0IiBOb21icmU9IklUICZhbXA7IFNXIERldmVsb3BtZW50IFNvbHV0aW9ucyBkZSBNZXhpY28gUyBkZSBSTCBkZSBDViIgVXNvQ0ZEST0iUDAxIi8+PGNmZGk6Q29uY2VwdG9zPjxjZmRpOkNvbmNlcHRvIENsYXZlUHJvZFNlcnY9IjEwMTIyMTAwIiBDYW50aWRhZD0iNSIgQ2xhdmVVbmlkYWQ9Ik03NCIgRGVzY3JpcGNpb249IlBydWViYSBDYXRhbG9nb3MgTnVldm9zIiBWYWxvclVuaXRhcmlvPSIyNTAuMDAiIEltcG9ydGU9IjEyNTAuMDAiIFVuaWRhZD0iS2lsbyI+PGNmZGk6SW1wdWVzdG9zPjxjZmRpOlRyYXNsYWRvcz48Y2ZkaTpUcmFzbGFkbyBCYXNlPSIxMjUwLjAwIiBJbXB1ZXN0bz0iMDAyIiBUaXBvRmFjdG9yPSJUYXNhIiBUYXNhT0N1b3RhPSIwLjE2MDAwMCIgSW1wb3J0ZT0iMjAwLjAwIi8+PC9jZmRpOlRyYXNsYWRvcz48L2NmZGk6SW1wdWVzdG9zPjwvY2ZkaTpDb25jZXB0bz48Y2ZkaTpDb25jZXB0byBDbGF2ZVByb2RTZXJ2PSIyNDExMTUwMCIgQ2FudGlkYWQ9IjEiIENsYXZlVW5pZGFkPSJLR00iIERlc2NyaXBjaW9uPSJ0cmFzbHVjaWRhIDkweDkwIGNtLiBjYWwuIDIwMCIgVmFsb3JVbml0YXJpbz0iMjIuMDAiIEltcG9ydGU9IjIyLjAwIiBVbmlkYWQ9ImtnIj48Y2ZkaTpJbXB1ZXN0b3M+PGNmZGk6VHJhc2xhZG9zPjxjZmRpOlRyYXNsYWRvIEJhc2U9IjIyLjAwIiBJbXB1ZXN0bz0iMDAyIiBUaXBvRmFjdG9yPSJUYXNhIiBUYXNhT0N1b3RhPSIwLjE2MDAwMCIgSW1wb3J0ZT0iMy41MiIvPjwvY2ZkaTpUcmFzbGFkb3M+PC9jZmRpOkltcHVlc3Rvcz48L2NmZGk6Q29uY2VwdG8+PGNmZGk6Q29uY2VwdG8gQ2xhdmVQcm9kU2Vydj0iMTMxMDE3MTIiIENhbnRpZGFkPSIxMCIgQ2xhdmVVbmlkYWQ9IktHTSIgRGVzY3JpcGNpb249IlBPTElFVElMRU5PIERFIEJBSkEgREVOU0lEQUQiIFZhbG9yVW5pdGFyaW89IjIzLjgwIiBJbXBvcnRlPSIyMzguMDAiIFVuaWRhZD0iS0ciPjxjZmRpOkltcHVlc3Rvcz48Y2ZkaTpUcmFzbGFkb3M+PGNmZGk6VHJhc2xhZG8gQmFzZT0iMjM4LjAwIiBJbXB1ZXN0bz0iMDAyIiBUaXBvRmFjdG9yPSJUYXNhIiBUYXNhT0N1b3RhPSIwLjE2MDAwMCIgSW1wb3J0ZT0iMzguMDgiLz48L2NmZGk6VHJhc2xhZG9zPjwvY2ZkaTpJbXB1ZXN0b3M+PC9jZmRpOkNvbmNlcHRvPjwvY2ZkaTpDb25jZXB0b3M+PGNmZGk6SW1wdWVzdG9zIFRvdGFsSW1wdWVzdG9zVHJhc2xhZGFkb3M9IjI0MS42MCI+PGNmZGk6VHJhc2xhZG9zPjxjZmRpOlRyYXNsYWRvIEltcHVlc3RvPSIwMDIiIFRpcG9GYWN0b3I9IlRhc2EiIFRhc2FPQ3VvdGE9IjAuMTYwMDAwIiBJbXBvcnRlPSIyNDEuNjAiLz48L2NmZGk6VHJhc2xhZG9zPjwvY2ZkaTpJbXB1ZXN0b3M+PC9jZmRpOkNvbXByb2JhbnRlPg==</sxml>
      </urn:timbrar_cfdi>
   </soapenv:Body>
</soapenv:Envelope>
```
Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) una vez hecho esto nos saldrá el resultado.

  ![](http://i.imgur.com/fwG4Rc2.png)
   
## Cancelar CFDI 
Para la cancelar_cfdi son necesarias las credenciales asignadas, RFC del emisor, un arreglo de nodos folios (el cual debe contener los elementos UUID, RFC del Receptor y Total), el certificado y llave convertidos en PEM (el contenido del archivo)

Crear un cliente para hacer la petición de cancelación al webservice:

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **cancelar_cfdi**:

![](https://i.imgur.com/RVyGwDm.png)

```
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:cancelar_cfdi soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">AAA010101000</username>
         <password xsi:type="xsd:string">h6584D56fVdBbSmmnB</password>
         <rfc_emisor xsi:type="xsd:string">PZA000413788</rfc_emisor>
         <folios xsi:type="urn:folios">
	         <folio>
		         	<uuid xsi:type="xsd:string">6B0D9A22-CE8E-4F00-BFD4-4DD294D6A772</uuid>
		         	<rfc_receptor xsi:type="xsd:string">TME960709LR2</rfc_receptor>
		         	<total xsi:type="xsd:string">5001</total>
	         	</folio>
         </folios>
         <cert_pem xsi:type="xsd:string">-----BEGIN CERTIFICATE-----
MIIGJzCCBA+gAwIBAgIUMjAwMDEwMDAwMDAzMDAwMjI3NjAwDQYJKoZIhvcNAQEL
BQAwggFmMSAwHgYDVQQDDBdBLkMuIDIgZGUgcHJ1ZWJhcyg0MDk2KTEvMC0GA1UE
CgwmU2VydmljaW8gZGUgQWRtaW5pc3RyYWNpw7NuIFRyaWJ1dGFyaWExODA2BgNV
BAsML0FkbWluaXN0cmFjacOzbiBkZSBTZWd1cmlkYWQgZGUgbGEgSW5mb3JtYWNp
w7NuMSkwJwYJKoZIhvcNAQkBFhphc2lzbmV0QHBydWViYXMuc2F0LmdvYi5teDEm
MCQGA1UECQwdQXYuIEhpZGFsZ28gNzcsIENvbC4gR3VlcnJlcm8xDjAMBgNVBBEM
BTA2MzAwMQswCQYDVQQGEwJNWDEZMBcGA1UECAwQRGlzdHJpdG8gRmVkZXJhbDES
MBAGA1UEBwwJQ295b2Fjw6FuMRUwEwYDVQQtEwxTQVQ5NzA3MDFOTjMxITAfBgkq
hkiG9w0BCQIMElJlc3BvbnNhYmxlOiBBQ0RNQTAeFw0xNjEwMjEyMDI1NTNaFw0y
MDEwMjEyMDI1NTNaMIIBEjE6MDgGA1UEAxMxSU5EVVNUUklBUyBQQVJBIEVMIEhP
R0FSIEEgTEEgVkFOR1VBUkRJQSBTQSBERSBDVjE6MDgGA1UEKRMxSU5EVVNUUklB
UyBQQVJBIEVMIEhPR0FSIEEgTEEgVkFOR1VBUkRJQSBTQSBERSBDVjE6MDgGA1UE
ChMxSU5EVVNUUklBUyBQQVJBIEVMIEhPR0FSIEEgTEEgVkFOR1VBUkRJQSBTQSBE
RSBDVjElMCMGA1UELRMcUFpBMDAwNDEzNzg4IC8gSEVHVDc2MTAwMzRTMjEeMBwG
A1UEBRMVIC8gSEVHVDc2MTAwM01ERlJOTjA5MRUwEwYDVQQLFAxQcnVlYmFzX0NG
REkwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCJUeq8lhyXM8Kamegn
YTt2UsISrihy/vhw+2Bt+ULUczpQtZ9rDTeMFXwtKoocAPd31yfqPdlj9dxKbQS5
PmDe3rmpDvcNbGVmfKOMxCWzrWikDVhJVC1++lI6leMnOZHBymbMct2rslUoNq1x
hSG/2R97hI7Cgshge9CvNFyyD6U3h+s7tAggWgGhIqTf/SI6TAwzkp+N6ocu+xdp
6d7PjWP8Jao0E3Gcf79Uy7Mxhxh5Zi7A4e1VQHd392De3ZzSewCCeQUF9fqI7OGZ
OrEsEC66nb5SiyP+qzIQWUCKrMgg6gWzeXfLYzewFLpFK6fxYA1k7C7INQcENb40
Vk9/AgMBAAGjHTAbMAwGA1UdEwEB/wQCMAAwCwYDVR0PBAQDAgbAMA0GCSqGSIb3
DQEBCwUAA4ICAQBz1OZj4/BN+o2BYbb16n3uy5SELXZhJR2uncGLLqz6rddFJsJD
tXjSv0lXs/daTnc8ZcrbT4pVOKGS4ZiQEwvtZbV4J+6wXryOAv03nyKxvKbW1efT
Z2f3ZC0POtr+aBhMcCdmYBpWP37bW+FYQcCXKYszlv0sFKo3WG8PVTHe1+pWeXpv
FY8rjWu6G+u/qkokQMld3ZPuvttSRkpQgTPhjZ7HYBPJOLzFEapURn0TiaK9rFNF
l9xeJntgaYkwMlP6rpp/uUaVtf4aLN4HJ1zpl4OKkMEnZDqaq8tGnSmNtmqvXnGi
3fWn7yR2KNgNVFBH1FZ3KKKj760ctniqaoh1JiB/VZDNKnTyfhulvNtJogR0jLyr
4xjRoTXvd6F4119SNq/0lYaa/qLpwJEv/7yoCvwfH48rqhIt+jPcVxchP9jCFVuD
uIGdhn8mywqqKuZIhgBstzcZFgAnJw5McQW8Pzo+r2Pf4fi3Fq6t9d8SOZ0YAkj+
LJK1CTGhydisk3alVnhjHZI2i4HltgWl8lcl2VyK8Mrtl9pADwgJdA6ccD//d6O/
ECjMNYaFsC3L8XTc9+ck97zQqlDvYPCzQJtXyIUdzvvV0bLpavwkNipb16QbSHkO
hjdkNQmGDnadkZX75k0rcqmRiCr2vHVxkeiUL1Pasdp3dVUvX7ATEqQMZw==
-----END CERTIFICATE-----
</cert_pem>
         <llave_pem xsi:type="xsd:string">-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCJUeq8lhyXM8Ka
megnYTt2UsISrihy/vhw+2Bt+ULUczpQtZ9rDTeMFXwtKoocAPd31yfqPdlj9dxK
bQS5PmDe3rmpDvcNbGVmfKOMxCWzrWikDVhJVC1++lI6leMnOZHBymbMct2rslUo
Nq1xhSG/2R97hI7Cgshge9CvNFyyD6U3h+s7tAggWgGhIqTf/SI6TAwzkp+N6ocu
+xdp6d7PjWP8Jao0E3Gcf79Uy7Mxhxh5Zi7A4e1VQHd392De3ZzSewCCeQUF9fqI
7OGZOrEsEC66nb5SiyP+qzIQWUCKrMgg6gWzeXfLYzewFLpFK6fxYA1k7C7INQcE
Nb40Vk9/AgMBAAECggEAbZFTP053WZ4PNNSBDIrkqzC1cbpMxBT1nxC0jItK68FV
UnjYzs4o+Dlcb511vYp36sNeMeVPxBa0wx3hmv1OxgXpFh++uJM5BWGGDhekDY3b
5KpRO5FTC/IoEl7udKnWx038YD126jzM/d1C30Ve/Hj+ScwnLMS1pWalyGZ7YAcw
ka9gc13RtEwhNU0k/PKVEvOxk77g6hZoE/TgXN6lfLB3BQ9crRwn3NeUKi7vzKOJ
hxdGe898u+Agva0VQPOmB9Lbaw8NuGlNeCa+P7wCpo/LjSpK/Hfp8Tcl/M/S+A1d
fSfitc5PxAWCWHBus/Dix6R01I1Fr+/uPqGAyqqH4QKBgQDsLpJwPaNts52I1zbn
+CYe7L6UcQuqwtDIQ9Pt5eOs2KNBbgWIW7IWrfVIJ0los+LmFDln3uCHgcPRHnJm
aSyDJ3qyNx73VtUz8ZRx0zqs2pbTUyUs9mF/nc/xpM0u7iCJ06q/S+AvyQPpmHCh
0Lch4Gpj4u/eu7uU+hgmzJgVrQKBgQCU17CjVpXlHvw+gTM4yvpFU7yjm4RUNqFr
1rRPZiV56c3wFYGVxTmFG3csCBxu+YrJtvdoQ/JbGR4VaXNXA/kC+YCh9dfYvG3h
vqpvi8BIw/rJN59NCxPaz83/kEZWTpbAZP5rQBieGfZ4G0UMCbG5i6+YHRtM8uRV
tf221YZnWwKBgB9uq0qIyYFGEEcv7Ty+B8TB2TNEQDs/pi2g6UmV+ND+G+wPSmk1
WuQtzqEFqX1nw2C/fExYmyUtnfPsy2jZwnTKAkhJkbN1OPaqxgjIBd0PUldZj28G
cz9ar1wHhM8kHex54RWIcZOqevzRrtu6PUUi6sXUY/wOnA5dom03eV4ZAoGAZAXC
PTGtj4hQCIz4Z/z3TGlmRif3OERyG67wAr9pBdFZxDIfoA8mhU2cuylEOktVuhJL
lnS6w/9QGSGBEgOobhhPGgfEonCWAvMHQ+iNMhkJSfkoAzUjhZLKIyjIK62qXuY/
lsE/Cdf2qmXg86L8HO1C9hzxQLelO/gN5LT/GisCgYEAh41EQHIC8PbXDwbuHmUG
QOAJYC0DkMbQ6EIXrFZ/jTkrat0H5u3KF//6S8SfPMuk68EwDBKWlulSyv6AcfJr
NY6dwreRfoYsuu+Gh+rC7FW+rFiUAjc6Tzfz21oVZ/l/T2HhFcMkCy7sFRtbXcAK
pMwSEnQuw3fiABu38MQJNHw=
-----END PRIVATE KEY-----
</llave_pem>
      </urn:cancelar_cfdi>
   </soapenv:Body>
</soapenv:Envelope>
```

Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) y una vez hecho esto nos saldrá el resultado.

![](https://i.imgur.com/XzRVWlX.png)

### Consultar Estatus
El servicio de “consultar_estatus” se utiliza para la consulta del estatus del CFDI, este servicio pretende proveer una forma alternativa de consulta que requiera verificar el estado de un comprobante en bases de datos del SAT. 

Para utilizar el servicio son necesarias las credenciales asignadas, UUID, RFC del emisor, RFC del receptor y el Total.

Crear un cliente para hacer la petición de cancelación al webservice:

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **consultar_estatus**:

![](https://i.imgur.com/RVyGwDm.png)

```
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:consultar_estatus soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">AAA010101000</username>
         <password xsi:type="xsd:string">h6584D56fVdBbSmmnB</password>
         <uuid xsi:type="xsd:string">6B0D9A22-CE8E-4F00-BFD4-4DD294D6A772</uuid>
         <rfc_emisor xsi:type="xsd:string">PZA000413788</rfc_emisor>
         <rfc_receptor xsi:type="xsd:string">TME960709LR2</rfc_receptor>
         <total xsi:type="xsd:string">5001</total>
      </urn:consultar_estatus>
   </soapenv:Body>
</soapenv:Envelope>
```

Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) y una vez hecho esto nos saldrá el resultado.

![](https://i.imgur.com/0bm5PlD.png)

