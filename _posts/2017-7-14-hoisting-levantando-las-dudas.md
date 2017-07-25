---
layout: post
title: <em>Hoisting</em>. Levantando las dudas
excerpt: Una más de las curiosidades del intérprete de JS. ¿Qué es <em>hoisting</em>? ¿Cómo afecta a las variables? ¿Qué reglas sigue?
author: Leonardo Muñoz
category: javascript
---

> _hoist_: **to lift something heavy, sometimes using ropes or a machine.**  
> **levantar, alzar, izar.**  
> She hoisted the sack onto her back. (Subió el saco sobre su espalda).  
> He hoisted the child up on to his shoulders. (Alzó al niño en sus hombros).  
> They hoisted the flag. (Izaron la bandera).  
>
> -- <cite>_Diccionario Cambridge_</cite>

Una palabra curiosa para una característica aún más curiosa. Según el diccionario, _hoist_ se puede traducir como alzar o izar.  
¿Qué tiene eso que ver con Javascript? Pues bien, el intérprete de Javascript hace otra vez de las suyas y trata a las variables de una forma peculiar que, si desconocemos, podría jugarnos una mala pasada.

Para definir qué es _hoisting_, un ejemplo. ¿Pueden adivinar el resultado del siguiente código?

```javascript
x = 10;
var x;
console.log('x = ' + x);
```

Este código no arrojará ningún error. Tampoco mostrará un valor `undefined`. Contra toda lógica, este código es legal y mostrará un valor de: `x = 10`.

## Hoisting en variables

Sucede que en Javascript, el código pasa por un proceso de compilación que busca optimizar el tiempo de ejecución. Durante este proceso, todas las declaraciones de variables y funciones son puestas en memoria para tenerlas listas y acceder a ellas de forma más rápida durante el proceso de interpretación. Como resultado, podemos imaginar que **en el proceso de compilación las declaraciones de variables y funciones son _alzadas_, _izadas_ o _subidas_ (_hoisted_) hacia el inicio del [scope](/encontrando-el-sentido-del-scope)**.

El código anterior sería entonces equivalente a:
```javascript
var x; // efecto del hoisting
x = 10;
console.log('x = ' + x); // x = 10
```

## Declaraciones vs. asignaciones

Volvamos a tomar una suposición más educada sobre el resultado del siguiente código:

```javascript
console.log('x = ' + x);
var x = 10;
```

Si supusieron que el resultado era `x = 10` porque el efecto del _hoisting_ subiría la línea `var x = 10` arriba de `console.log('x = ' + x)`, temo decirles que no es así. Lo que se mostrará es `x = undefined`.

Aunque esté escrito en una sola línea, para el compilador el código anterior es equivalente a:

```javascript
console.log('x = ' + x);
var x;
x = 10;
```

La sentencia `var x = 10` en realidad corresponde a dos acciones: una declaración de variable y su correspondiente asignación de un valor.  

Ya divididas las instrucciones en dos líneas es más fácil ver por qué el valor es `undefined` al imprimirlo. Las declaraciones de variables (y funciones) son atendidas durante el tiempo de compilación, mientras que las asignaciones lo son durante el tiempo de interpretación. Y debido a eso es que **sólo las declaraciones sufren de _hoisting_, las asignaciones no**.

Clarificado eso, el código equivalente, después del efecto de hoisting será:

```javascript
var x;
console.log(x);
x = 10;
```

Ahora sí resulta evidente por qué el valor `undefined`.

## ¿Y qué hay de las funciones?

Hasta ahora sólo hemos hablado de variables, pero el efecto de _hoisting_ también afecta a la declaración de las fuciones.

```javascript
foo(); // hola

foo() {
  console.log('hola');
}
```

De modo que podemos invocar una función incluso antes de que haya sido declarada. Aquí la declaración `foo` (con todo y el cuerpo de la función) será _"subida"_ al inicio del _scope_, haciendo legal este código.

```javascript
foo() { // La declaración sube por efecto del hoisting
  console.log('hola');
}

foo(); // hola
```

### ReferenceError o TypeError

Hagamos un paréntesis para hablar de dos tipos especiales de error que tiene Javascript: `ReferenceError` y `TypeError`.  

#### ReferenceError

Un error de tipo `ReferenceError` ocurre cuando se intenta **acceder a un identificador (nombre de variable o función) que no ha sido declarado**.

```javascript
var x = y + 10; // ReferenceError: y no ha sido definida

persona.nombre = 'Juanito'; // ReferenceError: persona no ha sido definida

funcionQueNoExiste(); // ReferenceError: funcionQueNoExiste no ha sido definida
```

#### TypeError

Un error de tipo `TypeError` ocurre cuando **un valor se usa de una forma no esperada**. Esto puede ser, al usar un operador con operandos incompatibles, al intentar invocar con parámetros algo que no es una función, o al intentar acceder a una propiedad sobre un valor `null` o `undefined`.

```javascript
var num = 5;
num(); // TypeError: num no es una función

var persona;
persona.nombre = 'Juanito'; // TypeError: No se puede asignar la propiedad 'nombre' de undefined
// El error cambió de un ReferenceError a un TypeError
// porque la variable persona sí existe esta vez,
// pero guarda un valor undefined
// que no posee propiedades

var vacio = null;
vacio.x = 1; // TypeError: No se puede asignar la propiedad 'x' de null.
```

