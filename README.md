# Sistema de Inventario de Zapatos — SQLAlchemy ORM

## 1. Información del proyecto

| Información | Descripción |
|---|---|
| Estudiante | Rafael Jose Saltarin Escobar |
| Asignatura | DESARROLLO DE APLICACIONES CON ACCESO A DATOS |
| Docente | EDUARDO PIMIENTA LEON |
| Fecha | 17/11/2026 |
| Lenguaje de programación | Python |
| ORM utilizado | SQLAlchemy |
| Base de datos | SQLite |

## 2. Descripción del proyecto

**¿En qué consiste el programa?**

Es una aplicación de consola para administrar el inventario de una tienda de zapatos. Maneja la información de cada zapato (ID, modelo, talla y precio) y la guarda de forma permanente en una base de datos SQLite (`zapateria.db`) usando el ORM SQLAlchemy, de modo que los datos no se pierden al cerrar el programa.

**¿Qué funcionalidades tiene?**

- Registrar zapatos.
- Consultar el inventario.
- Buscar zapatos por ID.
- Actualizar precios.
- Eliminar zapatos.

## 3. Estructura del proyecto

```text
tienda_zapatos/
├── main.py
├── datos.py
├── requirements.txt
├── README.md
└── evidencias/
```

**Explicación de los archivos:**

| Archivo | ¿Cuál es su responsabilidad? |
|---|---|
| `main.py` | Muestra el menú, pide los datos al usuario, los valida y llama a las funciones de `datos.py`. |
| `datos.py` | Define el motor, la clase base, el modelo `Zapato` y las funciones CRUD con SQLAlchemy. |
| `requirements.txt` | Lista la dependencia necesaria (`SQLAlchemy`). |
| `zapateria.db` | Archivo de base de datos SQLite. Se genera automáticamente la primera vez que se ejecuta el programa, cuando `inicializar_bd()` crea la tabla. |

## 4. Instalación y ejecución

**Paso 1. Requisitos**

Python 3.8 o superior y `pip`.

**Paso 2. Instalación de dependencias**

```bash
python -m pip install -r requirements.txt
```

**Paso 3. Ejecución**

```bash
python main.py
```

**Paso 4. Resultado esperado**

Aparece el menú "TIENDA DE ZAPATOS" con las opciones del 1 al 6, y se crea el archivo `zapateria.db` en la carpeta del proyecto.

---

# 5. Documentación técnica — Preguntas obligatorias

### Pregunta 1. Modelo y tabla

**Respuesta:**

La clase `Zapato` representa la tabla. Hereda de `Base`, que a su vez hereda de `DeclarativeBase`. SQLAlchemy obtiene el nombre de la tabla del atributo `__tablename__ = "zapatos"`, y las columnas de los atributos anotados con `Mapped` y `mapped_column`: el nombre del atributo es el nombre de la columna y el tipo de la anotación (`int`, `str`, `float`) define su tipo en SQLite.

**Fragmento de código relacionado:**

```python
class Zapato(Base):
    __tablename__ = "zapatos"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    modelo: Mapped[str] = mapped_column(String)
    talla: Mapped[int]
    precio: Mapped[float]
```

`id` es la clave primaria y se genera automáticamente al insertar.

### Pregunta 2. Creación de la base de datos

**Respuesta:**

Al ejecutar `main.py`, lo primero que hace `main()` es llamar a `datos.inicializar_bd()`. Esa función ejecuta `Base.metadata.create_all(engine)`, que revisa los modelos registrados en `Base` y crea la tabla `zapatos` si no existe. Como el motor apunta a `sqlite:///zapateria.db`, SQLite crea el archivo `zapateria.db` en el momento de abrir la primera conexión. Si el archivo y la tabla ya existen, no se vuelven a crear ni se borran datos.

**Fragmento de código relacionado:**

```python
engine = create_engine("sqlite:///zapateria.db")

def inicializar_bd():
    Base.metadata.create_all(engine)
```

### Pregunta 3. Registro de zapatos

**Respuesta:**

1. En el menú de `main.py` el usuario elige la opción 1 e ingresa modelo, talla y precio.
2. `main()` valida los datos y llama a `datos.crear_zapato(modelo, talla, precio)`.
3. `crear_zapato` crea un objeto `Zapato`, abre una sesión con `Session(engine)`, lo agrega con `session.add()` y confirma con `session.commit()`.
4. Al hacer `commit()`, SQLAlchemy genera el `INSERT` y SQLite guarda la fila en `zapateria.db`.
5. La función devuelve `True` y `main()` muestra "Zapato registrado correctamente."

**Fragmento de código relacionado:**

```python
def crear_zapato(modelo, talla, precio):
    zapato = Zapato(modelo=modelo, talla=talla, precio=precio)
    with Session(engine) as session:
        session.add(zapato)
        session.commit()
    return True
```

### Pregunta 4. Sesiones y transacciones

**Respuesta:**

La sesión (`Session`) es el espacio de trabajo donde SQLAlchemy sigue los objetos que se agregan, consultan, modifican o eliminan y los sincroniza con la base de datos dentro de una transacción. En este programa cada función abre su propia sesión con `with Session(engine) as session`. Si se agrega o modifica un objeto pero no se llama a `commit()`, los cambios no se guardan: al cerrar la sesión la transacción se descarta (rollback) y el registro no queda en `zapateria.db`.

### Pregunta 5. Búsqueda por ID

**Respuesta:**

`buscar_zapato(id_zapato)` usa `session.get(Zapato, id_zapato)`, que busca por clave primaria. Si encuentra el registro, devuelve el objeto `Zapato` y `main.py` lo muestra con `mostrar_zapato`. Si no existe, devuelve `None` y `main.py` imprime "Zapato no encontrado."

