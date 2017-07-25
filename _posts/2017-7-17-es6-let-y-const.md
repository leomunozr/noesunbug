---
layout: post
title: ES6&#58; <em>let</em> y <em>const</em>
excerpt: Una revisión a las nuevas formas para crear variables y constantes. Además de sus similitudes y diferencias con <em>var</em>.
author: Leonardo Muñoz
category: [javascript, es6]
---

Una de las nuevas características que introduce ES6, y en mi opinión, de las más trascendentales, es la creación de variables con `let` y `const`.  

En _ES5_, las variables creadas por `var` seguían algunas reglas oscuras que no eran del todo conocidas por todos, me refiero al [_scope_ por función](/encontrando-el-sentido-del-scope) y al [efecto de _hoisting_](/hoisting-levantando-las-dudas).  
Si no estás familiarizado con estos conceptos, deberías revisar ambos links para conocer un poco más de ellos. De cualquier forma, intentaré hacer una breve introducción para recordar de qué tratan ambas características.

## Nuevo _scope_ por bloque

A diferencia de otros lenguajes de programación populares (C, C#, Java, etc.), en Javascript, el _ámbito_ de las variables se determina a nivel de función. Esto es, cada función crea un _ámbito_ diferente en el que las variables pueden vivir.

Ahora, en _ES6_, la definición de variables con `let` introduce a Javascript el _ámbito a nivel de bloque_. Con esto, **el ámbito de las variables está delimitado por llaves `{...}`**; ya sea dentro de una función: `function foo(){...}`, una estructura de control: `for(...){...}` o un simple bloque explícito: `{...}`. Una variable definida con `let` es visible sólo en el bloque en el que fue definido y, además, dentro de cualquier otro bloque anidado a ese ámbito.

Las declaraciones hechas con `var` siguen conservando el _ámbito por función_.

```javascript
if (true) { 
  var foo = 'función';
  let bar = 'bloque';

  console.log(foo); // función
  console.log(bar); // bloque
}

console.log(foo); // función
console.log(bar); // ReferenceError: bar no está definido
```

La variable `bar` -creada con `let`- fue definida dentro de un bloque `if`, por lo que sólo será visible dentro de él. A diferencia de `foo` -creada con `var`-, que tiene como ámbito la función entera.

## ¿Qué ocurre con el _hoisting_?

[_Hoisting_](/hoisting-levantando-las-dudas) es el nombre que se le da a una optimización de código hecho por el intérprete de Javascript. Podemos imaginarlo como si el intérprete _moviera_ de lugar las declaraciones de variables (`var foo`) y las posicionara justo al inicio del scope.

Otra característica de `let` es que estas variables **no serán sujetas a [_hoisting_](/hoisting-levantando-las-dudas)**. En vez de ser cambiadas de lugar hacia el inicio del _scope_, las declaraciones con `let` permanecen en su lugar y **no pueden ser referenciadas hasta que se haya ejecutado su declaración**. Violar la regla anterior causará un `ReferenceError`.

A diferencia de `let`, las variables creadas con `var` sí permiten referenciarlas antes de su declaración con un valor `undefined`.

```javascript
console.log(foo); // undefined
console.log(bar); // ReferenceError

var foo = 'OK';
let bar = 'error';
```

## Más gotchas

Existen otras diferencias entre `var` y `let` que aunque más sutiles, también deben considerarse para evitar errores. 

Imaginemos que declaramos una variable `x` con un valor de `10` y, después de haberla definido, intentamos volver a declarar una variable con el mismo nombre. Si utilizamos `var`, esta segunda declaración será ignorada. Si, por el contrario, utilizamos `let`, el intérprete **arrojará un `SyntaxError`, indicando que ya existe una variable `x` declarada**.

```javascript
var x = 10;
var x;

console.log(x); // 10;

let y = 10;
let y; // SyntaxError: El identificador 'y' ya ha sido declarado

console.log(y);
```

Ahora repitamos el mismo caso pero agregando una asignación a la segunda declaración. En el caso de `var`, la segunda declaración será ignorada pero, aún así, se realizará la asignación del nuevo valor. Con `let`, el error que se produce **impedirá que se realice esta asignación** y la variable conservará el valor que tenía previamente.

```javascript
var x = 10;
var x = 'nuevo valor';

console.log(x); // nuevo valor;

let y = 10;
let y = 'nuevo valor'; // SyntaxError

console.log(y); // 10
```

Por último, cuando se decalara una variable en el ámbito global con `var` se creaba también una propiedad con su mismo nombre dentro del objeto global -que en el caso de los navegadores es `window`. Las variables globales `let` no siguen esta misma mecánica y **no tienen una propiedad equivalente dentro del objeto `window`**  . Fuera eso, las variables globales con `let` y `var` se comportan igual.

```javascript
var foo = 10;
let bar = 20;

console.log(window.foo); // 10
console.log(window.bar); // undefined
```

## const

_ES6_ también permite la creación de _constantes_ utilizando la palabra reservada `const`. Las constantes son muy similares a las variables `let`, pues siguen las mismas reglas de _scope_ y _hoisting_ que las mencionadas anteriormente. Pero su principal característica es que **su valor no puede ser reasignado**. Podemos pensar en ellas como variables de _sólo lectura_. Cuando una constante se declara, forzosamente debe llevar también una asignación, y es este valor el que conservará por el resto del programa.

```javascript
const pi = 3.14; // Constante. Este valor no se puede reasignar

let diametro = 10;
console.log("Circunferencia: ", pi * diametro); // 31.4

pi = 'otro valor'; // TypError: Asignación a constante
console.log(pi); // 3.14 

{ 
  // El uso de {} crean un nuevo scope.
  // Aquí podemos redefinir la constante pi
  // y se creará como una constante aparte
  // a la existente en el scope superiro
  
  const pi = 'pi en nuevo scope';

  console.log(pi); // 'pi en nuevo scope'
}

console.log(pi); // 3.14 (pi en el scope original).
```

Hemos dicho que las constantes no pueden cambiar de valor, pero debemos hacer una aclaración aquí, porque esto no implica que el valor que guarden las constantes sea _inmutable_. **Si en una constante se asigna un tipo de dato complejo -como lo son los _arreglos_ y los _objetos_- todavía es posible alterar el contenido de éstos**. En realidad, cuando se guarda un objeto o un arreglo dentro de una constante, lo que realmente se está guardando es una referencia hacia ellos, no su contenido exacto. Lo que `const` realmente limita no es el valor dentro de la variable, sino que no se pueda reasignar.

```javascript
const mandado = ['agua', 'frutas', 'pizza'];
mandado[2] = 'verduras';
mandado.push('helado');

console.log(mandado); // ["agua", "frutas", "verduras", "helado"]

const libro = {
  titulo: 'el principito',
  autor: 'paquito'
};
libro.autor = 'antoine de saint-exupery';
libro.genero = 'novela corta';

console.log(libro); // {titulo: "el principito", autor: "antoine de saint-exupery", genero: "novela corta"}
```
