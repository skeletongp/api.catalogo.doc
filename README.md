- [Api Catálogo](#api-catálogo)
  - [## 1.Generalidades](#-1generalidades)
    - [1.1 Descripción](#11-descripción)
  - [## 2.Admin](#-2admin)
    - [2.1 Auth](#21-auth)
      - [2.1.1 Registrarse](#211-registrarse)
      - [2.1.2 Loguearse](#212-loguearse)
    - [2.2 User](#22-user)
      - [2.2.1 Crear Usuario](#221-crear-usuario)
      - [2.2.2 Editar Usuario](#222-editar-usuario)
      - [2.2.3 Devolver todos los Usuarios](#223-devolver-todos-los-usuarios)
      - [2.2.4 Devolver un Usuario](#224-devolver-un-usuario)
      - [2.2.4 Eliminar un Usuario](#224-eliminar-un-usuario)
# Api Catálogo
![AtrionTech Soluciones Digitales](https://res.cloudinary.com/dboafhu31/image/upload/c_scale,w_150/v1640200158/j2tkinbsvvlzlrksy7nk.png)
## 1.Generalidades
------
### 1.1 Descripción
Esta api permite gestionar catálogo de productos en línea, agrupados por categoría y ciertas propiedades. Está desarrollada con Laravel 9, PHP 8.1, MySql y Sanctum. Entre las funcionalidades de esta api, está la posibilidad de registrar los productos y añadirles propiedades, así como para los clientes poder solicitar cotizaciones con detalles dentro de la misma plataforma.
```
FRAMEWORK: Laravel 9
LANGUAGE: PHP 8.1
DATABASE: MySQL 5.7
AUTHENTICATION: Laravel Sanctum Bearer
```
## 2.Admin
---
Funciones recomendadas únicamente para el administrador, ya que tienen que ver con el control del sistema y algunas de estas funciones tienen requerimientos específicos definidos como *scopes*. 

### 2.1 Auth
Funciones de autenticación de usuario. A través de estos endpoint, cualquier persona puede registrarse y loguearse como usuario del sistema, siempre que tenga los accesos requeridos, de acuerdo a la especificación de cada uno.

#### 2.1.1 Registrarse
Permite crear una cuenta de usuario con los datos básicos (nombre, correo, teléfono y contraseña). Al crear un usuario a través de este endpoint, se le asignan todos los permisos de la api, razón por la cual sólo se puede acceder a esta función con un token válido previamente registrado en el sistema, el cual debe incluirse en el encabezado de la petición como tipo Bearer. Todos los campos del body son obligatorios para el registro, el cual devuelve una respuesta de tipo json con el usuario registrado, su token y el tipo de token.
```
ENDPOINT: /admin/register
METHOD POST
AUTH: true | required existing token
SCOPES: create_users
PARAMS: none
BODY:
name=> required|string|max:255;  email=> required|email|max:255|unique
password=> required|string|min:8; phone=>required|string|max:12
RETURN: json=> user, token , token_type
```
#### 2.1.2 Loguearse
Este endpoint permite al usuario loguearse en la api, para lo cual debe proporcionar un correo y una contraseña válidos. Al recibir estos valores y validarlos, la api le genera un access token, el cual es devuelto en un respuesta tipo json, así como el usuario logueado y el tipo de token que le corresponde. IMPORTANTE: siempre que un usuario hace login a través de este endpoint, se revoka cualquier token antes registrado, de modo que siempre se mantendrá sólo el último creado.
```
ENDPOINT: /admin/login
METHOD POST
AUTH: false
SCOPES: none,
PARAMS: none
BODY:
email=> required|email|max:255|unique
password=> required|string|min:8;
RETURN: json=> user, token , token_type
```
### 2.2 User
Si bien los dos endpoints anteriores manejan el registro de usuarios en el sistema, están localizados en una categoría propia, ya que son las rutas para registro y login, es decir, para autenticación de usuario en el sistema, por lo que también generan o modifican su access token. En este apartado se incluyen los métodos para gestionar los usuarios sin intervenir la autenticación de los mismos. Todas estas rutas sólo son accesibles para usuarios registrados y con los permisos correspondientes.
#### 2.2.1 Crear Usuario
En esta parte un usuario con el scope (permiso) correspondeinte podrá registrar nuevos usuarios en el sistema y asignarle los permisos que conside de los que mencionan en el aparado **Scopes de la api**. Todos los campos del body son obligatorios, con excepción del "scopes". Este último es un array personal que contiene los permisos que se desea asignar al usuario. En caso de no enviarlo, el usuario tendrá todos los permisos del sistema; en caso de enviar un array vacío, el usuario no tendrá ningún permiso. El campo slug se genera automáticamente a partir del nombre suministrado. Retorna una respuesta json con el usuario creado.
```
ENDPOINT: /user
METHOD POST
AUTH: true
SCOPES: create_users
PARAMS: none
BODY:
name=> required|string|max:255;  email=> required|email|max:255|unique
password=> required|string|min:8; phone=>required|string|max:12
scopes=> optional|array
RETURN: json=> user
```
#### 2.2.2 Editar Usuario
Este endpoint recibe un *formdata* en el body y el slug de un usuario. Permite editar los datos de este usuario, por lo que todos los campos definidos para el body son requeridos. Recordar que, al editar un usuario, su slug se actualizará aunque el nombre no sea modificado, ya que se añade un número aleatorio al final de éste. Este endpoint puede recibir un campo llamada *scopes* de tipo array, el cual es opcional. Si se envía dicho campo, los permisos del usuario serán sustituidos por los valores de este array. Si no se envía, el usuario conservará sus permisos. Devuelve un json con los nuevos datos del usuario.
```
ENDPOINT: /user/{user->slug}
METHOD PUT
AUTH: true
SCOPES: edit_users
PARAMS: none
BODY:
name=> required|string|max:255;  email=> required|email|max:255|unique
password=> required|string|min:8; phone=>required|string|max:12
scopes=> optional|array (replace older scopes)
RETURN: json=> user
```
#### 2.2.3 Devolver todos los Usuarios
Este endpoint devuelve todos los usuarios registrados, a menos que se establezca algún filtro en los parámetros de la ruta. Se pueden aplicar ordenación, filtro por los campos marcados e incluir los borrados o recuperar sólo los borrados. Si se equiere recuperar el token el usuario, entonces se agrega un parámetro llamado *included* con valor *"tokens"*. Finalmente, puedes aplicar la búsqueda a través del parámaetro *q*, el cual recibe el string que se buscará entre los usuarios.
```
ENDPOINT: /user
METHOD GET
AUTH: true
SCOPES: see_users
PARAMS (optionals):
q=> string
filter[id:int]
order => id | created_at | name | email | deleted_at (required trashed=true|only)
sort => asc|desc
trash => true:withTrashed | only:onlyTrashed
included= tokens
BODY: none
RETURN: json=> users
```
#### 2.2.4 Devolver un Usuario
Este endpoint devuelve el usuario cuyo slug se especifique en la ruta. En este caso sólo aplica el parámetro *include*, con el cua indicamos las relaciones que deseamos cargar, aunque en el caso de los usuarios, sólo está disponible la de tokens, hasta una nueva actualización que incluya otras relaciones. Devuelve un json con el usuario solicitado. 
```
ENDPOINT: /user/{user->slug}
METHOD GET
AUTH: true
SCOPES: see_users
PARAMS: included= tokens
BODY: none
RETURN: json=> user
```
#### 2.2.4 Eliminar un Usuario
Este endpoint permite eliminar al usuario marcado de la base de datos. Sin embargo, no es un borrado absoluto, de modo que dicho usuario podrá ser consultado luego aplicando el parámetro *trashed* al endpoint que devuelve todos los usuarios. Aunque no es un borrado absoluto y requiere permiso de usuario, se recomienda ejecutar esta ruta con un filtro de confirmación previa.
```
ENDPOINT: /user/{user->slug}
METHOD DELETE
AUTH: true
SCOPES: delete_users
PARAMS: none
BODY: none
RETURN: json=> user->slug
```

Si te va gustando cómo va quedando la documentación, por favor sigue nuestra página de Facebook [AtrionTech Soluciones Digitales](https://www.facebook.com/atriontech)
