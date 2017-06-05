# 3- Templates, Partials y Layouts

Como se mencionó, la salida `HTML` final es una composición de tres elementos Rails: Templates, Partials y Layouts. A continuación se presenta un breve resumen de cada uno de ellos.

### 3.1 Templates

Las plantillas de Action View se pueden escribir de varias maneras. Si el archivo de plantilla tiene una extensión `.erb`, utiliza una mezcla de `ERB` \(Embedded Ruby\) y `HTML`. Si el archivo de plantilla tiene una extensión `.builder`, se utiliza la biblioteca `Builder::XmlMarkup`.

Rails soporta múltiples sistemas de plantilla y utiliza una extensión de archivo para distinguir entre ellos. Por ejemplo, un archivo `HTML` que utiliza el sistema de plantillas `ERB` tendrá `.html.erb` como una extensión de archivo.

#### 3.1.1 ERB

Dentro de una plantilla de `ERB`, el código Ruby se puede incluir usando las etiquetas `<% %>` y `<%= %>`. Las etiquetas `<% %>` se utilizan para ejecutar código Ruby que no devuelven nada, como condiciones, bucles o bloques, y las etiquetas `<%= %>` se utilizan cuando se desea que se devuelva un resultado.

Considere el bucle siguiente para los nombres:

```ruby
<h1>Names of all the people</h1>
<% @people.each do |person| %>
  Name: <%= person.name %><br>
<% end %>
```

El bucle se configura utilizando etiquetas de inserción \(`<% %>`\) y el nombre se inserta utilizando las etiquetas embebidas de salida `(<%= %>`\). Tenga en cuenta que esto no es solo una sugerencia de uso: las funciones de salida regulares como `print` y `puts` no se renderizarán a la vista con plantillas `ERB`. Así que esto estaría mal:

```ruby
<%# WRONG %>
Hi, Mr. <% puts "Frodo" %>
```

Para suprimir los espacios en blanco iniciales y remotos, puede utilizar `<%-  -%>` de forma intercambiable con `<%` y `%>`.

#### 3.1.2 Builder

Las plantillas de Builder son una alternativa más programática a `ERB`. Son especialmente útiles para generar contenido `XML`. Un objeto `XmlMarkup` denominado `xml` se pone automáticamente a disposición de plantillas con una extensión .`builder`.

Estos son algunos ejemplos básicos:

```ruby
xml.em("emphasized")
xml.em { xml.b("emph & bold") }
xml.a("A Link", "href" => "http://rubyonrails.org")
xml.target("name" => "compile", "option" => "fast")
```

Que produciría:

```ruby
<em>emphasized</em>
<em><b>emph &amp; bold</b></em>
<a href="http://rubyonrails.org">A link</a>
<target option="fast" name="compile" />
```

Cualquier método con un bloque será tratado como una etiqueta de marcado `XML` con marcas anidadas en el bloque. Por ejemplo, lo siguiente:

```ruby
xml.div {
  xml.h1(@person.name)
  xml.p(@person.bio)
}
```

Produciría algo como:

```ruby
<div>
  <h1>David Heinemeier Hansson</h1>
  <p>A product of Danish Design during the Winter of '79...</p>
</div>
```

A continuación se muestra un ejemplo completo de `RSS` utilizado en Basecamp:

```ruby
xml.rss("version" => "2.0", "xmlns:dc" => "http://purl.org/dc/elements/1.1/") do
  xml.channel do
    xml.title(@feed_title)
    xml.link(@url)
    xml.description "Basecamp: Recent items"
    xml.language "en-us"
    xml.ttl "40"

    for item in @recent_items
      xml.item do
        xml.title(item_title(item))
        xml.description(item_description(item)) if item_description(item)
        xml.pubDate(item_pubDate(item))
        xml.guid(@person.firm.account.url + @recent_items.url(item))
        xml.link(@person.firm.account.url + @recent_items.url(item))
        xml.tag!("dc:creator", item.author_name) if item_has_creator?(item)
      end
    end
  end
end
```

#### 3.1.3 Jbuilder

Jbuilder es una gema que mantiene el equipo de Rails y que se incluye en el Railfile Gemfile por defecto. Es similar a Builder, pero se utiliza para generar `JSON`, en lugar de `XML`.

Si no lo tiene, puede agregar lo siguiente a su Gemfile:

```ruby
gem 'jbuilder'
```

Un objeto Jbuilder llamado `json` se pone automáticamente a disposición de las plantillas con una extensión `.jbuilder`.

Aquí está un ejemplo básico:

```ruby
json.name("Alex")
json.email("alex@example.com")
```

Lo que produce

```ruby
{
  "name": "Alex",
  "email": "alex@example.com"
}
```

Consulte la documentación de Jbuilder para obtener más ejemplos e información.

#### 3.1.4 Caché de plantillas

De forma predeterminada, Rails compilará cada plantilla en un método para procesarla. Al modificar una plantilla, Rails comprobará la hora de modificación del archivo y la recompilará en modo de desarrollo.



### 3.2 Partials

Las plantillas parciales, usualmente llamadas "partials", son otro dispositivo para romper el proceso de renderizado en bloques más manejables. Con partials, puede extraer fragmentos de código de sus plantillas para separar archivos y también reutilizarlos a través de sus plantillas.

#### 3.2.1 Nombrando partials

Para renderizar un parcial como parte de una vista, utilice el método `render` dentro de la vista:

```ruby
<%= render "menu" %>
```

Esto convertirá un archivo denominado `_menu.html.erb` en ese punto dentro de la vista que se está procesando. Tenga en cuenta el carácter de subrayado principal: los partials se nombran con un carácter de subrayado principal para distinguirlos de las vistas regulares, aunque se consulten sin el subrayado. Esto es válido incluso cuando está extrayendo una parte de otra carpeta:

```ruby
<%= render "shared/menu" %>
```

Ese código extraerá el parcial de `app/views/shared/_menu.html.erb`



#### 3.2.2 Uso de Partials para simplificar las vistas

Una forma de usar partials es tratarlos como el equivalente de subrutinas; Una forma de mover los detalles de una vista para que pueda captar lo que está pasando más fácilmente. Por ejemplo, puede tener una vista que tenga este aspecto:

```ruby
<%= render "shared/ad_banner" %>
 
<h1>Products</h1>
 
<p>Here are a few of our fine products:</p>
<% @products.each do |product| %>
  <%= render partial: "product", locals: { product: product } %>
<% end %>

<%= render "shared/footer" %>
```

En este caso, los partials `_ad_banner.html.erb` y `_footer.html.erb` pueden contener contenido que se comparte entre muchas páginas de la aplicación. No necesita ver los detalles de estas secciones cuando se está concentrando en una página en particular.

#### 3.2.3 render sin opciones de partials ni locals

En el ejemplo anterior, `render` toma 2 opciones: `partial` y `locals`. Pero si estas son las únicas opciones que desea pasar, puede omitir el uso de estas opciones. Por ejemplo, en lugar de:

```ruby
<%= render partial: "product", locals: { product: @product } %>
```

También puede hacer:

```ruby
<%= render "product", product: @product %>
```

#### 3.2.4 Las opciones as y object

De forma predeterminada `ActionView::Partials::PartialRenderer` tiene su objeto en una variable local con el mismo nombre que la plantilla. Por lo tanto, dado:

```ruby
<%= render partial: "product" %>
```

Dentro del partial `_product` obtendremos `@product` en la variable local `product`, como si hubiéramos escrito:

```ruby
<%= render partial: "product", locals: { product: @product } %>
```

















