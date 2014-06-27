# Angular Magic-(mgCrud)

El objetivo principal de este m�dulo es demostrar como hacer todas las llamadas a servicios RESTfull que necesitemos de una forma declarativa con 7Kb y con menos de 600 l�neas de JavaScript gracias a la potencia de [Angularjs](https://angularjs.org/). Es decir en un 98% de tu app no tener que escribir ni controladores ni servicios y por tanto no c�digo.
Nos gustan los m�dulos [ngResource](https://docs.angularjs.org/api/ngResource/service/$resource) y [restangular](https://github.com/mgonto/restangular), pero nuestro objetivo es simplemente convencerte que con Angular Magic tu puedes evitar escribir JavaScript en un porcentaje muy alto de tu aplicaci�n.

## Directive mg-ajax

**mg-ajax** es la �nica directiva y tiene solo 4 atributos opcionales.

### Attribute mg-path

Este atributo permite enlazar parte de la ruta al modelo de datos o parametros.
/invoices/{{model.id}}
/invoices/{{params.id}}
El valor por defecto de este atributo es [location.path()](https://docs.angularjs.org/api/ng/service/$location#path) como el action de un Forms de Html.

### Attribute mg-options

Este m�dulo define c�mo y qu� se env�a a la capa servidora y c�mo se sincronizan los datos de respuesta con los ya existentes.
Lo m�s importante es definir qu� verbo http se va a utilizar entre: GET, POST, PUT, PATCH  o DELETE. Para ello se ha escrito un wrapper sobre [$http](https://docs.angularjs.org/api/ng/service/$http) que inserta llamadas a functions JavaScript en el ciclo de vida de la llamada a la servidora. Estas funciones se pueden insertar **before** o despu�s de que la promise [$http](https://docs.angularjs.org/api/ng/service/$http) retorne **success** o **error**. Adem�s te permite declarar distintos command que podr�s bindear a directivas de angular como [ngClick](https://docs.angularjs.org/api/ngTouch/directive/ngClick).
Para la configuraci�n de peticiones se han predefinido las siguientes opciones: **mgQuery**, **mgPost**, **mgPut**, **mgPach** y **mgDelete**. Si algunos de las opciones predefinidas en el m�dulo no se ajustan a tus necesidades puedes crear tus propios m�dulos o bien declarar un json con un esquema espec�fico en el atributo **mg-options**.