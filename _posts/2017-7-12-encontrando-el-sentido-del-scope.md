---
layout: post
title: Encontrando el sentido del <em>scope</em>
excerpt: Si crees que el <em>scope</em> de Javascript no tiene sentido, debes leer esta breve introducción a cómo usarlo.
author: Leonardo Muñoz
category: javascript
---

Si vienes de un lenguaje de programación como Java, C o PHP, Javascript podría parecer que no tiene sentido. En especial cuando del scope se trata, pues Javascript tiene algunas reglas que los demás lenguajes no. Podrías evitar aprenderlas y seguir usando el lenguaje a ciegas, pero tarde o temprano estas características vendrán a perseguirte. Por eso, he aquí una breve introducción a cómo usar el scope en Javascript.

### ¿Qué es scope?
El _ámbito_ -o _scope_, en inglés- es la zona en el código en donde una variable o el identificador de una función (o sea, su nombre) puede ser usada. Fuera de esta zona la variable es inaccesible, como si nunca hubiera existido. El nombre técnico para esto es _visibilidad_, una variable es visible si podemos usarla para leerla o guardar valores en ella.

### Pero, ¿para qué sirve?
Sin duda sería más fácil que todas las variables estuvieran disponibles en todas partes del código. Eso nos evitaría el trabajo de verificar si la variable existe o no en cierto ámbito, (recuerdo la mala costumbre que tenía de hacer globales a todas las variables cuando recién iniciaba a programar en C). 

Pero el ámbito de las variables tiene su razón de ser. Si todas las variables fueran visibles en todos lados, ¿qué impediría a una función modificar por accidente variables que no se suponía debía tocar?,  ¿qué sucedería si declarara una variable con un nombre que ya fue previamente usado, por ejemplo al usar una librería externa cuyo código desconocemos?. Estos son los problemas que intenta solucionar el ámbito.

### Scope global vs scope local
En Javascrip, y en la mayoría de lenguajes de programación populares, existen dos tipos de ámbito, el _global_ y el _local_. 
Al iniciar un programa de Javascript, ya nos encontramos en el _ámbito global_. Y todas las variables (también aplica para la definición de las funciones) que sean declaradas aquí pertenecen al _ámbito global_ -o lo que es lo mismo, son _variables globales_.

```javascript
// scope global
var saludo = 'hola';
```

Al encerrar una parte de código dentro de una función se crea un ámbito nuevo. Todos los identificadores (variables y funciones por igual) que estén dentro de esa función sólo serán visibles desde el interior. Se dice entonces que las variables pertenecen al _ámbito local_ de la función.

```javascript
function saludar() {
  var saludo = 'hola'; // scope local
}
console.log(saludo); // Uncaught ReferenceError: saludo is not defined
```

La diferencia entre ambos tipos de ámbitos es que las **variables globales pueden ser usadas (y alteradas) en cualquier parte del código**, mientras que las **variables locales sólo están disponibles dentro de la función en la que fueron creadas**. 
Es necesario conocer la diferencia entre ambos y saber cuándo utilizarlos. Es muy cómodo utilizar variables globales, ya que están disponibles en toda la aplicación. Pero un abuso de variables globales puede provocar modificaciones indeseadas, conflictos de nombres entre variables, y dificulta debuggear la aplicación.

```javascript
function suma(a, b) {
  return a + b; // scope local
}

var res = suma(3 + 5)
var mensaje; // scope global

if(res > 10) {
  mensaje = 'Mayor a 10'; // mensaje sigue siendo parte del scope global
} else {
  mensaje = 'Menor o igual a 10'; // scope global
}

console.log(mensaje);
```

> Las variables `mensaje` y `res`, así como el identificador `suma` se encuentran en el _ámbito global_ porque no están contenidos dentro de una función. Por otro lado, la definición de las variables `a` y `b` forman parte de la función `suma` por lo que pertenecen a su _ámbito local_

### Block scope
A diferencia de otros lenguajes, **en Javascript el ámbito está basado en funciones**. Cada función crea un ámbito separado para ella, pero los demás tipos de bloques (`if`, `for`, `switch`, bloques explícitos usando `{}`, etc.) no generan un nuevo ámbito. 

```javascript
// scope global
var foo = function(a) {
  // scope local
  var b = true;
  if(b) {
    // mismo scope local
    var c = 'local';
  }
  console.log(c); // 'local'
}

for (var i = 0; i < 5; i++) {
  // sigue siendo scope global
}
console.log(i) // 5
```

> Las variables `a`, `b` y `c` pertenecen al _ámbito local_ de la función `foo`. `c` puede ser incluso accedida fuera del bloque `if` porque los bloques `if` no generan un ámbito distinto.
> La variable `i` también puede ser accedida fuera del ciclo `for`, ya que tampoco crean un nuevo ámbito.

### En el scope hay niveles

También puede ocurrir que una función contenga en su interior la declaración de otra función. En este caso habrá dos ámbitos distintos, el que corresponde a la función exterior y el de la función anidada. **Las variables creadas dentro de cada función son independientes entre sí**.

```javascript
// scope global
var a = 1

function exterior() { // scope 1
  var b = 2;

  function interior() { // scope 2
    var c = 3;
  }
}
```

Cada vez que accedemos a una variable -ya sea para lectura o escritura-, el intérprete de Javascript empieza buscando su declaración a partir de la función en la que se accedió. Si la encuentra ahí, entonces utiliza esa declaración. Si no la encuentra, sigue la búsqueda accediendo al ámbito superior, ya sea que se trate de una función exterior o que haya llegado al ámbito global. La búsqueda por la definición de la variable termina con la primera ocurrencia que el intérprete encuentre. De modo que es válido que existan dos variables con el mismo nombre en diferentes niveles y su valor dependerá del ámbito en el que sea referenciada.

```javascript
var bar = 'a nadie'; // scope global

function exterior() { // scope 1

  var bar = 'a todos';
  
  function interior() { // scope 2

    var foo = 'Hola ';    
    console.log(foo + bar); // busca la variable bar y la encuentra en el scope 1
  }

  interior();
}

exterior(); // 'Hola a todos'
```

> Al ejecutar la función `interior`, el intérprete busca dónde fueron definidas `foo` y `bar`, iniciando desde la función más interna hasta la más externa. `foo` se encuentra en el _scope 2_ (`'Hola'`); no hay necesidad de seguir buscando. `bar` no se encuentra en el _scope 2_, subimos un nivel; encontramos una declaración `bar = 'a todos'` y esa usamos. La declaración global `bar = 'a nadie'` nunca fue usada.

### Variables sin declaración

Una peculiaridad de Javascript es que, si intentamos asignar un valor a una variable que no ha sido definida dos cosas pueden suceder: si el programa en Javascript se ejecutó utilizando el modo `'use strict'`, un `ReferenceError` será arrojado, indicando que la variable no ha sido definida. Pero si el programa no está en modo estricto, se creará automáticamente una variable global y en ella será asignado el valor.

```javascript
function foo() {
  mensaje = 'Soy global';
}

foo();
console.log(mensaje) // 'Soy global'
```

Aunque el código anterior sea legal, se considera mala práctica y debe evitarse, pues hace confuso el programa y puede sobreescribir un valor sin desearlo. 

```javascript
'use strict';

function foo() {
  mensaje = 'Soy global';
}

foo(); // Uncaught ReferenceError: mensaje is not defined
console.log(mensaje)
```

> Siempre es mejor agregar `'use strict'` para generar mensajes de error y evitar errores indeseados.
