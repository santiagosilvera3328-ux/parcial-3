Sistema de inventario – Tienda de zapatos
Estudiante: Santiago Silvera 

Descripción: Aplicación de consola en Python para administrar el inventario de una pequeña tienda de zapatos. La persistencia se implementa con SQLAlchemy ORM y SQLite.

1. Archivos del proyecto
tienda_zapatos/
main.py
datos.py
requirements.txt
README.md 


* main.py: menú e interacción con el usuario. Se conserva el archivo suministrado.
* datos.py: conexión, modelo ORM y funciones CRUD.
* requirements.txt: dependencia SQLAlchemy.

El modelo contiene:
- id: entero, clave primaria y autoincremental.
- modelo: texto de hasta 100 caracteres.
- talla: entero.
- precio: número real (Float).
La función inicializar_bd() ejecuta Base.metadata.create_all(engine). Las funciones CRUD trabajan con sesiones Session(engine) y objetos Zapato; no utilizan sqlite3 ni sentencias SQL escritas manualmente.

2. Respuestas a las preguntas del parcial
* Pregunta 1. ¿Qué clase de Python representa la tabla de la base de datos?
La clase Zapato representa la tabla de la base de datos. En el código hereda de Base y declara __tablename__ = "zapatos", por lo que SQLAlchemy relaciona esa clase con la tabla llamada zapatos. Cada atributo declarado con mapped_column() representa una columna: id, modelo, talla y precio. Por ejemplo, id usa Integer y primary_key=True, mientras que modelo usa String(100).

* Pregunta 2. ¿Qué ocurre cuando ejecutas el programa por primera vez?
Al iniciar main(), se llama a datos.inicializar_bd(). Esa función ejecuta Base.metadata.create_all(engine). El engine está conectado a sqlite:///zapateria.db; si el archivo todavía no existe, SQLite lo crea. SQLAlchemy revisa los modelos registrados en Base y crea la tabla zapatos si aún no está creada. Si ya existe, no vuelve a crearla ni elimina sus registros.

* Pregunta 3. ¿Qué recorrido siguen los datos al registrar un zapato?
En main.py, la opción 1 solicita el modelo, la talla y el precio. Después valida que el modelo no esté vacío y que la talla y el precio sean positivos. Si los datos son válidos, llama a datos.crear_zapato(modelo, talla, precio). En datos.py, esa función construye un objeto Zapato, abre una sesión, lo agrega con session.add(zapato) y confirma la operación con session.commit(). Si la función devuelve True, el menú muestra “Zapato registrado correctamente”.

* Pregunta 4. ¿Para qué se utiliza una sesión de SQLAlchemy? ¿Qué ocurre si no confirmas con commit()?
La sesión administra las operaciones entre los objetos Python y la base de datos. En este proyecto se crea con Session(engine) dentro de cada operación. session.add() incorpora el objeto a la sesión, pero session.commit() confirma la transacción y hace persistentes los cambios. Si se agrega o modifica un objeto y no se ejecuta commit(), el cambio no queda confirmado en la base de datos; al cerrar la sesión sin confirmar, la transacción pendiente se revierte.

* Pregunta 5. ¿Cómo funciona la búsqueda de zapatos por ID?
La función buscar_zapato(id_zapato) abre una sesión y ejecuta session.get(Zapato, id_zapato). SQLAlchemy busca el registro cuya clave primaria coincide con el ID. Si lo encuentra, devuelve el objeto Zapato; si no existe, devuelve None. En main.py, el menú comprueba ese resultado y muestra “Zapato no encontrado” cuando recibe None.

* Pregunta 6. ¿Cómo se actualiza el precio mediante el ORM?
La función actualizar_precio(id_zapato, nuevo_precio) obtiene el objeto con session.get(Zapato, id_zapato). Si el resultado es None, devuelve False. Si lo encuentra, cambia su atributo mediante zapato.precio = nuevo_precio y ejecuta session.commit() para guardar el cambio. Finalmente devuelve True, y el menú muestra “Precio actualizado”.

* Pregunta 7. ¿Qué ocurre cuando eliminas un zapato?
La función eliminar_zapato(id_zapato) primero busca el objeto con session.get(). Si no existe, devuelve False. Si existe, lo elimina de la sesión con session.delete(zapato) y confirma la eliminación con session.commit(). Para comprobarlo se puede volver a buscar el mismo ID: buscar_zapato(id_zapato) debe devolver None, y el listado ya no debe mostrar ese registro.

* Pregunta 8. ¿Qué sucede cuando cierras y vuelves a ejecutar el programa?
Los zapatos que no se eliminaron permanecen almacenados porque los cambios se confirman con commit() en el archivo SQLite zapateria.db. Al cerrar el programa termina la sesión, pero no se borra el archivo de base de datos. Al ejecutar nuevamente main.py, inicializar_bd() vuelve a comprobar la estructura y el listado consulta los registros que siguen guardados en el mismo archivo.


3. Conclusión
La aplicación utiliza SQLAlchemy como ORM para representar los zapatos mediante objetos Python y SQLite para conservar los datos en un archivo local. Las funciones de datos.py implementan las operaciones de crear, listar, buscar, actualizar y eliminar. La confirmación de las transacciones mediante commit() permite que los cambios permanezcan después de cerrar el programa.