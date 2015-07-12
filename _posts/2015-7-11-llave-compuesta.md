---
layout: post
title: Llave compuesta en PostgreSQL
date:   2015-07-11
categories: Programacion "Base de datos"
tags: Programación
comments: true
---

Después de buscar y buscar en la internerd sobre cómo crear una llave compuesta en PostgreSQL, encontre la solución, así que la compartiré por si alguna vez lo necesitan.

Antes que nada es bueno saber que sí es posible hacer una llave compuesta y no es tan complicado una vez que te pones a leer la documentación.

Pondré de ejemplo las siguientes tablas:


```
CREATE TABLE public.country
(
 country_id varchar(2),
 country varchar(40)

 CONSTRAINT pk_country_country_id PRIMARY KEY (country_id)
) 
WITH (
 OIDS = FALSE
)
;

CREATE TABLE public.state
(
 state_id varchar(2),
 state varchar(50),
 country_id varchar(2)
)
WITH (
 OIDS = FALSE
)
;
```

Ahora generaremos la llave compuesta en nuestra tabla State, para que de esta forma tengamos como llave primaria `[country_id, state_id]`.

Para poder crear nuestra llave primaria, los campos que serán nuestra llave compuesta deben de ser `NOT NULL` y el campo principal de nuestra tabla `UNIQUE`. Quedará de la siguiente forma:

```
CREATE TABLE public.state
(
 state_id varchar(2) NOT NULL UNIQUE,
 state varchar(50),
 country_id varchar(2) NOT NULL UNIQUE,
 PRIMARY KEY(country_id, state_id)
)
WITH (
 OIDS = FALSE
)
```

Listo, tienen su llave primaria.

Saludos.