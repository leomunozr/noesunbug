---
layout: post
title: Encontrando el sentido del scope
excerpt: Si crees que el scope de Javascript no tiene sentido, debes leer esta breve introducción a cómo usarlo.
category: javascript
---

Si vienes de un lenguaje de programación como Java, C o PHP, Javascript podría parecer que no tiene sentido. En especial cuando del scope se trata, pues Javascript tiene algunas reglas que los demás lenguajes no. Podrías evitar aprenderlas y seguir usando el lenguaje a ciegas, pero tarde o temprano estas características vendrán a perseguirte. Por eso, he aquí una breve introducción a cómo usar el scope en Javascript.

#### ¿Qué es scope?
El _ámbito_ - o _scope_, en inglés - es la zona en el código en donde una variable o el identificador de una función (o sea, su nombre) puede ser usada. Fuera de esta zona la variable es inaccesible, como si nunca hubiera existido. El nombre técnico para esto es _visibilidad_, una variable es visible si podemos usarla para leerla o guardar valores en ella.

#### Pero, ¿para qué sirve?
Sin duda sería más fácil que todas las variables estuvieran disponibles en todas partes del código. Eso nos evitaría el trabajo de verificar si la variable existe o no en cierto ámbito, (recuerdo la mala costumbre que tenía de hacer globales a todas las variables cuando recién iniciaba a programar en C). 

Pero el ámbito de las variables tiene su razón de ser. Si todas las variables fueran visibles en todos lados, ¿qué impediría a una función modificar por accidente variables que no se suponía debía tocar?,  ¿qué sucedería si declarara una variable con un nombre que ya fue previamente usado, por ejemplo al usar una librería externa cuyo código desconocemos?. Estos son los problemas que intenta solucionar el ámbito.

#### Scope global vs scope local
En Javascrip, y en la mayoría de lenguajes de programación populares, existen dos tipos de ámbito, el _global_ y el _local_. 
Al iniciar un programa de Javascript, ya nos encontramos en el _ámbito global_. Y todas las variables (también aplica para la definición de las funciones) que sean declaradas aquí pertenecen al _ámbito global_ - o lo que es lo mismo, son _variables globales_.

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

> Las variables _mensaje_ y _res_, así como el identificador _suma_ se encuentran en el _ámbito global_ porque no están contenidos dentro de una función. Por otro lado, la definición de las variables _a_ y _b_ forman parte de la función _suma_ por lo que pertenecen a su _ámbito local_

#### Block scope
A diferencia de otros lenguajes, **en Javascript el ámbito está basado en funciones**. Cada función crea un ámbito separado para ella, pero los demás tipos de bloques (_if_, _for_, _switch_, bloques explícitos usando _{}_, etc.) no generan un nuevo ámbito. 

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

> Las variables _a_, _b_ y _c_ pertenecen al _ámbito local_ de la función _foo_. _C_ puede ser incluso accedida fuera del bloque _if_ porque los bloques _if_ no generan un ámbito distinto.
> La variable _i_ también puede ser accedida fuera del ciclo _for_, ya que tampoco crean un nuevo ámbito.

#### En el scope hay niveles

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

> Al ejecutar la función _interior_, el intérprete busca dónde fueron definidas _foo_ y _bar_, iniciando desde la función más interna hasta la más externa. Foo se encuentra en el _scope 2_ ('Hola'), no hay necesidad de seguir buscando. _bar_ no se encuentra en el scope 2, subimos un nivel, encontramos una declaración _bar = 'a todos'_ y esa usamos. La declaración global _bar = 'a nadie'_ nunca fue usada.

#### Variables sin declaración

Una peculiaridad de Javascript es que, si intentamos asignar un valor a una variable que no ha sido definida dos cosas pueden suceder: si el programa en Javascript se ejecutó utilizando el modo _“use strict”_, un _ReferenceError_ será arrojado, indicando que la variable no ha sido definida. Pero si el programa no está en modo estricto, se creará automáticamente una variable global y en ella será asignado el valor.

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

> Siempre es mejor agregar 'use strict' para generar mensajes de error y evitar errores indeseados.
