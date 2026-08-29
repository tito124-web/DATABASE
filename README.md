# Clase 5 - CRUD con JDBC, sin Maven

Ejemplo de un **CRUD completo** (Crear, Leer, Actualizar, Eliminar) contra MySQL desde
Java, **sin usar ninguna herramienta de build** (ni Maven ni Gradle). Todo se hace a
mano, para que se vea con claridad que problema resuelve Maven en el proyecto hermano
`clase05-jdbc-con-maven`.

## Estructura del proyecto

```
src/edu/umg/programacion2/clase05/
├── Main.java              -> menu de consola (entrada/salida con el usuario)
├── modelo/Estudiante.java -> clase de dominio (solo datos + encapsulamiento)
└── dao/EstudianteDAO.java -> TODO el codigo SQL/JDBC vive aqui
```

`Main` nunca escribe SQL directamente: solo llama metodos de `EstudianteDAO` como
`crear(...)`, `listarTodos()`, `buscarPorCarnet(...)`, `actualizarNombre(...)` y
`eliminar(...)`. Esta separacion (interfaz de usuario vs. acceso a datos) es una
version simple del **DAO pattern** que van a formalizar mas adelante en el curso.

## Que es un driver JDBC (y por que lo necesitamos)

Java no sabe hablar el protocolo de MySQL de fabrica. JDBC (Java Database
Connectivity) es la API estandar que Java define para conectarse a bases de datos, pero
cada motor (MySQL, PostgreSQL, SQL Server, etc.) necesita su propia implementacion de
esa API: el **driver**. Sin el driver correcto en el classpath, `DriverManager` no tiene
como abrir la conexion.

En este proyecto el driver ya esta descargado en `lib/mysql-connector-j-8.0.33.jar`.
Si tuvieras que conseguirlo vos mismo (por ejemplo, en tu propia laptop): se descarga
desde [Maven Central](https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/) o
desde el sitio oficial de MySQL, y se coloca en una carpeta como `lib/` dentro del
proyecto.

## Requisitos

- JDK 11 instalado (`java -version`).
- MySQL 8 corriendo en `localhost:3306` (el que instalaste para la tarea de la Clase 3).

## Paso 1: crear la base de datos

Ejecuta `sql/schema.sql` en MySQL Workbench o desde la consola:

```bash
mysql -u root -p < sql/schema.sql
```

## Paso 2: configurar la conexion

Abre `src/edu/umg/programacion2/clase05/Main.java` y ajusta `USUARIO` y `PASSWORD` con
tus credenciales reales de MySQL.

## Paso 3: compilar

El flag `-cp` (classpath) le dice a `javac` donde buscar clases adicionales ademas de
tu propio codigo fuente; aqui le apuntamos al .jar del driver. Como ahora son varios
archivos `.java` (Main, Estudiante, EstudianteDAO), usamos `find` para pasarlos todos
de una vez:

```bash
javac -cp "lib/mysql-connector-j-8.0.33.jar" -d out $(find src -name "*.java")
```

## Paso 4: ejecutar

El separador de rutas en `-cp` es `:` en Mac/Linux y `;` en Windows:

```bash
# Mac / Linux
java -cp "out:lib/mysql-connector-j-8.0.33.jar" edu.umg.programacion2.clase05.Main

# Windows
java -cp "out;lib/mysql-connector-j-8.0.33.jar" edu.umg.programacion2.clase05.Main
```

## Salida esperada

```
=== CRUD de Estudiantes (MySQL) ===
1. Agregar estudiante
2. Listar todos los estudiantes
3. Buscar estudiante por carnet
4. Actualizar nombre de un estudiante
5. Eliminar estudiante
6. Salir
Elige una opcion: 2
[1] Ana Lopez - carnet 2024001
[2] Carlos Perez - carnet 2024002
[3] Maria Gonzalez - carnet 2024003
```

## Nota sobre `Class.forName(...)`

En tutoriales viejos vas a ver una linea como
`Class.forName("com.mysql.cj.jdbc.Driver")` antes de conectar. Desde JDBC 4.0 (Java 6
en adelante) ya no hace falta: el driver se registra solo con `DriverManager` gracias a
un mecanismo llamado *Service Provider Interface* (el .jar incluye un archivo que le
avisa a Java "yo soy un driver JDBC"). Si ves ese codigo en internet, ahora sabes por
que ya no es necesario escribirlo.

## Errores comunes

```
# Error: no suitable driver found for jdbc:mysql://...
```
Significa que el .jar del driver no esta en el classpath al ejecutar. Revisa el `-cp`
del paso 4.

```
# Error: Access denied for user 'root'@'localhost'
```
El usuario o password en `Main.java` no coinciden con los de tu instalacion de MySQL.

## Para comparar

Abre el proyecto `clase05-jdbc-con-maven` y compara el `pom.xml` con el `-cp` de aca:
Maven no hace magia, solo automatiza descargar el .jar correcto y armar el classpath
por vos.

## Ejercicio propuesto

Agrega una validacion en `agregarEstudiante()`: si el nombre o el carnet vienen
vacios, muestra un mensaje de error y no llames a `estudianteDAO.crear(...)`
(pista: `String.isBlank()`).
