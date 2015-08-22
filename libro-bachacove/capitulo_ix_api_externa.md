Capítulo 9. Externa API - Integración con otros sistemas

Hasta ahora,	hemos estado trabajando con el codigo del lado de servidor.Sin embargo, el servidor Odoo también proporciona una API externa, que se utiliza por su cliente web y también está disponible para otras aplicaciones cliente.

En este capítulo, vamos a aprender cómo utilizar la API externa Odoo de nuestros propios programas cliente. Para simplificar, vamos a centrarnos en los clientes basados en Python.
 

**Configuración de un cliente Python**

La API Odoo se puede acceder externamente utilizando dos protocolos diferentes: XML-RPC y JSON-RPC. Cualquier programa externo capaz de implementar un cliente para uno de estos protocolos será capaz de interactuar con un servidor de Odoo. Para evitar la introducción de los lenguajes de programación adicionales, vamos a seguir usando Python para explorar la API externa.

Hasta ahora, hemos estado corriendo código Python sólo en el servidor. Esta vez, vamos a utilizar Python en el lado del cliente, por lo que es posible que pueda necesitar para hacer algo de configuración adicional en su estación de trabajo.

Para seguir los ejemplos de este capítulo, tendrá que ser capaz de ejecutar archivos de Python en su equipo de trabajo. El servidor Odoo requiere Python 2, pero nuestro cliente RPC puede ser en cualquier idioma, así que Python 3 estará bien. Sin embargo, ya que algunos lectores pueden estar ejecutando el servidor en la misma máquina que están trabajando (hola usuarios de Ubuntu!), Será más fácil para todo el mundo a seguir si nos atenemos a Python 2.

