---
layout: post
title: Cómo crear un API REST con Phoenix, Parte 1. <Legado>
date:   2015-10-07
categories: Programacion elixir phoenix
tags: Programación
comments: true
---


_________

**NOTA:** Pense en actualizar el post a las versiones actuales, pero mejor lo dejaré como legado, será interesante ver que cambios hay de esta version a las versiones actuales. 

Esperen el post o busquenlo en los post, ¡quiza ya lo escribí!.
_________

Hola, en esta ocasión les traigo un tutorial de como crear una api REST usando [Phoenix framework](http://www.phoenixframework.org/).

Al momento de escribir esto, las versiones actuales de nuestras aplicaciones son:

- **Elixir:** v1.1.0
- **Phoenix:** v1.0.3
- **Ecto:** v1.0.4

Si estás leyendo esto y no son las últimas versiones, comentalo y voy a actualizar este tutorial.

### Instalar phoenix

Las mejores instrucciones para instalar phoenix, pueden ser encontradas en su [pagina web](http://www.phoenixframework.org/docs/installation).

### Primer paso: crear nuestro proyecto.

Primero les dejo una excelente presentación con una explicación de como diseñar un API REST creada por [Brian Mulloy](https://twitter.com/landlessness), pueden ver la presentación [aquí](../assets/tarballs/restful-api-design--mulloy-2ed.pdf).

Continuando con lo que les quiero mostrar, Phoenix nos facilita la creación de una API REST, quitandonos de encima html inescesarios para poder crear el proyecto lo hacemos de esta manera:

```bash
$ mix phoenix.new api_json --no-html
```

Esto nos generará un proyecto sin templates html, que al menos en este momento, no son necesarios. 

Lo siguiente que haremos es crear nuestros recursos, para ello utilizaremos los generadores de Phoenix utilizando **phoenix.gen.json**.

```bash
$ mix phoenix.gen.json User users user_id:serial first_name:string last_name:string
```

Despues de crear el recurso, tenemos que agregarlo en las rutas, para hacer esto habilitaremos el scope **api**

```elixir
scope "/api", Api do
  pipe_through :api
  resources "/users", UserController, except: [:new, :edit]
end
```

Ahora generaremos el siguiente recurso, books:

```bash
$ mix phoenix.gen.json Book books book_id:integer book:string editorial:string
```
Y repetimos el paso de agregar el recurso en nuestras rutas, quedando de la siguiente forma:

```elixir
scope "/api", Api do
  pipe_through :api
  resources "/users", UserController, except: [:new, :edit]
  resources "/books", BookController, except: [:new, :edit]
end
```

### Segundo paso: Migraciones

Ahora modificaremos las migraciones, para crear la base de datos a nuestro gusto. Primero modificaremos nuestra migración de usuarios, quedará de la siguiente forma:

```elixir
defmodule Api.Repo.Migrations.CreateUser do
  use Ecto.Migration

  def change do
    create table(:users, primary_key: false) do
      add :user_id, :serial, primary_key:true
      add :first_name, :string
      add :last_name, :string

      timestamps
    end

  end
end
```
Luego nuestra migración de books:

```elixir
defmodule ApiJson.Repo.Migrations.CreateBook do
  use Ecto.Migration

  def change do
    create table(:books, primary_key: false) do
      add :book_id, :serial, primary_key: true
      add :book, :string
      add :editorial, :string
      add :user_id, references(:users, [column: :user_id])

      timestamps
    end
    create index(:books, [:book_id])

  end
end

```
Si notan en está migración tenemos una referencia al recurso **User**, además del indice book_id.

Terminando podemos correr las migraciones con ecto:

```bash
# Creamos la base de datos.
$ mix ecto.create
# Creamos nuestras migraciones.
$ mix ecto.migrate
```

### Tercer paso: Modelos

Ahora iremos a modificar nuestros modelos, comenzaremos por el modelo de usuarios:

```elixir
::::
  @derive {Phoenix.Param, key: :user_id}
  @primary_key {:user_id, :id, autogenerate: true}
  schema "users" do
    field :first_name, :string
    field :last_name, :string

    has_many :books, ApiJson.Book, foreign_key: :user_id, references: :user_id

    timestamps
  end
  
  
  @required_fields ~w(first_name last_name)
  @optional_fields ~w(books)
::::
```

Ahora pasemos al modelo de libros:

```elixir
::::
  @derive {Phoenix.Param, key: :book_id}
  @primary_key {:book_id, :id, autogenerate: true}

  schema "books" do
    field :book, :string
    field :editorial, :string

    belongs_to :user, ApiJson.User, references: :user_id

    timestamps
  end
::::
```

Como ven usamos las opciones, **@derive** y **@primary_key**, esto es por que al generar las migraciones cambiamos el nombre de nuestra llave primaria, pueden ver con mas detalle el por que de estas opciones en mi anterior post [Personalizar una llave primaria con Phoenix-Ecto](/personalizar-llave-primaria-phoenix/).

Ya podemos arrancar nuestra api REST, asi que nos movemos a raiz de nuestro proyecto y ejecutamos:

```elixir
$ iex -S mix phoenix.server
```

Ahora probemos nuestra API, para esto usaremos la siguiente CURL: 

```bash
$ curl -i -X POST -H 'Content-Type:application/json' -d '{"user": {"first_name":"uriel","last_name":"molina", "books":[{"book":"Neo","editorial":"responsible"}, {"book":"user","editorial":"nanderson"}] } }' http://localhost:4000/api/users > out.html
```
Responde:

```json
HTTP/1.1 201 Created
server: Cowboy
date: Thu, 08 Oct 2015 00:59:40 GMT
content-length: 60
content-type: application/json; charset=utf-8
cache-control: max-age=0, private, must-revalidate
x-request-id: q8n5lpuro3qkgihdmp26ove415thtm49
location: /api/users/20

{"data":{"last_name":"molina","id":20,"first_name":"uriel"}}
```

Listo tenemos nuestra API REST.

Aunque sea muy basico, cualquier duda que tengan dejen un comentarlo.

[Repositorio con ejemplo.](https://github.com/Urielable/api_json)