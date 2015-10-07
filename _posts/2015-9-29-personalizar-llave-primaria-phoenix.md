---
layout: post
title: Personalizar una llave primaria con Phoenix-Ecto
date:   2015-9-29
categories: Programacion elixir phoenix ecto 
tags: Programación
comments: true
---

Uno de los elementos más interesantes de [Phoenix Framework](http://www.phoenixframework.org/) es Ecto, un DSL (Domain Specific Language) que fue integrado para sus últimas versiones. Pero bueno, en otra ocasión les platicaré mas a detalle sobre Ecto.

Hoy quiero mostrarles cómo crear una llave primaria personalizada usando Ecto.

Pero dirán, ¿para qué crear una llave personalizada? Bueno, pues por default la llave primaria que se genera con Ecto tiene como nombre `id`, esto puede llegar a no ser tan descriptivo cuando queramos hacer un `join` con alguna otra tabla que por default también tendrá una llave primaria llamada `id`. Por ejemplo:

```sql
-- En este ejemplo lo primero que nos mostrará sera un error de que existen columnas con el mismo nombre.

select id
from user
inner join book on user.id = book.user_id
where user.id = '2'
```

Esto se soluciona así:

```sql
-- Implícitamente le agregamos la tabla del campo que buscamos y agregamos un nombre descriptivo

select user.id as user_id
from user
inner join book on user.id = book.user_id
where user.id = '2'
```

¿Pero no sería mas sencillo que desde el principio creáramos la llave primaria como **user_id** y **book_id**?

Pues desde mi punto de vista, sí es más sencillo, así que, tomaré este ejemplo para agregar una llave primaria personalizada con Phoenix y Ecto.

Al crear la migración, tendremos que modificar el campo id:

```elixir
# Al crear la tabla le pasamos como opción primary_key: false, esto para que no cree el id default.
create table(:user, primary_key: false) do
 add :user_id, :serial, primary_key: true
 add :name, :string

 timestamps
end
```

Pasando a nuestro modelo, debemos ajustar la llave primaria:

```
@primary_key {:user_id, :id, autogenerate: true}
@derive {Phoenix.Param, key: :user_id}
schema "user" do
 field :first_name, :string 
 field :last_name, :string

 timestamps
end
```
**@primary_key:** configura el esquema de la llave primaria, espera una tupla con `nombre`, `tipo (:id, :binary_id)` y `opciones`. Por default el valor es {:id, :id, autogenerate: true}`.

Phoenix utiliza el protocolo `Phoenix.Param` para convertir las estructuras de datos en parametros de URL. Este protocolo es utilizado en varias partes del stack de phoenix, entre ellos los helpers.

Utilizando este protocolo Phoenix sabe que tiene que extraer 'id' de '@user'. _Su valor predeterminado es 'id'_.

Con base en esto, ¿qué haremos para que Phoenix se entere que ahora su primary_key es user_id?, usaremos `@derive`. 

**@derive:** Agrega dentro del protocolo Phoenix.Param el nuevo id usando como opción: `key`.


Bueno espero lo escrito en este post les sirva en algún momento, dejen sus comentarios sobre alguna duda que tengan, nos vemos pronto.