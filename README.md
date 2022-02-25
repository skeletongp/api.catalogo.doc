# Api Catálogo
## 1.Generalidades
### 1.1 Descripción
*Esta api permite gestionar catálogo de productos en línea, agrupados por categoría y ciertas propiedades. Está desarrollada con Laravel 9, PHP 8.1, MySql y Sanctum. Entre las funcionalidades de esta api, está la posibilidad de registrar los productos y añadirles propiedades, así como para los clientes poder solicitar cotizaciones con detalles dentro de la misma plataforma.
```
FRAMEWORK: Laravel 9
LANGUAGE: PHP 8.1
DATABASE: MySQL 5.7
AUTHENTICATION: Laravel Sanctum Bearer
```
## 2.Admin
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
```
ENDPOINT: user
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
```
ENDPOINT: user/{user->slug}
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
```
ENDPOINT: user
METHOD GET
AUTH: true
SCOPES: see_users
PARAMS (optionals): 
filter[id:int]
order => id | created_at | name | email | deleted_at (required trashed=true|only)
sort => asc|desc
trash => true:withTrashed | only:onlyTrashed
BODY: none
RETURN: json=> users
```
#### 2.2.4 Devolver un Usuario
```
ENDPOINT: user/{user->slug}
METHOD GET
AUTH: true
SCOPES: see_users
PARAMS: none
BODY: none
RETURN: json=> user
```
#### 2.2.4 Eliminar un Usuario
```
ENDPOINT: user/{user->slug}
METHOD DELETE
AUTH: true
SCOPES: delete_users
PARAMS: none
BODY: none
RETURN: json=> user->slug
```
