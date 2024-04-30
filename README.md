# Unidad 6 - Tarea 6.2

## A - Investiga que son los procedimientos y las funciones en postgreSQL, intenta crear una tarea simple como crear un tabla que se llame prueba_ud6.

Los procedimiento o las funciones son "bloques" de código los cuales pueden ser reutilizados las veces que se requieran sin tener que repetir el mismo código cada vez. Una de las ventajas de utilizar procedimientos o funciones es que a la hora de realizar llamadas para obtener datos se ve mucho más limpio y ordenado todo el código.

Dividiendo conceptos, las *funciones* y *procedimientos* se podrían explicar **independientemente**. Las **funciones** son por así decrilo la parte que más va a **interactuar** con la **base de datos**. Esta se encargará de realizar las consultas y los cálculos necesarios para devolver el resultado deseado.

En cambio, los **procedimientos** son por así decirlo la parte más **"amigable"** con el usuario. Esta interactúa con las funciones, que a su vez interactuan con la base de datos. En un procedimiento puedes usar las funciones que quieras, a parte puedes utilizar los datos que te den para realizar controles, cálculos, etc.

Todo esto de los procedimientos y las funciones es gracias al lenguaje de procedimientos que estamos dando en este tema: **`PL/pgSQL`**. Hay muchos lenguajes de procedimientos, uno muy similar es: **``PL/SQL``**, este en cambio es usado por Oracle.

Las características fuertes de este lenguaje son:
  1. Realizar cálculos complejos.
  2. Fácil de usar. (entre comillas "").
  3. Estructuras de control para el lenguaje SQL.


### Estructura de una función simple:

Lenguaje: **`PL/pgSQL`**

```postgresql
CREATE OR REPLACE
PROCEDURE add_employer(name TEXT, las_name TEXT) 
AS $$
    BEGIN
        INSERT INTO employer (name, last_name)
        VALUES (name, last_name);
    END;
    $$ LANGUAGE plpgsql;
```

*Copiado de un ejemplo de amazon.*

Esta consulta permite añadir empleados, estos tendrán el nombre y apellidos que se les pase como parámetro.


### Estructura de un procedimiento simple:

```postgresql
DO $$
    DECLARE
        nombre_empleado TEXT := 'John';
        apellido_empleado TEXT := 'Doe';
    BEGIN
        PERFORM add_employer(nombre_empleado, apellido_empleado);
        RAISE NOTICE '¡Empleado agregado correctamente!';
    END;
    $$;
```

#### ***Nota***
*Se está usando **`PERFORM`** porque en este caso no se necesita obtener los resultados, simplemente los añade y ya. Básicamente llama a la función sin esperar nada a cambio, como si en Java llamas a un método de tipo ***`void`***.*

Este procedimiento está creando un nombre y apellidos ya que no hay una tabla para ello, pero si la hubiera, se podría hacer un `SELECT` para obtener los datos de la tabla y luego añadirlos a la tabla de empleados. Después de eso está llamando a la función `add_employer` para añadir el empleado a la tabla de empleados con los atributos que se le están pasando.

---

### Tarea simple: Crear una tabla llamada `prueba_ud6`

#### Procedimiento:

```postgresql
CREATE OR REPLACE
PROCEDURE crear_tabla_prueba_ud6()
AS $$
    BEGIN
        CREATE TABLE prueba_ud6 (
            id SERIAL PRIMARY KEY,
            name TEXT NOT NULL
        );
    END;
    $$ LANGUAGE plpgsql;
```

#### ***Nota***
***`SERIAL`** es un tipo de dato que se autoincrementa, es decir, que cada vez que se inserte un dato en la tabla se va a autoincrementar el valor de la columna `id`.*

#### Llamada al procedimiento:

```postgresql
DO $$
    BEGIN
        PERFORM crear_tabla_prueba_ud6();
        RAISE NOTICE 'La tabla ha sido creada correctamente';
    END;
    $$;
```

#### ***Nota***
*Se está usando otra vez **`PERFORM`** ya que no necesitamos que nos devuelva un valor la función, simplemente tiene que crear la tabla y ya.*
<br><br>


## B - A partir de la funciona anterior intenta crear ahora una función que genera una tabla recibiendo un nombre como parámetro.

