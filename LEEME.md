# Magic Crud Angular (mgCrud)

El objetivo principal de este m�dulo es demostrar como hacer todas las llamadas a servicios RESTfull que necesitemos de una forma declarativa con 7Kb y con menos de 600 l�neas de JavaScript gracias a la potencia de [Angularjs](https://angularjs.org/). Es decir en un 98% de tu app no tener que escribir ni controladores ni servicios y por tanto no c�digo.
Nos gustan los m�dulos [ngResource](https://docs.angularjs.org/api/ngResource/service/$resource) y [restangular](https://github.com/mgonto/restangular), pero nuestro objetivo es simplemente convencerte que con Angular Magic tu puedes evitar escribir JavaScript en un porcentaje muy alto de tu aplicaci�n.

## Directive mg-ajax

**mg-ajax** es la �nica directiva y tiene solo 4 atributos opcionales.

### Atributo path

Este atributo permite enlazar parte de la ruta al modelo de datos o parametros.
/invoices/{{model.id}}
/invoices/{{params.id}}
El valor por defecto de este atributo es [location.path()](https://docs.angularjs.org/api/ng/service/$location#path) como el action de un Forms de Html.

### Atributo options

Este m�dulo define c�mo y qu� se env�a a la capa servidora y c�mo se sincronizan los datos de respuesta con los ya existentes.
Lo m�s importante es definir qu� verbo http se va a utilizar entre: GET, POST, PUT, PATCH  o DELETE. Para ello se ha escrito un wrapper sobre [$http](https://docs.angularjs.org/api/ng/service/$http) que inserta llamadas a functions JavaScript en el ciclo de vida de la llamada a la servidora. Estas funciones se pueden insertar **before** o despu�s de que la promise [$http](https://docs.angularjs.org/api/ng/service/$http) retorne **success** o **error**. Adem�s te permite declarar distintos command que podr�s bindear a directivas de angular como [ngClick](https://docs.angularjs.org/api/ngTouch/directive/ngClick).
Para la configuraci�n de peticiones se han predefinido las siguientes opciones: **mgQuery**, **mgPost**, **mgPut**, **mgPach** y **mgDelete**. Si algunos de las opciones predefinidas en el m�dulo no se ajustan a tus necesidades puedes crear tus propios m�dulos o bien declarar un json con un esquema espec�fico en el atributo **options**.
Para modificar el comportamiento de un verbo en tu m�dulo o crear uno nuevo puedes implementarlo de la siguiente forma:

```
(function (angular, undefined) {
    var module;
    if (!angular) return;

    module = angular.module('myModule', ['mgCrud']);

    myIndex.$inject = ['mgIndex'];
    function myIndex(mgIndex) {
        return angular.extend(mgIndex, {init:"{index.model.description='hello'}"});
    }

    module.factory('myIndex', myIndex);

})(window.angular);
```

Una vez creado myIndex tu puedes declarar en el html el comportamiento de index de la siguiente forma:

```
<mg-ajax mg-path=�/invoices� mg-options=�myIndex�>
�
</mg-ajax> 
```

o sin declarar una nueva factor�a:

```
<mg-ajax mg-path=�/invoices� mg-options=�{.......}�>
�
</mg-ajax> 
```

### Atributo override

Este atributo se utiliza para reemplazar el comportamiento predifinido de options de forma declarativa. El tipo de dato permitido es un object js declarado como string.

```
<mg-ajax mg-path=�/invoices� mg-options=�mgIndex� mg-override=�{init:�index.filter={page:0,recordsPerPage:15}�}� >
�
</mg-ajax> 


En este ejemplo se utiliza el comportamiento predefinido de mgIndex pero se reemplaza en el init el valor de recordsPerPage por 15. En concreto este html realiza la petici�n:

```
GET /invoices?page=0&recordsPerPage=15
```

### Atributo partialmodel

En determinados escenarios al editar una entidad podemos mostrar una descripci�n de todas clases relacionadas y solo querer actualizar solo sus identificadores (una proyecci�n de la informaci�n almacenada en el cliente). Con esto ahorramos un gran volumen de tr�fico en cada roundtrip. Esto se puede utilizar en todos los verbos sin ninguna restricci�n, pero solo tiene sentido para los verbos http POST, PUT y PATCH.
En este ejemplo de edit solamente se enviar� al servidor el name, puesto que el id ya va en el URI.

```
<mg-ajax data-path="/invoices/get/1" data-options="mgEdit" data-scope="false">
	<mg-ajax data-path="/invoices/put/{{edit.model.id}}" data-options="mgPut" data-scope="false" data-partialmodel='{name:edit.model.name}'>
		<form name="updatefrm" ng-submit="put.accept()">
			<input type="text" ng-model="edit.model.id" />
			<input type="text" ng-model="edit.model.name" />
			<br />
			computed field: {{edit.model.computed}}
			<br />
			<button type="submit">Guardar</button>
			<button type="button" ng-click="put.close()">Cancelar</button>
		</form>
	</mg-ajax>
</mg-ajax>
```

## Modelo de objeto JavaScript para los atributos options y override

### as

El campo as tiene un comportamiento similar a ngController as y crea dentro de tu scope un object con ese nombre. Adem�s se hace un bind contra este objeto con lo que cambia el �mbito de todas las functions y nos permite utilizar "this" en cualquier function de nuestras factor�as. Es requerido y �nico dentro de un mismo scope.

###  config

Aqu� se pueden definir las cabeceras http que se quieren enviar al servidor. De forma temporal estas son almacenadas en un m�dulo y luego son enviadas al servidor cuando se realice la llamada. Con esto se puede tener diferentes proveedores de servicios RESTFul.

El objeto config tiene la siguiente signatura

```
config{url,additionalConfig{...}}
```

Con additionalConfig se pueden resolver los siguientes valores de $http

* headers
* xsrfHeaderName
* xsrfCookieName 
* transformRequest 
* transformResponse 
* cache 
* timeout 
* withCredentials 
* responseType 

Opcionalmente tambi�n se puede configurar un valor por defecto dentro de un m�dulo consumidor.

```
var module = angular.module('myModule', ['mgCrud']);
module.config(function (mgHttpProvider) {
        mgHttpProvider.setDefaultConfig({ url: 'http://localhost:48196' });
});
```

### init

Tiene un comportamiento id�ntico a ngInit. Nos permite actualizar nuestro �mbito (a unos valores predeterminados) dentro del objeto as.

### method

Los m�todos soportados son query, get, post, put, patch y delete. Este campo es requerido.

### service

Es una factor�a wrapper sobre $http a la que se le dota de m�todos cortos para query (que se resuelve con el verbo http GET) y patch. El valor es requerido y por defecto se utiliza �mgHttpFactory� aunque se puede crear un servicio propio y reemplazar este con override o crear tu propia factor�a. Se resuelve con el m�todo get de $injector. 

### before

Conjunto de funciones que se van a ejecutar antes de la llamada al servicio rest. Este campo es optional. Es una factoria donde se van a ejecutar todas las funciones de esta y en el orden en el que est�n declaradas. 
Por ejemplo antes de cada llamada http nos puede interesar establecer dentro de mi objeto �mbito (as) una propiedad show con valor a true. Esto por ejemplo  va a permitir mostrar un spinner que se ocultar� con otra function desde success y error.

```
beforeHttpFactory.$inject = ['phSpinnerFactory'];
function beforeHttpFactory(spinner) {
        return {
            show: spinner.show
        };
}
```
