# CFDI 3.3

Documentación para la integración del anexo 20 versión 3.3 (CFDI).

Aviso:

El 1 de julio de 2017 entra en vigor la versión 3.3 de la factura, no obstante podrás continuar emitiendo facturas en la versión 3.2 hasta el 30 de noviembre; a partir del 1 de diciembre, la única versión válida para emitir las facturas será la versión 3.3

[Documentación del SAT para emitir CFDI con la nueva versión 3.3](http://www.sat.gob.mx/informacion_fiscal/factura_electronica/Paginas/Anexo_20_version3.3.aspx)

Ejemplo de comó utilizar el servicio con [SOAP UI](https://www.soapui.org/downloads/soapui.html). Utilizamos la versión open source.

Se deberá hacer uso de la URL que hace referencia al WSDL, en cada petición realizada:

- [Timbox Pruebas](https://staging.ws.timbox.com.mx/timbrado_cfdi33/wsdl)

## Pasos a considerar antes de Timbrar CFDI
Para poder timbrar el CFDI, hay que considerado los siguientes pasos:
   
**<b>Paso 1</b>**
   
   Construir el XML en base al Anexo 20 de acuerdo al estándar definido por el SAT:
  
- [Esquema XSD](http://www.sat.gob.mx/sitio_internet/cfd/3/cfdv33.xsd) 
   
- [Estandar PDF](http://www.sat.gob.mx/informacion_fiscal/factura_electronica/Documents/cfdv33.pdf)
   
**<b>Paso 2</b>** 
   
   Obtener la cadena Original basandose en el estándar XSLT (Secuencia de cadena Original), para realizar este paso utilizamos la siguiente herramienta  [XSL tranformer](https://www.freeformatter.com/xsl-transformer.html#ad-output) donde subiremos el archivo Estándar de tranformación y el XML.
   
   [Estándar de transformación](http://www.sat.gob.mx/sitio_internet/cfd/3/cadenaoriginal_3_3/cadenaoriginal_3_3.xslt)
 
 Ejemplo de una Cadena Original
 
  ```
||3.3|2017-07-03T12:51:09|01|30001000000300023708|1510.00|MXN|1|1751.60|I|PUE|06300|AAA010101AAA|SENTIENT SA DE CV|601|IAD121214B34|IT & SW Development Solutions de Mexico S de RL de CV|P01|10122100|5|M74|Kilo|Prueba Catalogos Nuevos|250.00|1250.00|1250.00|002|Tasa|0.160000|200.00|24111500|1|KGM|kg|traslucida 90x90 cm. cal. 200|22.00|22.00|22.00|002|Tasa|0.160000|3.52|13101712|10|KGM|KG|POLIETILENO DE BAJA DENSIDAD|23.80|238.00|238.00|002|Tasa|0.160000|38.08|002|Tasa|0.160000|241.60|241.60||
  ```
**<b>Paso 3</b>**
   
Generar sello digital para los CFDIs
Tal como lo específica el anexo 20 en el inciso I Sección B "Generación de sellos digitales para comprobantes fiscales digitales a través de Internet"

1. Generación de la digestión o hash.
```
openssl dgst -sha256 -sign 'CSD01_AAA010101AAA.key.pem' -out 'digest.txt' 'cadena_original.txt'
```

2. Creación del archivo PEM de la llave privada.
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
  
## Confirmación CFDI
La confirmación es única e irrepetible, Esto es para expedir un comprobante con importes o tipo de cambio fuera del rango establecido o en ambos casos. La confirmación deben registrar valores alfanuméricos de 5 posiciones.

Para la petición de confirmación de timbrado de un CFDI, deberá enviar las credenciales asignadas.

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **confirmacion_cfdi**:
  
  ![](http://i.imgur.com/GubQcAw.png)
  
Despues de dar click nos saldrá la siguiente ventana, debemos modificar la petición usando nuestras credenciales, como el siguiente código:

![](http://i.imgur.com/rO23b9Q.png)

```
 <soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:confirmacion_cfdi soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">AAA010101000</username>
         <password xsi:type="xsd:string">h6584D56fVdBbSmmnB</password>
      </urn:confirmacion_cfdi>
   </soapenv:Body>
</soapenv:Envelope> 
 ```
  
Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) nos saldra el resultado.

![](http://i.imgur.com/7ZPYvy1.png)
 
## Cancelar CFDI
Para la cancelación son necesarias las credenciales asignadas, RFC del emisor, un arreglo de UUIDs, el archivo PFX convertido a cadena en base64 y el password del archivo PFX:

Crear un cliente para hacer la petición de cancelación al webservice:

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **cancelar_cfdi**:

![](http://i.imgur.com/py8K2eL.png)
```
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:cancelar_cfdi soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">AAA010101000</username>
         <password xsi:type="xsd:string">h6584D56fVdBbSmmnB</password>
         <rfcemisor xsi:type="xsd:string">AAA010101AAA</rfcemisor>
         <uuids xsi:type="urn:uuid">
            <!--Zero or more repetitions:-->
            <uuid xsi:type="xsd:string">9C6F27D1-608A-4468-B57A-AF6C293F5728</uuid>
         </uuids>
         <pfxbase64 xsi:type="xsd:string">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</pfxbase64>
         <pfxpassword xsi:type="xsd:string">12345678a</pfxpassword>
      </urn:cancelar_cfdi>
   </soapenv:Body>
</soapenv:Envelope>
```
Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) y una vez hecho esto nos saldrá el resultado.

   ![](http://i.imgur.com/oE7IB3H.png)

## Crear archivo PFX
Para poder crear el archivo PFX es necesario contar con el certificado y la llave privada.

Una vez teniendo esta informacion para la creación del PFX usamos el programa [OPENSSL](https://www.openssl.org/) en el cual utilizamos los siguientes tres comandos de consola que son los siguientes:

***Los certificados de prueba se encuentran en el proyecto en la carpeta certificados o puede descargarlos de [http://www.sat.gob.mx/informacion_fiscal/factura_electronica/Paginas/certificado_sello_digital.aspx](http://www.sat.gob.mx/informacion_fiscal/factura_electronica/Paginas/certificado_sello_digital.aspx)**

1. Creación del archivo PEM del certificado.
```
openssl x509 -in 'CSD01_AAA010101AAA.cer' -inform DER -out 'CSD01_AAA010101AAA.cer.pem' -outform PEM
```

2. Creación del archivo PEM de la llave privada.
```
openssl pkcs8 -inform DER -in 'CSD01_AAA010101AAA.key' -passin pass:12345678a -out 'CSD01_AAA010101AAA.key.pem'
```

3. Creación del archivo PFX.
```
openssl pkcs12 -export -out 'CSD01_AAA010101AAA.pfx' -in 'CSD01_AAA010101AAA.cer.pem' -inkey 'CSD01_AAA010101AAA.key.pem' -password pass:12345678a
```

## Cancelar CFDI CERTS
Para la cancelación_cfdi_certs son necesarias las credenciales asignadas, RFC del emisor, un arreglo de UUIDs, el certificado y llave convertidos en PEM (el contenido del archivo)y por ultimo su contraseña:

Crear un cliente para hacer la petición de cancelación al webservice:

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **cancelar_cfdi_certs**:

![](http://i.imgur.com/eGGRh43.png)

```
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:cancelar_cfdi_certs soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">AAA010101000</username>
         <password xsi:type="xsd:string">h6584D56fVdBbSmmnB</password>
         <rfcemisor xsi:type="xsd:string">AAA010101AAA</rfcemisor>
         <uuids xsi:type="urn:uuid">
            <!--Zero or more repetitions:-->
            <uuid xsi:type="xsd:string">9C6F27D1-608A-4468-B57A-AF6C293F5728</uuid>
         </uuids>
         <cert_pem xsi:type="xsd:string">-----BEGIN CERTIFICATE-----
MIIF+TCCA+GgAwIBAgIUMzAwMDEwMDAwMDAzMDAwMjM3MDgwDQYJKoZIhvcNAQEL
BQAwggFmMSAwHgYDVQQDDBdBLkMuIDIgZGUgcHJ1ZWJhcyg0MDk2KTEvMC0GA1UE
CgwmU2VydmljaW8gZGUgQWRtaW5pc3RyYWNpw7NuIFRyaWJ1dGFyaWExODA2BgNV
BAsML0FkbWluaXN0cmFjacOzbiBkZSBTZWd1cmlkYWQgZGUgbGEgSW5mb3JtYWNp
w7NuMSkwJwYJKoZIhvcNAQkBFhphc2lzbmV0QHBydWViYXMuc2F0LmdvYi5teDEm
MCQGA1UECQwdQXYuIEhpZGFsZ28gNzcsIENvbC4gR3VlcnJlcm8xDjAMBgNVBBEM
BTA2MzAwMQswCQYDVQQGEwJNWDEZMBcGA1UECAwQRGlzdHJpdG8gRmVkZXJhbDES
MBAGA1UEBwwJQ295b2Fjw6FuMRUwEwYDVQQtEwxTQVQ5NzA3MDFOTjMxITAfBgkq
hkiG9w0BCQIMElJlc3BvbnNhYmxlOiBBQ0RNQTAeFw0xNzA1MTgwMzU0NTZaFw0y
MTA1MTgwMzU0NTZaMIHlMSkwJwYDVQQDEyBBQ0NFTSBTRVJWSUNJT1MgRU1QUkVT
QVJJQUxFUyBTQzEpMCcGA1UEKRMgQUNDRU0gU0VSVklDSU9TIEVNUFJFU0FSSUFM
RVMgU0MxKTAnBgNVBAoTIEFDQ0VNIFNFUlZJQ0lPUyBFTVBSRVNBUklBTEVTIFND
MSUwIwYDVQQtExxBQUEwMTAxMDFBQUEgLyBIRUdUNzYxMDAzNFMyMR4wHAYDVQQF
ExUgLyBIRUdUNzYxMDAzTURGUk5OMDkxGzAZBgNVBAsUEkNTRDAxX0FBQTAxMDEw
MUFBQTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJdUcsHIEIgwivvA
antGnYVIO3+7yTdD1tkKopbL+tKSjRFo1ErPdGJxP3gxT5O+ACIDQXN+HS9uMWDY
naURalSIF9COFCdh/OH2Pn+UmkN4culr2DanKztVIO8idXM6c9aHn5hOo7hDxXMC
3uOuGV3FS4ObkxTV+9NsvOAV2lMe27SHrSB0DhuLurUbZwXm+/r4dtz3b2uLgBc+
Diy95PG+MIu7oNKM89aBNGcjTJw+9k+WzJiPd3ZpQgIedYBD+8QWxlYCgxhnta3k
9ylgXKYXCYk0k0qauvBJ1jSRVf5BjjIUbOstaQp59nkgHh45c9gnwJRV618NW0fM
eDzuKR0CAwEAAaMdMBswDAYDVR0TAQH/BAIwADALBgNVHQ8EBAMCBsAwDQYJKoZI
hvcNAQELBQADggIBABKj0DCNL1lh44y+OcWFrT2icnKF7WySOVihx0oR+HPrWKBM
Xxo9KtrodnB1tgIx8f+Xjqyphhbw+juDSeDrb99PhC4+E6JeXOkdQcJt50Kyodl9
URpCVWNWjUb3F/ypa8oTcff/eMftQZT7MQ1Lqht+xm3QhVoxTIASce0jjsnBTGD2
JQ4uT3oCem8bmoMXV/fk9aJ3v0+ZIL42MpY4POGUa/iTaawklKRAL1Xj9IdIR06R
K68RS6xrGk6jwbDTEKxJpmZ3SPLtlsmPUTO1kraTPIo9FCmU/zZkWGpd8ZEAAFw+
ZfI+bdXBfvdDwaM2iMGTQZTTEgU5KKTIvkAnHo9O45SqSJwqV9NLfPAxCo5eRR2O
Gibd9jhHe81zUsp5GdE1mZiSqJU82H3cu6BiE+D3YbZeZnjrNSxBgKTIf8w+KNYP
M4aWnuUMl0mLgtOxTUXi9MKnUccq3GZLA7bx7Zn211yPRqEjSAqybUMVIOho6aqz
kfc3WLZ6LnGU+hyHuZUfPwbnClb7oFFz1PlvGOpNDsUb0qP42QCGBiTUseGugAzq
OP6EYpVPC73gFourmdBQgfayaEvi3xjNanFkPlW1XEYNrYJB4yNjphFrvWwTY86v
L2o8gZN0Utmc5fnoBTfM9r2zVKmEi6FUeJ1iaDaVNv47te9iS1ai4V4vBY8r
-----END CERTIFICATE-----
</cert_pem>
         <llave_pem xsi:type="xsd:string">-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCXVHLByBCIMIr7
wGp7Rp2FSDt/u8k3Q9bZCqKWy/rSko0RaNRKz3RicT94MU+TvgAiA0Fzfh0vbjFg
2J2lEWpUiBfQjhQnYfzh9j5/lJpDeHLpa9g2pys7VSDvInVzOnPWh5+YTqO4Q8Vz
At7jrhldxUuDm5MU1fvTbLzgFdpTHtu0h60gdA4bi7q1G2cF5vv6+Hbc929ri4AX
Pg4sveTxvjCLu6DSjPPWgTRnI0ycPvZPlsyYj3d2aUICHnWAQ/vEFsZWAoMYZ7Wt
5PcpYFymFwmJNJNKmrrwSdY0kVX+QY4yFGzrLWkKefZ5IB4eOXPYJ8CUVetfDVtH
zHg87ikdAgMBAAECggEALS8Z1KJXzVIxLVoWcRh0kAcxPMJlIgsvaz6xrTTaf2Ui
mcAjIvMuXPZTbR/MEuD4SS+Pq1xMeoz8UV5cM50vkm3QLoU9n0SyrQVJQ+6q4Npl
9SwuMqNXVS/l1YEEcJNTYwq7rE5OtAYIPn7s7i5dhJIUKgeZsu7xcf9VpdLgjVCD
qGgJw/EfhagR7iPF+PKoeyRyBZI9xuHmtElHVgn2/Qv/16UJv0YpAqRgVq7YQzZC
c7yo0Y2+3dqHabRg+MnIKkN4pBFBzYxsjwM7YUDk/8zFlF5kwCS74ep0JWWSYAJ1
3DYDtCYSyWk1DvxX9Srv/S2htZM6MnhboafjLch4QQKBgQDRGGLpYdqGt6/cXKQe
JGWFrG33AMiYKrd4NOw7LK7kzrQESeaeAXSwr2eOnNV3tDMyslkjpC05m3Lbefsh
Ul6Qj/Qj9PEIpv7e4X4r++O/FsA9X6iQFicMEDzRYYjm4AfFggYrhzmjXh2rNACL
KRX5i9wIRGQuoAG7KuZyYWSBuwKBgQC5Rsv75S6FNUpKe8RC2nw13Vaf9uua2W1+
spg2pWfKqw88vvFATQOj9A9aFJ+wqrvwRziua5xtbch9gHK7M9Nnl565Tk8muueO
OUBaFeHYXsDaYZfTFILOZU4/b6//r6QK2cO892VXyUydbRXavCpRX8s2EoxtwfFG
mgbStX+HBwKBgQCICHKJXXU7QhPyrH7FcW5vKgAcu3DFtrzIQr4RvX9HMsdhJucX
kuDk9ijMWnJyv1Szvd5KVsxpdx2hdlmQkzMcn9r47alGtMaKIG/ik6zWrCmDhFF4
9ECRE5tNqUPU2JmVwILdHMu94kQxFtLntmIqiPgslLoMr2KQ71cfwQcPcwKBgQCk
iNKtqCFf+qs26iKonA6iZyV+eXFR2rT6RvAV114NBUxKzebBC6On/h2ECbymz3iH
MTiM7NPF+jCKA3/f725WGLfEKF7yLhlknEMhvT0LQVpSlUiXEyf20tBiVXUew4QS
fsDtF2bQRtvbEfzOezu5eDCmnGJJNmpmIHLevH+8EQKBgF9Ff09RISQJHbABka8f
wj8sdBKWG3TUQ2SwQ9U3L/Y/unuyaRUF+J3wFRYBMQGu0jzLG5TFfAVZAc3VJCBj
xG6K8WnJS6OM9ycV0qBa2WnkC7M7uAt4K9IEIqlOljY/R2tBN7qHZwE7nCLS88rv
L5YWIiKp71SlXyoGLfM0h7bl
-----END PRIVATE KEY-----</llave_pem>
         <llave_password xsi:type="xsd:string">12345678a</llave_password>
      </urn:cancelar_cfdi_certs>
   </soapenv:Body>
</soapenv:Envelope>
```

Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) y una vez hecho esto nos saldrá el resultado.

![](http://i.imgur.com/UI0JkHw.png)
