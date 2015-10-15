---
layout: post
title: Cómo crear un API REST con Phoenix, Parte 2. Testing.
date:   2015-10-14
categories: Programacion elixir phoenix
tags: Programación
comments: true
---

Una de las partes más importantes en el desarrollo de software es el manejo de pruebas, y en los frameworks modernos es indispensable poder implementarlas fácilmente. En Phoenix se tomaron muy en serio la creación de pruebas unitarias, para ello nos facilitan una integración completa con [ExUnit](http://elixir-lang.org/docs/stable/ex_unit/ExUnit.html), el framework para crear pruebas unitarias en Elixir.

En este post explicaré el funcionamiento y utilización de pruebas usando ExUnit, para luego continuar la construcción de nuestra API RESTful.

Al momento de escribir esto, las versiones actuales de nuestras aplicaciones son:

- **Elixir:** v1.1.0
- **Phoenix:** v1.0.3
- **Ecto:** v1.0.4

Si estás leyendo esto y no son las últimas versiones, coméntalo y actualizaré este tutorial.

### Introducción

Hay diferentes tipos de pruebas, pero a grandes rasgos, podemos reducirlo a 2: **unitarias** y de **integración**.

La idea de las pruebas unitarias es que le den forma a tu código y asegurarse de ejecutar y comprobar todas las líneas de código, por lo menos una vez.

La manera más simple de hacer una prueba consiste en hacer una afirmación. Por ejemplo, con assert:

```
assert 1 == 1
assert 1 != 1
- Test fallido porque la condición no cumple el valor esperado.
```

Todo el código "lógico", que no tiene efectos colaterales, tales como enviar datos a través de la red o escribir en el sistema de archivos, se puede probar con pruebas unitarias.

Si lo que necesitas es probar el flujo completo de algo, por ejemplo, que haya escrito correctamente en base de datos, entonces se trata de un test de integración.

### Pruebas

Tomaremos como ejemplo el API RESTful que creé para mi anterior post, [cómo crear un API REST con Phoenix](/crear-api-rest-phoenix). Ejecutaremos la siguiente sentencia desde nuestra consola:

```bash
mix test
```

Esto nos dará como resultado una lista de errores de las pruebas que fallaron o, simplemente, que todas nuestras pruebas pasaron con éxito.

En nuestro caso es:

```bash
#
# Anteriores pruebas fallidas
# 
  6) test shows chosen resource (ApiJson.UserControllerTest)
     test/controllers/user_controller_test.exs:18
     ** (KeyError) key :id not found in: %ApiJson.User{__meta__: #Ecto.Schema.Metadata<:loaded>, books: #Ecto.Association.NotLoaded<association :books is not loaded>, first_name: nil, inserted_at: #Ecto.DateTime<2015-10-11T04:03:38Z>, last_name: nil, updated_at: #Ecto.DateTime<2015-10-11T04:03:38Z>, user_id: 5}
     stacktrace:
       test/controllers/user_controller_test.exs:21
       
Finished in 3.1 seconds (3.0s on load, 0.1s on tests)
24 tests, 6 failures
```

Aunque el mensaje de error puede llegar a ser muy descriptivo, para encontrar la ubicación exacta del mismo sería bueno conocer la estructura de archivos que usamos para nuestras pruebas:

```bash
test
├── channels
├── controllers
│   ├── book_controller_test.exs
│   ├── page_controller_test.exs
│   └── user_controller_test.exs
├── models
│   ├── book_test.exs
│   └── user_test.exs
├── support
│   ├── channel_case.ex
│   ├── conn_case.ex
│   └── model_case.ex
├── test_helper.exs
└── views
    ├── error_view_test.exs
    ├── layout_view_test.exs
    └── page_view_test.exs
```

Bien, éstas son las 6 pruebas fallidas que deberemos solucionar:

1. test updates and renders chosen resource when data is valid (ApiJson.UserControllerTest)
2. test deletes chosen resource (ApiJson.UserControllerTest)
3. test does not update chosen resource and renders errors when data is invalid (ApiJson.UserControllerTest)
4. test shows chosen resource (ApiJson.BookControllerTest)
5. test deletes chosen resource (ApiJson.BookControllerTest)
6. test shows chosen resource (ApiJson.UserControllerTest)

> Se omite la descripción completa de la prueba fallida, pero pueden ver el stacktrace completo [aquí](https://gist.github.com/Urielable/fbc496466850c7c1c6f1).

Después de haber cambiado las llaves primarias default por nuestras personalizadas, seguimos teniendo 2 pruebas fallidas: 

1. test updates and renders chosen resource when data is valid (ApiJson.UserControllerTest)
2. test does not update chosen resource and renders errors when data is invalid (ApiJson.UserControllerTest)

El mensaje de estas pruebas fallidas es:

```elixir
** (ArgumentError) attempting to cast or change association `books` from `ApiJson.User` that was not loaded. Please preload your associations before casting or changing the model
```

Muy descriptivo, así que para solucionar este problemas iremos a nuestro controlador de `user` para precargar nuestro modelo **books**.

En este caso, siguiendo la descripción de la prueba fallida, iremos a nuestra acción **update** del controlador **user**, y agregaremos la línea `Repo.preload :books` en la inicialización de la instancia **user**:

```elixir
::::
  def update(conn, %{"id" => id, "user" => user_params}) do
    user = Repo.get!(User, id) |> Repo.preload :books
    changeset = User.changeset(user, user_params) 

    case Repo.update(changeset) do
      {:ok, user} ->
        render(conn, "show.json", user: user)
      {:error, changeset} ->
        conn
        |> put_status(:unprocessable_entity)
        |> render(ApiJson.ChangesetView, "error.json", changeset: changeset)
    end
  end
::::
```

Guardamos y corremos de nueva cuenta `mix test`, que nos da como resultado:

```bash
Compiled web/controllers/user_controller.ex
Generated api_json app
........................

Finished in 0.5 seconds (0.3s on load, 0.1s on tests)
24 tests, 0 failures

Randomized with seed 848645
```

Ahora sí, podemos continuar con la construcción de nuestra api.

### Creación y modificación de pruebas

Primero crearemos nuestra prueba en `api_json/test/controllers/user_controller_test.exs`:

```elixir
::::
  test "shows chosen resource and assoc books", %{conn: conn} do
    params         = %{ first_name: "Uriel", last_name: "Molina", books: [%{book: "book", editorial: "nameeee"}] }
    user_changeset = User.changeset(%User{}, params)
    user           = Repo.insert! user_changeset

    conn = get conn, user_path(conn, :show, user)
    assert json_response(conn, 200)["data"] == %{"id" => user.user_id,
      "first_name" => user.first_name,
      "last_name" => user.last_name,
      "books" => Enum.map(user.books , fn(book) -> %{ "book" => book.book, "editorial" => book.editorial } end)
    }
  end
::::
```

En esta prueba le estoy diciendo que, además de los datos del usuario, quiero que en la misma prueba traiga también los libros que tiene relacionados.

Si ejecutamos `mix test`, nos dará la siguiente prueba fallida:

```elixir
::::
  1) test shows chosen resource and assoc books (ApiJson.UserControllerTest)
     test/controllers/user_controller_test.exs:27
     Assertion with == failed
     code: json_response(conn, 200)["data"] == %{"id" => user.user_id(), "first_name" => user.first_name(), "last_name" => user.last_name(), "books" => Enum.map(user.books(), fn book -> %{"book" => book.book(), "editorial" => book.editorial()} end)}
     lhs:  %{"first_name" => "Uriel", "id" => 520, "last_name" => "Molina"}
     rhs:  %{"books" => [%{"book" => "book", "editorial" => "nameeee"}], "first_name" => "Uriel", "id" => 520, "last_name" => "Molina"}
     stacktrace:
       test/controllers/user_controller_test.exs:33
::::
```

> Como nota adicional, el significado de lo prefijos rhs y lhs son: "Right Hand Side" y "Left Hand Side", respectivamente.

Si bien nuestro código está preparado para realizar la inserción de la relación **user-books**, nuestra vista del usuario no lo está. Como vemos, la respuesta de nuestra acción **show**(lhs), difiere de lo que en la prueba le pedimos(rhs).

Entonces vamos a modificar nuestra vista, para ello iremos a nuestro archivo `api_json/web/view/user_view.ex`.

```elixir
::::
defmodule ApiJson.UserView do
  use ApiJson.Web, :view

  def render("index.json", %{users: users}) do
    %{data: render_many(users, ApiJson.UserView, "user.json")}
  end
  
  def render("show.json", %{user: user}) do
    %{data: render_one(user, ApiJson.UserView, "user.json")}
  end

  def render("user.json", %{user: user}) do
    %{id: user.user_id,
      first_name: user.first_name,
      last_name: user.last_name
    }
  end
end
::::
```

Lo que haremos será agregarle el arreglo de libros al `render` del usuario:

```elixir
::::
  def render("user.json", %{user: user}) do
    %{id: user.user_id,
      first_name: user.first_name,
      last_name: user.last_name,
      books: Enum.map(user.books , fn(book) -> %{ book: book.book, editorial: book.editorial } end)
    }
  end
::::
```

Podemos apreciar cómo es el proceso de `render` de los elementos. En nuestro template `index` tenemos lo siguiente:

```elixir
::::
  def render("index.json", %{users: users}) do
    %{data: render_many(users, ApiJson.UserView, "user.json")}
  end
::::
```

Y en nuestro `show`:

```Elixir
::::
  def render("show.json", %{user: user}) do
    %{data: render_one(user, ApiJson.UserView, "user.json")}
  end
::::
```

Es decir, al final ambas usan una (show.json) o varias veces (index.json) lo siguiente:

```Elixir
::::
  def render("user.json", %{user: user}) do
    %{id: user.user_id,
      first_name: user.first_name,
      last_name: user.last_name
    }
  end
::::
```

Eso quiere decir que, además de modificar el template `user.json`, tendremos que modificar también el test `'shows chosen resource'` del usuario, quedando de la siguiente forma:

```
::::
  test "shows chosen resource", %{conn: conn} do
    user = Repo.insert! %User{}
    conn = get conn, user_path(conn, :show, user)
    assert json_response(conn, 200)["data"] == %{"id" => user.user_id,
      "first_name" => user.first_name,
      "last_name" => user.last_name,
      "books" => []
      }
  end
  # Como está prueba solo está ingresando usuario sin libros, se agrega el arreglo books vacío.
::::
```

Hasta aquí corramos nuestras pruebas:

```bash
$ mix test
Compiled web/controllers/user_controller.ex
Generated api_json app
.......

  1) test creates and renders resource when data is valid (ApiJson.UserControllerTest)
     test/controllers/user_controller_test.exs:47
     ** (Protocol.UndefinedError) protocol Enumerable not implemented for #Ecto.Association.NotLoaded<association :books is not loaded>
     stacktrace:
       (elixir) lib/enum.ex:1: Enumerable.impl_for!/1
       (elixir) lib/enum.ex:112: Enumerable.reduce/3
       (elixir) lib/enum.ex:981: Enum.map/2
       (api_json) web/views/user_view.ex:16: ApiJson.UserView.render/2
       (api_json) web/views/user_view.ex:9: ApiJson.UserView.render/2
       (phoenix) lib/phoenix/view.ex:341: Phoenix.View.render_to_iodata/3
       (phoenix) lib/phoenix/controller.ex:628: Phoenix.Controller.do_render/4
       (api_json) web/controllers/user_controller.ex:1: ApiJson.UserController.action/2
       (api_json) web/controllers/user_controller.ex:1: ApiJson.UserController.phoenix_controller_pipeline/2
       (api_json) lib/phoenix/router.ex:255: ApiJson.Router.dispatch/2
       (api_json) web/router.ex:1: ApiJson.Router.do_call/2
       (api_json) lib/api_json/endpoint.ex:1: ApiJson.Endpoint.phoenix_pipeline/1
       (api_json) lib/phoenix/endpoint/render_errors.ex:34: ApiJson.Endpoint.call/2
       (phoenix) lib/phoenix/test/conn_test.ex:193: Phoenix.ConnTest.dispatch/5
       test/controllers/user_controller_test.exs:48

.................
Finished in 1.6 seconds (1.0s on load, 0.6s on tests)
25 tests, 1 failures

Randomized with seed 114371
```

Bueno, esto es porque al insertar nuestro elemento user, se ocupa hacer un `preload` de la asociación.

> [Preload](http://hexdocs.pm/ecto/Ecto.Query.html#preload/3): Sirve para especificar que una asociación sea precargada dentro de un modelo.

Para hacer esta precarga tendremos que modificar nuestra acción `create`, para que después de insertar el usuario, precargue sus asociaciones, así que vamos al archivo `api_json/web/controllers/user_controllers.ex`:

```elixir
::::
  def create(conn, %{"user" => user_params}) do
    changeset = User.changeset(%User{}, user_params)
    case Repo.insert(changeset) do
      {:ok, user} ->
        user = user |> Repo.preload :books
        conn
        |> put_status(:created)
        |> put_resp_header("location", user_path(conn, :show, user))
        |> render("show.json", user: user)
      {:error, changeset} ->
        conn
        |> put_status(:unprocessable_entity)
        |> render(ApiJson.ChangesetView, "error.json", changeset: changeset)
    end
  end
::::
```

De nuevo corremos nuestras pruebas:

```bash
$ mix test
.........................

Finished in 2.8 seconds (1.7s on load, 1.1s on tests)
25 tests, 0 failures

Randomized with seed 739900
```

Y listo, hemos creado nuestra primer prueba en Elixir.

![Aplausos](../assets/images/joker_clap.gif)

### Conclusión

Estuvo un poco larga esta parte de la construcción de nuestra API RESTful, pero creo que es importante terminar todo un flujo de cómo crear, modificar y reparar una prueba.

Sé que es algo que, en ocasiones, no estamos acostumbrados a hacer (me incluyo), por falta de tiempo o por alguna otra razón, pero es indispensable en el desarrollo de software.

Si quieren conocer un poco más sobre pruebas pueden revisar la documentación oficial de [ExUnit](http://elixir-lang.org/docs/stable/ex_unit/ExUnit.DocTest.html#content), que además está muy completa.

También les dejo [esta presentación](http://machinesareus.github.io/elixir_testing/#1), donde Agustín Ramos ([MachinesAreUs](https://twitter.com/MachinesAreUs)) nos explica un poco más a detalle sobre los test en Elixir.

Cualquier duda o comentario que tengan al respecto de este post, coméntenlo y trataré de resolverla.

[Repositorio con ejemplo.](https://github.com/Urielable/api_json)