Conociendo estos errores, podemos deducir la causa de una función mal invocada.  
Si se intenta invocar una función que no ha sido definida, el intérprete arrojará un `ReferenceError`. Si por otro lado el identificador sí existe, pero su valor no corresponde a una función, arrojará un `TypeError`.

### Declaraciones vs. expresiones

Hagamos otro paréntesis para explicar algunas diferencias al definir funciones.  
Una función puede ser definida mediante la palabra reservada `function`, el nombre de la función (usualmente, pero puede ser omitido en ciertas ocasiones), y el cuerpo de ésta:

```javascript
function foo() {
  return 10;
}
```
A esto le llamamos una declaración de función. 

Pero debido a que en Javascript todo (bueno, casi todo) es un objeto, incluyendo las funciones, éstas también se pueden asignar como el valor de una variable:

```javascript
  var foo = function() {
    return 10;
  }
``` 
A esto se le llama una expresión de función.

### _Hoisting_ o no _hoisting_

Con estos antecedentes estamos listos para la siguiente regla: **las declaraciones de funciones son afectadas por _hoisting_, pero las expresiones de funciones no**.  
Esto es análogo a lo que ya habíamos analizado con la declaración y asignación de variables: las declaraciones suben al inicio del scope, mientras que la asignación de valores no lo hacen.

De tal modo que podemos hacer esto:

```javascript
foo(); // hola

foo() { // Hoisting en la declaración
  console.log('hola');
}
```

Y por efecto del _hoisting_ la declaración subirá al inicio del _scope_.  

Pero no funcionará igual de esta otra forma:

```javascript
foo(); // TypeError: foo no es una función

var foo = function() { // No hay hoisting en las expresiones
  console.log('hola');
}
```

En el segundo código, en lugar de mostrar `hola`, lo que obtuvimos es un `TypeError`. Recordemos que este error indica que el identificador `foo` existe, pero no es una función.
¿Por qué aquí no obtuvimos un `hola` como en el caso de la declaración? Porque la expresión de la función fue dividida por el compilador en declaración y asignación: 

```javascript
foo();

var foo; // Declaración del identificador foo
foo = function() { // Asignación de foo a la función
  console.log('hola');
}
```

Y posteriormente interpretada (ocurrió _hoisting_) como:

```javascript
var foo;
foo(); // TypeError: foo existe, pero su valor es undefined

foo = function() {
  console.log('hola');
}
```

Por el tipo de error `TypeError` concluimos que `foo` existe antes de invocarlo, pero con el valor `undefined`. (Sin el efecto de _hoisting_, en lugar de un `TypeError` hubiéramos tenido un `ReferenceError`, ya que `foo` no existiría en el momento de invocarlo.)

## ¿Quién va primero?

Finalmente, ¿qué ocurrirá en el siguiente código, mostrará `'hey'`, `'ho'`, `ReferenceError` o `TypeError`?

```javascript
foo();

var foo = function() {
  console.log('hey');
}

function foo() {
  console.log('ho');
}
```

La respuesta correcta es `'ho'`. ¿La razón?. **Las declaraciones de funciones toman prioridad sobre las declaraciones de variables para subir (_hoist_) al inicio del _scope_**. 

Para el compilador el equivalente del código anterior sería:

```javascript
function foo() {
  console.log('ho');
}
// var foo; // Esta declaración es omitida,
//pues foo ya fue definido 

foo();

foo = function() {
  console.log('hey');
}
```

La definición de la función `foo` sube primero al inicio del _scope_. Después vendría la declaración de `foo` como variable. Pero en Javascript, **si un identificador ya fue declarado, no se vuelve a redeclarar** (esto no significa que el identificador no pueda ser asignado con nuevos valores las veces que sea), entonces esta declaración desaparece. Cuando se invoca `foo()`, imprime el valor `'ho'`. Posteriormente se redefine a `foo` para imprimir `'hey'`, pero esa función ya no llega a invocarse de nuevo.

## Conclusión

Si les da vueltas la cabeza después de tantas relgas y preguntas capciosas, no hay que temer. Aquí el resumen de las reglas de _hoisting_:

- Las declaraciones de variables y funciones siempre son movidas hacia el inicio del scope. A esto se llama _hoisting_.
- El orden importa, las funciones toman prioridad sobre las variables. Si existe una declaración de una función y una declaración de una variable con el mismo nombre, la función es la que sube al inicio del scope y gana la definición.
- Las asignaciones de valor hacia una variable, o bien las expresiones de funciones no sufren de _hoisting_, es decir, permanecen en su lugar correspondiente en el código.

Tan simple como eso. Ahora, ¿podrán decir qué regresan estos códigos y por qué?

```javascript
function codigo1() {

  function foo() {
    return 'hola';
  }

  return foo();

  function foo() {
    return 'adios';
  }
}
codigo1();
```

```javascript
function codigo2() {

  var foo = function() {
    return 'hola';
  }

  return foo();
  
  var foo = function() {
    return 'adios';
  }
}
codigo2();
```