**Fragmento de código relacionado:**

```python
def buscar_zapato(id_zapato):
    with Session(engine) as session:
        return session.get(Zapato, id_zapato)
```

### Pregunta 6. Actualización de precios

**Respuesta:**

`actualizar_precio` obtiene el objeto con `session.get(Zapato, id_zapato)`. Si existe, se modifica su atributo (`zapato.precio = nuevo_precio`); SQLAlchemy detecta el cambio en el objeto, y al llamar a `session.commit()` genera el `UPDATE` y lo guarda. Devuelve `True`. Si el ID no existe, devuelve `False` sin cambiar nada.

**Fragmento de código relacionado:**

```python
def actualizar_precio(id_zapato, nuevo_precio):
    with Session(engine) as session:
        zapato = session.get(Zapato, id_zapato)
        if zapato is None:
            return False
        zapato.precio = nuevo_precio
        session.commit()
        return True
```

### Pregunta 7. Eliminación de registros

**Respuesta:**

`eliminar_zapato` obtiene el objeto con `session.get`, lo marca con `session.delete(zapato)` y al hacer `commit()` SQLAlchemy ejecuta el `DELETE`. Devuelve `True`; si el ID no existe devuelve `False`. Para comprobar que ya no existe, se usa la opción 2 (el zapato ya no aparece en el listado) o la opción 3 con ese ID (muestra "Zapato no encontrado.").

**Fragmento de código relacionado:**

```python
def eliminar_zapato(id_zapato):
    with Session(engine) as session:
        zapato = session.get(Zapato, id_zapato)
        if zapato is None:
            return False
        session.delete(zapato)
        session.commit()
        return True
```

### Pregunta 8. Persistencia de datos

**Respuesta:**

Los zapatos permanecen porque se guardan en el archivo `zapateria.db` en el disco, no en la memoria del programa. Al volver a ejecutar, `create_all` no borra la tabla existente y el programa se conecta al mismo archivo, por lo que `listar_zapatos()` devuelve los registros que quedaron. [Indica aquí qué zapatos viste al reabrir el programa.]

---

## 6. Pruebas de funcionamiento y evidencias

### Prueba 1. Registrar zapatos

- **Acción realizada:** registrar dos zapatos.
- **Datos utilizados:** Jordan , 38 , 300000.
- **Resultado obtenido:** Se registro el zapato.

**Evidencia:**

![Registro de zapatos](evidencias/registro.png)

### Prueba 2. Listar inventario

- **Acción realizada:** consultar todos los zapatos.
- **Resultado obtenido:** El programa mostro los zapatos disponibles.

**Evidencia:**

![Listado de zapatos](evidencias/listado.png)

### Prueba 3. Buscar por ID

- **ID utilizado:** 2.
- **Resultado obtenido:** El programa mostro el zapato que lleva el id 2 que es el zapato jordan.

**Evidencia:**

![Búsqueda de zapatos](evidencias/busqueda.png)

### Prueba 4. Actualizar precio

- **Zapato seleccionado:** Jordan.
- **Precio anterior:** 300000.
- **Precio nuevo:** 250000.
- **Resultado obtenido:** Se actualizo el precio del zapato.

**Evidencia:**

![Actualización de precio](evidencias/actualizacion.png)

### Prueba 5. Eliminar zapato

- **ID eliminado:** 1.
- **Resultado obtenido:** La verifique listando todos los zapatos y verificando que no aparezca el zapato con id 1.

**Evidencia:**

![Eliminación de zapato](evidencias/eliminacion.png)

### Prueba 6. Comprobar persistencia

- **Acción realizada:** cerrar y ejecutar nuevamente el programa.
- **Resultado obtenido:** Permanecia guardado el zapato coon id 2.

**Evidencia:**

![Persistencia de datos](evidencias/persistencia.png)

### Pruebas con registros inexistentes

| Operación | ¿Qué sucedió? |
|---|---|
| Buscar ID inexistente | `buscar_zapato` devuelve `None` y el programa muestra "Zapato no encontrado." |
| Actualizar ID inexistente | `actualizar_precio` devuelve `False` y el programa muestra "Zapato no encontrado."; no se modifica nada. |
| Eliminar ID inexistente | `eliminar_zapato` devuelve `False` y el programa muestra "Zapato no encontrado."; no se elimina nada. |

## 7. Conclusiones

1. ¿Qué aprendiste al implementar las operaciones CRUD con SQLAlchemy?

Aprendí que con SQLAlchemy se puede guardar, consultar, modificar y eliminar datos trabajando con objetos de Python. Entendí el papel del engine, de la sesión y del `commit()`, y que sin confirmar la transacción los cambios no se guardan en la base de datos.

2. ¿Qué diferencia identificaste entre trabajar con objetos Python y escribir SQL directamente?

Con el ORM trabajo con la clase `Zapato` y sus atributos, y SQLAlchemy genera el SQL por mí. Con SQL directo tendría que escribir cada sentencia (`INSERT`, `SELECT`, `UPDATE`, `DELETE`) a mano y convertir los resultados en datos de Python. El ORM hace el código más corto y legible, y evita errores al armar las consultas.

3. [¿Cuál fue la principal dificultad y cómo la solucionaste?]

La mayor dificultad fue entender el manejo de la sesión y la transacción. Al principio no tenía claro por qué un cambio no quedaba guardado si no usaba `commit()`. Lo solucioné probando el programa: cerré y volví a abrir la aplicación, y comprobé que los datos solo permanecían cuando se confirmaba la transacción. También tuve que respetar los nombres de las funciones y lo que debían devolver (`True`, `False` o `None`), porque `main.py` depende de esos valores para mostrar los mensajes.