Si está usando Ubuntu o un Macintosh, probablemente Python ya está instalado. Abra una consola de terminal, escriba python, y usted debe ser recibido con algo como lo siguiente:
``` 
Python	2.7.8	(default,	Oct	20	2014,	15:05:29) [GCC	4.9.1]	on	linux2 Type	"help",	"copyright",",	"credits"	or	"license"	for	more	information. 
>>>  
```
*Nota*  
*
Los usuarios de Windows pueden encontrar un instalador y también obtener rápidamente a la velocidad. Los paquetes oficiales de instalación se pueden encontrar en	 [https://www.python.org/downloads/](https://www.python.org/downloads/).*  
 
 
**Llamar a la API Odoo usando XML-RPC**

El método más sencillo para acceder al servidor utiliza XML-RPC. Podemos utilizar la biblioteca xmlrpclib de la biblioteca estándar de Python para esto. Recuerde que estamos programando un cliente con el fin de conectarse a un servidor, por lo que necesitamos una instancia de servidor Odoo corriendo a conectar. En nuestros ejemplos, vamos a suponer que se está ejecutando una instancia de servidor Odoo en el mismo equipo (localhost), pero puedes utilizar cualquier dirección IP o el nombre del servidor, si el servidor se está ejecutando en otra máquina.
 

**Abrir una conexión de XML-RPC**

Vamos a conseguir un contacto puño con la API externa. Iniciar una consola de Python y escriba lo siguiente: 
```
>>> import xmlrpclib 
>>> srv,	db	=	'http://localhost:8069',	'v8dev' >>>	user,	pwd	=	'admin',	'admin' 
>>>	common	=	xmlrpclib.ServerProxy('%s/xmlrpc/2/common'	%	srv) 
>>>	common.version() {'server_version_info':	[8,	0,	0,	'final',	0],	'server_serie':	'8.0',	 'server_version':	'8.0',	'protocol_version':	1} 
```
Aquí, importamos la biblioteca xmlrpclib y luego establecimos algunas variables con la información para la ubicación del servidor de conexión y credenciales. Siéntase libre de adaptar éstos a su configuración específica.

A continuación, hemos creado acceso a los servicios del servidor público (que no requieren un inicio de sesión), expuestas en el / xmlrpc / 2 / punto final común. Uno de los métodos que están disponibles es la versión (), que inspecciona la versión del servidor. Lo utilizamos para confirmar que nos podemos comunicar con el servidor.

Otro método público es autenticar (). De hecho, esto no crea una sesión, ya que podría estar hecho creer. Este método sólo confirma que el nombre de usuario y la contraseña son aceptadas y devuelve el ID de usuario que se debe utilizar en las solicitudes en lugar del nombre de usuario, como se muestra aquí:
```
>>>	uid	=	common.authenticate(db,	user,	pwd,	{}) 
>>>	print	uid 1 
```
 
**La lectura de datos desde el servidor**

Con XML-RPC, ninguna sesión se mantiene y las credenciales de autenticación se envía con cada solicitud. Esto añade cierta sobrecarga al protocolo, pero hace que sea más fácil de usar.
A continuación, hemos creado el acceso a los métodos de servidor que necesitan un ingreso para ser visitada. Estos están expuestos en el / xmlrpc / 2 / objeto de punto final, como se muestra en la siguiente:
```
>>>	api	=	xmlrpclib.ServerProxy('%s/xmlrpc/2/object'	%	srv) 
>>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'search_count',		[[]]) 70 
```
Aquí, estamos haciendo nuestro primer acceso a la API de servidor, la realización de un recuento de los registros asociados. Los métodos se denominan utilizando el método execute_kw () que toma los siguientes argumentos:
El nombre de la base de datos para conectarse a El ID de usuario de conexión La contraseña de usuario El identificador de modelo objetivo name El método para llamar a una lista de argumentos posicionales Un diccionario opcional con argumentos de palabras clave

El ejemplo anterior llama al método search_count del modelo res.partner con un argumento posicional, [], y no hay argumentos de palabra clave. El argumento posicional es un dominio de búsqueda; ya que estamos ofreciendo una lista vacía, cuenta todos los socios.

Acciones frecuentes son la búsqueda y lectura. Cuando se llama desde la RPC, el método de búsqueda devuelve una lista de los ID coinciden con un dominio. El método de exploración no está disponible en la RPC y de lectura se debe utilizar en su lugar para, dada una lista de ID de registro, recuperar sus datos, como se muestra en el siguiente código:
```
>>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'search',	[[('country_id',	 '=',	'be'),	('parent_id',	'!=',	False)]]) [43,	42] 
>>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'read',		[[43]],	{'fields':	 ['id',	'name',	'parent_id']}) [{'parent_id':	[7,	'Agrolait'],	'id':	43,	'name':	'Michel	Fletcher'}] 
```
Tenga en cuenta que para el método de lectura, estamos utilizando un argumento posicional para la lista de ID,
[43], y un argumento de palabra clave, campos. También podemos observar que los campos relacionales se recuperan como un par, con acreditación y la pantalla el nombre del registro relacionado. Eso es algo a tener en cuenta a la hora de procesar los datos en el código.

La combinación búsqueda y leer es tan frecuente que se proporciona un método search_read para realizar ambas operaciones en un solo paso. El mismo resultado que los dos pasos anteriores se puede obtener con la siguiente:
```
>>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'search_read',	 [[('country_id',	'=',	'be'),	('parent_id',	'!=',	False)]],	{'fields':	 ['id',	'name',	'parent_id']}) 
```
El método search_read comporta como lectura, pero espera que como primer argumento posicional de un dominio en lugar de una lista de ID. Vale la pena mencionar que el argumento de campo sobre lectura y
search_read no es obligatorio. Si no se proporciona, se recuperarán todos los campos.
 

**Llamar a otros métodos**

Los métodos modelo restantes están expuestos a través de RPC, a excepción de los que empiezan con "_" que se consideran privados. Esto significa que podemos utilizar a crear, escribir y desvincular de modificar los datos en el servidor de la siguiente manera:
```
>>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'create',	[{'name':	 'Packt'}]) 75 >>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'write',	[[75],	{'name':	 'Packt	Pub'}]) True 
>>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'read',	[[75],	['id',	 'name']]) [{'id':	75,	'name':	'Packt	Pub'}] >>>	api.execute_kw(db,	uid,	pwd,	'res.partner',	'unlink',	[[75]]) True 
```
Una de las limitaciones del protocolo XML-RPC es que no admite valores Ninguno. La implicación es que los métodos que no devuelven nada, no se podrán utilizar a través de XML-RPC, ya que están regresando implícitamente Ninguno. Esta es la razón por métodos siempre deben terminar con al menos una declaración verdadera rentabilidad.
 
![328_1](/images/Odoo Development Essentials - Daniel Reis-328_1.jpg)

Escribir una aplicación de escritorio Notas Vamos a hacer algo interesante con la API RPC. ¿Qué pasa si los usuarios podían gestionar sus Odoo tareas a realizar directamente desde el escritorio de su ordenador? Vamos a escribir una aplicación Python simple de hacer precisamente eso, como se muestra en la siguiente captura de pantalla:

Para mayor claridad, vamos a dividirlo en dos archivos: uno en cuestión de interactuar con el backend del servidor, note_api.py, y otro con la interfaz gráfica de usuario, note_gui.py.
 

**Capa de comunicación con Odoo**

Vamos a crear una clase para establecer la conexión y almacenar su información. Debe exponer dos métodos: get () para recuperar datos de tareas y set () para crear o tareas de actualización.
Seleccione un directorio para organizar los archivos de la aplicación y crear el archivo note_api.py. Podemos empezar por añadir el constructor de la clase, de la siguiente manera: 
```
import	xmlrpclib class	NoteAPI(): 				def	__init__(self,	srv,	db,	user,	pwd): 								common	=	xmlrpclib.ServerProxy( 												'%s/xmlrpc/2/common'	%	srv) 								self.api	=	xmlrpclib.ServerProxy( 												'%s/xmlrpc/2/object'	%	srv) 								self.uid	=	common.authenticate(db,	user,	pwd,	{}) 								self.pwd	=	pwd 								self.db	=	db 								self.model	=	'todo.task' 
```
Aquí almacenamos en el objeto creado toda la información necesaria para ejecutar las llamadas en un modelo: la referencia de la API, uid, contraseña, nombre de base de datos, y el modelo de su uso.
A continuación vamos a definir un método de ayuda para ejecutar las llamadas. Se aprovecha de los datos de objeto almacenados para proporcionar una firma de función más pequeño, como se muestra a continuación:
```
				def	execute(self,	method,	arg_list,	kwarg_dict=None): 								return	self.api.execute_kw( 												self.db,	self.uid,	self.pwd,	self.model, 												method,	arg_list,	kwarg_dict	or	{}) 
```
Ahora podemos utilizarlo para implementar el nivel superior get () y establecer métodos ().
El método get () aceptará una lista opcional de ID para recuperar. Si no hay ninguno en la lista, se devolverán todos los registros, como se muestra aquí:
```
				def	get(self,	ids=None): 								domain	=	[('id','	in',	ids)]	if	ids	else	[] 								fields	=	['id',	'name'] 								return	self.execute('search_read',	[domain,	fields]) 
```
El método set () tendrá como argumentos el texto tarea de escribir, y un ID opcional. Si el ID no se proporciona, se creará un nuevo registro. Devuelve el ID del registro escrito o creado, como se muestra aquí:
```
				def	set(self,	text,	id=None): 								if	id: 												self.execute('write',	[[id],	{'name':	text}]) 								else: 												vals	=	{'name':	text,	'user_id':	self.uid} 												id	=	self.execute('create',	[vals])
``` 								return	id 
Vamos a terminar el archivo con un pequeño trozo de código de prueba que se ejecutará si corremos el archivo de Python:
``` 
if	__name__	==	'__main__': 				srv,	db	=	'http://localhost:8069',	'v8dev' 				user,	pwd	=	'admin',	'admin' 				api	=	NoteAPI(srv,	db,	user,	pwd) 				from	pprint	import	pprint 				pprint(api.get()) 
```
Si ejecutamos el script en Python, debemos ver el contenido de nuestras tareas a realizar impresas. Ahora que tenemos un simple envoltorio alrededor de nuestro backend Odoo, vamos a tratar con la interfaz de usuario de escritorio.
 

**Creación de la interfaz gráfica de usuario**

Nuestro objetivo aquí era aprender a escribir la interfaz entre una aplicación externa y el servidor Odoo, y esto se hizo en la sección anterior. Pero sería una pena no ir un paso más allá y en realidad haciendo que esté disponible para el usuario final.
Para mantener la configuración de lo más simple posible, utilizaremos Tkinter para implementar la interfaz gráfica de usuario. Puesto que es parte de la biblioteca estándar, que no requiere ninguna instalación adicional. No es nuestro objetivo de explicar cómo funciona Tkinter, por lo que será corto en las explicaciones al respecto.

Cada tarea debe tener una pequeña ventana amarilla en el escritorio. Estas ventanas tendrán un solo widget de texto. Al presionar Ctrl * * + * N * se abrirá una nueva nota, y presionando Ctrl * * + * S * escribirá el contenido de la nota actual en el servidor Odoo.

Ahora, junto con el archivo note_api.py, cree un nuevo archivo note_gui.py. Será primero importar los módulos Tkinter y widgets que utilizaremos, y entonces la clase NoteAPI, como se muestra en la siguiente:
desde Tkinter Texto importación, Tk tkMessageBox importación de NoteAPI importación note_api

A continuación, creamos nuestro propio widget texto derivado de la Tkinter. Cuando se crea una instancia, se espera una referencia de la API, que se utilizará para la acción guardar y también el texto de la tarea y de identificación, como se muestra en la siguiente:
```
class	NoteText(Text): 				def	__init__(self,	api,	text='',	id=None): 								self.master	=	Tk() 								self.id	=	id 								self.api	=	api 								Text.__init__(self,	self.master,	bg='#f9f3a9', 																						wrap='word',	undo=True) 								self.bind('<Control-n>',	self.create) 								self.bind('<Control-s>',	self.save) 								if	id: 												self.master.title('#%d'	%	id) 								self.delete('1.0',	'end') 								self.insert('1.0',	text) 								self.master.geometry('220x235') 								self.pack(fill='both',	expand=1) 
```
El Tk () crea una nueva ventana de interfaz de usuario y el texto de widget se coloca en su interior, por lo que la creación de una nueva instancia NoteText abre automáticamente una ventana de escritorio.
A continuación, vamos a poner en práctica las acciones de crear y guardar. La acción de crear abre una nueva ventana vacía, pero se almacenará en el servidor sólo cuando se realiza una acción de guardar, como se muestra en el siguiente código:
```
				def	create(self,	event=None): 								NoteText(self.api,	'') 				def	save(self,	event=None): 
 
 								text	=	self.get('1.0',	'end') 								self.id	=	self.api.set(text,	self.id) 								tkMessageBox.showinfo('Info',	'Note	%d	Saved.'	%	self.id) 
```
La acción ahorrar se puede realizar ya sea en existentes o nuevas tareas, pero no hay necesidad de preocuparse por eso aquí desde esos casos ya son manejados por el método set () del
NoteAPI.

Por último, vamos a añadir el código que recupera y crea todas las ventanas de la nota cuando se inicia el programa, como se muestra en el siguiente código:
```
if	__name__	==	'__main__': 				srv,	db	=	'http://localhost:8069',	'v8dev' 				user,	pwd	=	'admin',	'admin' 				api	=	NoteAPI(srv,	db,	user,	pwd) 				for	note	in	api.get(): 								x	=	NoteText(api,	note['name'],	note['id']) 				x.master.mainloop() 
```
La carreras Mainloop último comando () en la última ventana Nota creado, para iniciar la espera de los acontecimientos de la ventana.

Esta es una aplicación muy básica, pero el punto aquí es hacer un ejemplo de maneras interesantes para aprovechar la API Odoo RPC.
 

**Introduciendo el cliente ERPpeek**

ERPpeek es una herramienta versátil que puede ser utilizado tanto como una línea de comandos interactiva Interface (CLI) y como una biblioteca de Python, con una forma más cómoda API que el proporcionado por xmlrpclib. Está disponible a partir del índice PyPI y puede ser instalado con lo siguiente:
```
$	pip	install	-U	erppeek  
```
En un sistema Unix, si está instalando el sistema de par en par, es posible que deba anteponer sudo al comando.
 

**El ERPpeek API**

La biblioteca erppeek proporciona una interfaz de programación, envolviéndose alrededor xmlrpclib, que es similar a la interfaz de programación que tenemos para el código del lado del servidor.
Nuestro punto aquí es proporcionar una visión de lo ERPpeek tiene para ofrecer, y no proporcionar una explicación completa de todas sus características.

Podemos empezar por la reproducción de nuestros primeros pasos con xmlrpclib usando erppeek de la siguiente manera:
```
>>>	import	erppeek 
>>>	api	=	erppeek.Client('http://localhost:8069',	'v8dev',	'admin',	 'admin') 
>>>	api.common.version() >>>	api.count('res.partner',	[]) >>>	api.search('res.partner',	[('country_id',	'=',	'be'),	('parent_id',	 '!=',	False)]) >>>	api.read('res.partner',	[43],	['id',	'name',	'parent_id']) 
```
Como se puede ver, las llamadas a la API utilizan menos argumentos y son similares a las contrapartes del lado del servidor.

Pero ERPpeek no se detiene aquí, y también proporciona una representación de modelos. Tenemos las siguientes dos formas alternativas para obtener una instancia para un modelo, ya sea usando el método del modelo () o acceder a un atributo en caso de camellos:
```
>>>	m	=	api.model('res.partner') 
>>>	m	=	api.ResPartner 
```
Ahora podemos realizar acciones en que el modelo de la siguiente manera:
```
>>>	m.count([('name',	'like',	'Packt%')]) 1 
>>>	m.search([('name',	'like',	'Packt%')]) [76] 
```
También proporciona la representación de objetos del lado del cliente para los registros de la siguiente manera:
```
>>>	recs	=	m.browse([('name',	'like',	'Packt%')]) 
>>>	recs <RecordList	'res.partner,[76]'> 
>>>	recs.name ['Packt'] 
```
Como se puede ver, ERPpeek va un largo camino desde xmlrpclib llano, y hace posible escribir código que puede ser reutilizado lado del servidor con poca o ninguna modificación.
 

**El ERPpeek CLI**

No sólo puede erppeek ser utilizado como una biblioteca Python, también es un CLI que se puede utilizar para realizar acciones administrativas en el servidor. Cuando el comando shell Odoo proporcionó una sesión interactiva local en el servidor host, erppeek ofrece una sesión interactiva remota en un cliente de toda la red.

La apertura de una línea de comandos, podemos echar un vistazo a las opciones disponibles, como se muestra en la siguiente:
```
$	erppeek	--help  
```
Vamos a ver una sesión de muestra de la siguiente manera:
```
$	erppeek	--server='http://localhost:8069'	-d	v8dev	-u	admin Usage	(some	commands): 				models(name)																				#	List	models	matching	pattern 				model(name)																					#	Return	a	Model	instance (...) Password	for	'admin': Logged	in	as	'admin' v8dev	
>>>	model('res.users').count() 3 v8dev	
>>>	rec	=	model('res.partner').browse(43) v8dev	
>>>	rec.name 'Michel	Fletcher'  
```
Como se puede ver, se hizo una conexión con el servidor, y el contexto de ejecución proporciona una referencia al método de modelo () para obtener instancias de modelo y realizar acciones en ellos.

La instancia erppeek.Client utilizado para la conexión también está disponible a través de la variable de cliente. En particular, proporciona una alternativa al cliente web para gestionar los siguientes módulos instalados:

- client.modules():	Esto puede buscar y módulos lista disponible o instalada
- client.install():	Esto realiza la instalación del módulo
- client.upgrade():	Este órdenes módulos para actualizar 
- client.uninstall():	Esto desinstala módulos 

Así, ERPpeek también puede proporcionar un buen servicio como una herramienta de administración remota para servidores Odoo. 
 

**Resumen**

Nuestro objetivo para este capítulo era aprender cómo funciona la API externa y lo que es capaz de hacer. Empezamos explorarlo usando un simple Python cliente XML-RPC, pero la API externa se puede utilizar desde cualquier lenguaje de programación. De hecho, los documentos oficiales proporcionan ejemplos de código para Java, PHP y Ruby.

Hay una serie de bibliotecas para manejar XML-RPC o JSON-RPC, algunos genérico y algunos específicos para el uso con Odoo. Nosotros no intentamos señalar ninguna biblioteca, en particular, a excepción de erppeek, ya que no es sólo un envoltorio probada para el / OpenERP XML-RPC Odoo sino porque también es una valiosa herramienta para la gestión de servidor remoto e inspección.

Hasta ahora, hemos utilizado nuestras instancias de servidor Odoo para el desarrollo y pruebas. Pero para tener un servidor de grado de producción, hay configuraciones de seguridad y optimización adicionales que necesitan ser hechas. En el siguiente capítulo, nos centraremos en ellos.
