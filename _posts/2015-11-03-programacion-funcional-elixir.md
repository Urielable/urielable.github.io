---
layout: post
title: Elixir, otro paradigma.
date:   2015-11-02
categories: elixir "programación funcional"
tags: Programación
comments: true
---

En los últimos milisegundos he estado leyendo, aprendiendo, construyendo y escribiendo sobre Elixir. Más específicamente sobre Phoenix, pero creo que por entusiasmo y ánimo de compartir lo que estaba aprendiendo, omití explicar algunos conceptos que, creo, son muy importantes conocer antes de continuar aprendiendo.

### Programación funcional

La definición más conocida sobre programación funcional es la siguiente:

> En ciencias de la computación, la programación funcional es un paradigma de programación declarativa basado en la utilización de funciones aritméticas que no maneja datos mutables o de estado.

La programación funcional cambia mucho respecto al paradigma de programación imperativo, que es como funciona la gran mayoría de los lenguajes de programación. La idea es conocer un nuevo paradigma: uno declarativo, que se base en declarar propiedades y reglas, en lugar de sentencias de instrucciones.

Esto podría ser la razón de por qué es complicado comenzar con algún lenguaje que use este tipo de paradigma. O que terminemos programando imperativamente en un lenguaje que nos permite usar el paradigma declarativo.

**Programación imperativa**

> Describe la computación en términos del estado del programa para cambiar el estado de un programa y  sentencias para cambiar dicho estado. (...) conjunto de instrucciones que le indican al computador cómo realizar una tarea.
>
> -Wikipedia
 
**Programación declarativa**

> (...) es un paradigma de programación que está basado en el desarrollo de programas especificando o "declarando" un conjunto de condiciones, proposiciones, afirmaciones, restricciones, ecuaciones o transformaciones que describen el problema y detallan su solución.
>
> -Wikipedia

**Ejemplos.**

Programación imperativa:

```
# Programacion Imperativa
// books   = [{name: "Metamorfosis", owner_id: 1}, {...}, ... ]
// authors = [{id: 1, name:"Kafka"}, {...}, ... ]

var booksWithAuthors = []
var book, author

for(var di=0; di < books; di ++){
    books = books[di]
    for(var oi=0; oi < authors; oi ++){
        booksWithAuthors.push({
            book: book,
            author: author
        })
    }
}
```

Programación declarativa:

```
# Programación declarativa
SELECT * from books
INNER JOIN authors
WHERE books.author_id = author.id
```

Programación imperativa:

```ruby
grades = { "john" => 8, "mary" => 8.8, "marty" => 9.7, "elisa" => 9.5 }

def avg list
  i   = 0
  sum = 0
  while i < list.length
    sum += list[i]
    i += 1
  end
  sum / list.length.to_f
end

def grades_to_list grades
  i = 0
  result = []
  keys = grades.keys

  while i < keys.length
    key = keys[i]
    i += 1
    result.push(grades[key])
  end

  result
end

p avg grades_to_list grades
```

Programación funcional:

```Elixir
# Ruby
# Califaciones
grades = { "john" => 8, "mary" => 8.8, "marty" => 9.7, "elisa" => 9.5 }

# Algoritmo "average", ruby mode
avg = ->(list) { 
  list.reduce(:+) / list.size 
}

# Algoritmo "max", ruby mode
max = ->(list) { 
  list.max_by {|k,v| v}
}

# Extra califaciones y calcula `average` o `max`
def calculate fn, grades
  fn.(grades.map { |child, grade| grade })
end

# Imprime resultado
p calculate avg, grades
p calculate max, grades
```

En este último ejemplo podemos ver:

