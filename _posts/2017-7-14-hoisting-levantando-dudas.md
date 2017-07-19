---
layout: post
title: Hoisting. Levantando dudas
excerpt: Una más de las curiosidades del intérprete de JS. ¿Qué es hoisting? ¿Cómo afecta a las variables? ¿Qué reglas sigue?
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
console.log('x = ' + x);
```

## Declaraciones vs. asignaciones

Volvamos a tomar una suposición más educada sobre el siguiente código:

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

La última línea se dividió en dos: en una línea está la declaración y en otra la asignación de la variable.  

Ya divididas las instrucciones es más fácil ver por qué el valor es `undefined` al imprimirlo. Las declaraciones de variables (y funciones) son atendidas durante el tiempo de compilación, mientras que las asignaciones lo son durante el tiempo de interpretación. Y debido a eso es que **sólo las declaraciones sufren de _hoisting_, las asignaciones no**.

Clarificado eso, el código equivalente, después del efecto de hoisting será:

```javascript
var x;
console.log(x);
x = 10;
```

Ahora sí resulta evidente por qué el valor `undefined`.