### Procedimiento modificado:

```postgresql
CREATE OR REPLACE
PROCEDURE crear_tabla_personalizada(nombre_tabla TEXT)
 AS $$
    BEGIN
        EXECUTE format('CREATE TABLE %I (id SERIAL PRIMARY KEY, name TEXT NOT NULL)', nombre_tabla);
    END;
    $$ LANGUAGE plpgsql;
```

#### Notas
  1. El **`EXECUTE`** en este caso es una buena práctica porque esta consulta es dinámica. Al poder variar el resultado según el parámetro que le pase el usuario es mejor. También estoy usando esto porque no puedes ponerle **nombres dinámicos** a **tablas** y/o **columnas**, así que se usa el **`format()`**.
  2. El **`format()`** va de la mano del **`EXECUTE`**. El formateador está utilizando el **`%I`**, que este se está interpretando como el valor que hay al final de toda la cadena: **`'...', nombre_tabla)`**. El resultado final sería la consulta con el nombre de tabla que se le pase como parámetro gracias a **`EXECUTE`** y **`format()`**.

```postgresql
DO $$
    BEGIN
        PERFORM crear_tabla_personalizada('Profesores');
        PERFORM crear_tabla_personalizada('Alumnos');
        RAISE NOTICE 'Las tablas han sido creadas correctamente';
    END;
    $$;
```
<br>


## C - ¿Qué diferencias crees que hay entre funciones y procedimientos?

La diferencia principal es que un procedimiento no debe retornar ningún valor y una función sí que debe retornar un valor. [Foro](https://www.postgresql.org/message-id/f7f6b4c70710300558s373d455dh511391a72018851@mail.gmail.com#:~:text=Muchachos%2C,y%20otro%20asunto%20%3A).

Pero hay un problema, estamos usando **PostgreSQL** y este usa el lenguaje **`PL/pgSQL`**. En este lenguaje **no** hay diferencia entre funciones y procedimientos, **son lo mismo** y tanto funciones como procedimientos son funciones en este lenguaje. En otros lenguajes de procedimientos como por ejemplo en Oracle, sí que hay diferencia entre una función y un procedimiento. 
<br><br>


## D - Realiza los siguientes procedimientos (sin ejecutarlos):

### 1. Un procedimiento que devuelva el stock de un determinado artículo.

```postgresql
CREATE OR REPLACE
PROCEDURE stock_articulo(articulo TEXT)
AS $$
    DECLARE
        stock INT;
    BEGIN
        SELECT stock INTO stock
        FROM articulos
        WHERE nombre = articulo;
        RETURN stock;
    END;
    $$ LANGUAGE plpgsql;
```

```postgresql
DO $$
    BEGIN
        PERFORM stock_articulo('Camiseta');
        RAISE NOTICE 'El stock de la camiseta es: %', stock_articulo;
    END;
    $$;
```


### 2. Un procedimiento que devuelva el total facturado entre 2 fechas.

```postgresql
CREATE OR REPLACE
PROCEDURE total_facturado(fecha1 DATE, fecha2 DATE)
AS $$
    DECLARE
        total_facturado NUMERIC;
    BEGIN
        SELECT SUM(total) INTO total_facturado
        FROM facturas
        WHERE fecha BETWEEN fecha1 AND fecha2;
        RETURN total_facturado;
    END;
    $$ LANGUAGE plpgsql;
```

```postgresql
DO $$
    BEGIN
        PERFORM total_facturado('2021-01-01', '2021-12-31');
        RAISE NOTICE 'El total facturado entre las fechas 2021-01-01 y 2021-12-31 es: %', total_facturado;
    END;
    $$;
```


### 3. Un procedimiento de tu invención que creas que te podría venir bien para futuros usos.

```postgresql
CREATE OR REPLACE
PROCEDURE mostrar_tabla(tabla TEXT)
AS $$
    BEGIN
        EXECUTE format('SELECT * FROM %I', tabla);
    END;
    $$ LANGUAGE plpgsql;
```

```postgresql
DO $$
    BEGIN
        PERFORM mostrar_tabla('empleados');
        RAISE NOTICE 'La tabla de empleados se ha mostrado correctamente';
    END;
    $$;
```