- **[Composicion](https://es.wikipedia.org/wiki/Objeto_%28programaci%C3%B3n%29#Composici.C3.B3n):** porque la función para un promedio y la de un máximo son intercambiables.
- **[Funciones de orden superior](https://es.wikipedia.org/wiki/Funci%C3%B3n_de_orden_superior):** porque la función de promedio y máximo se podrán pasar como parámetros a otra función.
- **[Inmutabilidad](https://es.wikipedia.org/wiki/Objeto_Inmutable):** porque no se necesita una variable mutable para mantener el estado.
- **[Y transparencia referencial](https://es.wikipedia.org/wiki/Transparencia_referencial):** porque las funciones son puras, no causan efectos colaterales.

### Elixir

Elixir es un lenguaje de programación, creado por [Jose Valim](https://twitter.com/josevalim), está construido sobre la plataforma de erlang; Elixir es un lenguaje **dinámico** y **funcional**, diseñado para construir **aplicaciones escalables** y **mantenibles**, además tiene una sintaxis muy flexible.

Elixir cuenta con las siguientes caracteristicas:

- Compila bytecode para la Erlang Virtual Machine (BEAM)
- Se puede invocar código Erlang dentro de Elixir y viceversa
- Polimorfismo
- Pattern Matching
- Funciones de alto orden
- Inmutabilidad

### ¿Por qué Elixir?

El pensar diferente, esto no sólo nos ayudará para programar en Elixir, si no en otros lenguajes. Verán que luego que se adentren en Elixir, comenzarán a ver otras formas de resolver los problemas.

Erlang, el tener este viejón por detras... (a caray). Bueno, al tener la base de este lenguaje, nos da un sin número de ventajas y experiencia adquirida.

Comunidad, la comunidad alrededor de Elixir. Apenas está en la versión 1.1.1 y ya tiene 4 libros impresos.

Adaptable, rendimiento, tolerante a fallos. Para muestra, dejo algunas comparaciones que ya probaron y comprobaron estas caracteristicas:

- [Phoenix showdown.](https://github.com/mroth/phoenix-showdown)
- [Phoenix Channels 1 million connections single server.](https://www.youtube.com/watch?v=N4Duii6Yog0&feature=youtu.be)
- [Elixir vs Ruby Showdown - Phoenix vs Rails.](/http://www.littlelines.com/blog/2014/07/08/elixir-vs-ruby-showdown-phoenix-vs-rails/)

Respaldado por grandes programadores, como:

- Dave Thomas
- Devin Torres
- Simon St. Laurent
- Joe Armstrong
- Benjamin Tan
- Jose Valim

**Introducción**

Al momento de escribir esto, las versiones actuales de nuestras aplicaciones son:

- **Erlang:** Erlang/OTP 18
- **Elixir:** v1.1.1

Si no has instalado Elixir, puedes seguir las instrucciones que se [encuentran en la página oficial](http://elixir-lang.org/install.html).

Elixir cuenta con un modo interactivo (repl) donde podemos escribir cualquier expresión de Elixir. Para iniciar la consola interactiva ejecutamos:

```
$ iex
Erlang/OTP 18 [erts-7.1] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

Interactive Elixir (1.1.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)>
```

Nos muestra cuál es la versión de Erlang/OTP y Elixir que estamos usando.

Iniciaré explicando algunas de las funciones básicas de Elixir.

### Elixir: Tipos básicos

```
iex(1)> 1          # integer
1
iex(2)> 0x1F       # integer
31
iex(3)> 1.0        # float
1.0
iex(4)> true       # boolean
true
iex(5)> :atom      # atom / symbol
:atom
iex(6)> "elixir"   # string
"elixir"
iex(7)> [1, 2, 3]  # list
[1, 2, 3]
iex(8)> {1, 2, 3}  # tuple
{1, 2, 3}
```

**Strings**

Los strings en Elixir soportan por default codificación en utf-8.

```
iex(1)> "Hellö"
"Hellö"
iex(2)> "àáäâ"
"àáäâ"
```

Son representados internamente como binarios.

```
iex(3)> String.upcase("hellö")
"HELLÖ"
```

Puedes revisar más sobre `strings` [aquí](http://elixir-lang.org/getting-started/binaries-strings-and-char-lists.html)

**Listas**

Se pueden crear listas de cualquier tipo, utilizando corchetes:

```
iex(1)> [1, 2, true, 3]
[1, 2, true, 3]
```

Se pueden concatenar o substraer datos con ++ o --.

```
iex(2)> [1, 2, 3] ++ [4, 5, 6]
[1, 2, 3, 4, 5, 6]
iex(3)> [1, true, 2, false, 3, true] -- [true, false]
[1, 2, 3, true]
```

Pueden encontrar más información de los tipos básicos [aquí](http://elixir-lang.org/getting-started/basic-operators.html).

**Tuplas**

Las tuplas se definen usando llaves, similar a las listas:

```
iex(1)> {:ok, "hello"}
{:ok, "hello"}
```

Son elementos en memoria, se puede acceder a la tupla usando índices:

```
iex(2)> tuple = {:ok, "hello"}
{:ok, "hello"}
```


### Elixir: Operadores básicos

```
@                                   Unary
.                                   Left to right
+ - ! ^ not ~~~                     Unary
* /                                 Left to right
+ -                                 Left to right
++ -- .. <>                         Right to left
in                                  Left to right
|> <<< >>> ~>> <<~ ~> <~ <~> <|>    Left to right
< > <= >=                           Left to right
== != =~ === !==                    Left to right
&& &&& and                          Left to right
|| ||| or                           Left to right
=                                   Right to left
=>                                  Right to left
|                                   Right to left
::                                  Right to left
when                                Right to left
<-, \\                              Left to right
&                                   Unary
```

**Pattern matching**

En Elixir el operador `=` en realidad es un operador de igualdad utilizado para hacer *"pattern matching"*.

```
iex(1)> x = 1
1
iex(2)> x
1
iex(3)> 1 = x
1
iex(4)> 2 = x
** (MatchError) no match of right hand side value: 1
```

Este operador no sólo sirve para este tipo de igualdad, si no tambien es usado en estructuras de datos más complejas:

```
iex(5)> {a, b, c} = {:hello, "world", 17}
{:hello, "world", 127}
iex(6)> a
:hello
iex(7)> b
"world"
iex(8)> c
17
```

Hay algunos casos en el que al usar *pattern matching*, no se puede igualar, en los casos ccmo:

- Cuando las tuplas son de diferente tamaño.

```
iex(9)> {a, b, c} = {:hello, "world"}
** (MatchError) no match of right hand side value: {:hello, "world"}
```

- Cuando se comparan diferentes tipos de datos:

```
iex(9)> {a, b, c} = [:hello, "world", "!"]
** (MatchError) no match of right hand side value: [:hello, "world", "!"]
```

Otro caso interesante es cuando la tupla de lado izquierdo inicia con el átomo `:ok`:

```
iex(9)> {:ok, result} = {:ok, 17}
{:ok, 17}
iex(10)> result
17
iex(11)> {:ok, result} = {:error, :oops}
** (MatchError) no match of right hand side value: {:error, :oops}
```

En este caso ambas tuplas deben iniciar con el atomo `:ok`.

Puedes encontrar más información de *"pattern matching"*, [aquí](http://elixir-lang.org/getting-started/pattern-matching.html).

### Elixir: ejercicios.

Comentaré algunas caracteristicas de Elixir haciendo algunos ejercicios.

**Fibonacci.**

```elixir
defmodule Fibonnaci do
  def fib(0), do: [0]

  def fib(1), do: [0, 1]

  def fib(n) do
    fib(n-1, [0, 1])
  end

  def fib(n, xs) do
    if n > 0 do
      fib(n -1, xs ++ [Enum.at(xs, -1) + Enum.at(xs, -2)])
    else
      xs
    end
  end
end

IO.inspect Fibonnaci.fib(10)
```

En este ejemplo podemos encontrar:

- Un buen ejemplo de recursión, usando la función `fib` para nuestra itaración hasta encontrar la sucesión con el `n` buscado.

- El manejo de listas, utilizando el modulo [Enum de Elixir](http://elixir-lang.org/docs/v1.1/elixir/Enum.html). Este módulo sirve para implementar el [protocolo Enumerable](http://elixir-lang.org/docs/v1.1/elixir/Enumerable.html).

- La definición de módulos en Elixir, utilizando la macro [defmodule](http://elixir-lang.org/docs/v1.1/elixir/Kernel.html#defmodule/2).

Ejemplo:

```elixir
# Definición del modulo
iex(1)> defmodule Foo do
...>   # contenido del modulo
...>   defmodule Bar do
...>   end
...>   def bar, do: :baz 
...> end

iex(2)> Foo.bar
:baz
```

El llamado al modulo se hace:

`$ Foo.bar`

**Números primos.**

Reto [Desafio: Divisores y primos](http://retronet.com.ar/?p=64)

```elixir
divisors = fn(n) ->
  for d <- 1..n, rem(n, d) == 0, do: d
end

possible_primes = fn(n) ->
  for d <- 1..n, rem(n, d) == 0, do: div(n, d) + d
end

primes = %{}

prime? = fn(n) ->  
  if Dict.get(primes, n) do
    true
  else
    is_prime = Enum.count(divisors.(n)) == 2
    Dict.put(primes, n, is_prime)
    is_prime
  end
end

IO.puts 10_000..1
  |> Stream.filter( &(Enum.all?(possible_primes.(&1), prime?)) ) 
  |> Enum.take(1)
  |> hd
```
En este ejemplo podemos encontrar:

- Ejemplo de una [evaluación perezosa](https://es.wikipedia.org/wiki/Evaluaci%C3%B3n_perezosa).

- En este caso usamos `Stream.filter`, ya que sólo ocupamos el último posible primo para mostrar, es decir, no es necesario tomar todos los valores de la lista, esto hace que no mantengamos en memoria una lista enorme, que podría ser imposible de manejar.

- La utilización de funciones anónimas.

- Uso de modulo Enum:

    - [Enum.take](http://elixir-lang.org/docs/v1.1/elixir/Enum.html#take/2).
    - [Enum.count](http://elixir-lang.org/docs/v1.1/elixir/Enum.html#count/2).
    - [Enum.all?](http://elixir-lang.org/docs/v1.1/elixir/Enum.html#all?/2).

- Uso del módulo Stream:
	- [Stream](http://elixir-lang.org/docs/stable/elixir/Stream.html).

**Conclusión**
 
Por ahora, creo que esto es todo, en un próximo post sobre Elixir me enfocaré sólo en ejercicios, katas, desafíos, como les gusten llamar.

Por ahora, muchas gracias por leer, y espero les ayude en algo en su proceso de iniciación en Elixir.

Cualquier duda o comentario que tengan al respecto de este post, coméntenlo y trataré de resolverla.
