# 2.2 Uso del render

En la mayoría de los casos, el método `ActionController::Base#render` hace el trabajo pesado de renderizar el contenido de su aplicación para su uso en el navegador. Hay una variedad de maneras de personalizar el comportamiento de `render`. Puede renderizar la vista predeterminada para una plantilla de Rails, o una plantilla específica, o un archivo, o código inline, o nada en absoluto. Puede procesar texto, `JSON` o `XML`. También puede especificar el tipo de contenido o el estado `HTTP` de la respuesta renderizada.

Si desea ver los resultados exactos de una llamada a `render` sin necesidad de inspeccionarla en un navegador, puede llamar a `render_to_string`. Este método toma exactamente las mismas opciones que `render`, pero devuelve una cadena en lugar de enviar una respuesta al navegador.

### 2.2.1 Visualización de una acción

Si desea representar la vista que corresponde a una plantilla diferente dentro del mismo controlador, puede utilizar `render` con el nombre de la vista:

```ruby
def update
  @book = Book.find(params[:id])
  if @book.update(book_params)
    redirect_to(@book)
  else
    render "edit"
  end
end
```

Si falla la llamada a `update`, llama nuevamente a la acción `update` y este controlador mostrará la plantilla `edit.html.erb` perteneciente al mismo controlador.

Si lo prefiere, puede utilizar un símbolo en lugar de una cadena para especificar la acción a renderizar:

```ruby
def update
  @book = Book.find(params[:id])
  if @book.update(book_params)
    redirect_to(@book)
  else
    render :edit
  end
end
```

### 2.2.2 Procesamiento de un template de una acción desde otro controlador

¿Qué sucede si desea procesar una plantilla desde un controlador completamente distinto del que contiene el código de acción? También puede hacer eso con `render`, que acepta la ruta completa \(relativa a `app/views`\) de la plantilla a renderizar. Por ejemplo, si ejecuta código en un `AdminProductsController` que vive en `app/controllers/admin`, puede renderizar los resultados de una acción a una plantilla en `app/views/products` de esta manera:

```ruby
render "products/show"
```

Rails sabe que esta vista pertenece a un controlador diferente debido all carácter de barra diagonal incrustado en la cadena. Si desea ser explícito, puede utilizar la opción `:template` \(que se requería en Rails 2.2 y anteriores\):

```ruby
render template: "products/show"
```

### 2.2.3 Procesamiento de un archivo arbitrario

El método `render` también puede utilizar una vista que está totalmente fuera de la aplicación:

```ruby
render file: "/u/apps/warehouse_app/current/app/views/products/show"
```

La opción :file toma una ruta absoluta del sistema de archivos. Por supuesto, necesita tener derechos sobre la vista que está utilizando para procesar el contenido.

> El uso de la opción :file en combinación con la entrada de los usuarios puede dar lugar a problemas de seguridad ya que un atacante podría utilizar esta acción para acceder a archivos sensibles a la seguridad en su sistema de archivos.
>
> De forma predeterminada, el archivo se procesa utilizando el layout actual.
>
> Si está ejecutando Rails en Microsoft Windows, debería utilizar la opción :file para procesar un archivo, ya que los nombres de archivo de Windows no tienen el mismo formato que los nombres de archivo Unix.

### 2.2.4 Envolviéndolo

Las tres formas de representación anteriores \(renderizar otra plantilla dentro del controlador, renderizar una plantilla dentro de otro controlador y renderizar un archivo arbitrario en el sistema de archivos\) son en realidad variantes de la misma acción.

De hecho, en la clase `BooksController`, dentro de la acción `update` donde queremos renderizar la plantilla `edit` si el `book` no se actualiza correctamente, todas las siguientes llamadas de renderizarían se harán a la plantilla `edit.html.erb` que esta en `views/books`

```ruby
render :edit
render action: :edit
render "edit"
render "edit.html.erb"
render action: "edit"
render action: "edit.html.erb"
render "books/edit"
render "books/edit.html.erb"
render template: "books/edit"
render template: "books/edit.html.erb"
render "/path/to/rails/app/views/books/edit"
render "/path/to/rails/app/views/books/edit.html.erb"
render file: "/path/to/rails/app/views/books/edit"
render file: "/path/to/rails/app/views/books/edit.html.erb"
```

Cuál utiliza usted es realmente una cuestión de estilo y de convención, pero la regla de oro es utilizar el más simple y que tiene sentido para el código que usted está escribiendo.

### 

### 2.2.5 Uso de render con :inline

El método `render` puede prescindir de una vista completa, si usa la opción `:inline` para suministrar el `ERB` como parte de la llamada al método. Esto es perfectamente válido:

```ruby
render inline: "<% products.each do |p| %><p><%= p.name %></p><% end %>"
```

Rara vez hay una buena razón para usar esta opción. La mezcla de `ERB` en sus controladores derrota la orientación MVC de Rails y dificultará que otros desarrolladores sigan la lógica de su proyecto. Utilice una vista `erb` separada y en su lugar respectivo.

De forma predeterminada, el rendering inline utiliza `ERB`. Puede forzarlo a que utilice `Builder` en su lugar con la opción `:type`.

```ruby
render inline: "xml.p {'Horrid coding practice!'}", type: :builder
```

### 2.2.6 Renderizado de texto

Puede enviar texto plano \(sin markups\) - de nuevo al navegador usando la opción `:plain` para renderizar:

```ruby
render plain: "OK"
```

> Renderizar texto plano es muy útil cuando respondes a un Ajax o a web services request que esperan algo distinto del `HTML` apropiado.

De forma predeterminada, si utiliza la opción `:plain`, el texto se procesa sin utilizar el layout actual. Si desea que Rails coloque el texto en el layout actual, debe agregar la opción layout `:true` y usar la extensión `.txt.erb` para el archivo de diseño.

### 2.2.7 Renderización de HTML

Puede enviar una cadena `HTML` de vuelta al navegador mediante la opción `:html` para renderizar:

```ruby
render html: "<strong>Not Found</strong>".html_safe
```

> Esto es útil cuando estás procesando un pequeño fragmento de código `HTML`. Sin embargo, es posible que desee considerar moverlo a un archivo template si el marcado es complejo.
>
> Cuando se utiliza la opción `:html`, las entidades HTML se escaparán si la cadena no está marcada como `HTML` seguro mediante el uso del método `html_safe`.

### 2.2.8 Renderización de JSON

`JSON` es un formato de datos JavaScript utilizado por muchas bibliotecas de `Ajax`. Rails tiene soporte incorporado para convertir objetos a `JSON` y hacer que `JSON` regrese al navegador:

```ruby
render json: @product
```

> No es necesario llamar al método `to_json` en el objeto que desea procesar. Si utiliza la opción `:json`, el `render` llamará automáticamente a `to_json` por usted.

### 2.2.9 Rendering XML

Rails también tiene soporte incorporado para convertir objetos a `XML` y devolver ese `XML` a quien lo llama:

```ruby
render xml: @product
```

> No es necesario llamar a `to_xml` en el objeto que desea procesar. Si utiliza la opción `:xml`, `render` le llamará automáticamente a `to_xml` por usted.

### 2.2.10 Renderizado de Vanilla JavaScript

Rails puede hacer Vanilla JavaScript :

```ruby
render js: "alert('Hello Rails');"
```

Esto enviará la cadena suministrada al navegador con un MIME de typo `text/javascript`.

### 2.2.11 Renderización de un body sin procesar

Puede enviar un contenido sin formato de nuevo al navegador, sin establecer ningún tipo de contenido, utilizando la opción `:body` para renderizar:

```ruby
render body: "raw"
```

> Esta opción sólo debe utilizarse si no le preocupa el tipo de contenido de la respuesta. Usar `:plain` o `:html` podría ser más apropiado para la mayor parte del tiempo.
>
> A menos que se sobre-escriba, la respuesta devuelta de esta opción de `render` será `text/html`, ya que es el tipo de contenido predeterminado de la respuesta de Action Dispatch.

### 2.2.12 Opciones para el renderizado

Las llamadas al método `render` generalmente aceptan cinco opciones:

* :content\_type
* :layout
* :location
* :status
* :formats

#### 2.2.12.1 La opción :content\_type

De forma predeterminada, Rails entregará los resultados de una operación de renderizado con el tipo de contenido MIME `text/html` \(o `application/json` si utiliza la opción `:json` o `application/xml` para la opción `:xml`\). Hay ocasiones en las que puede que desee cambiar esto, y puede hacerlo estableciendo la opción `:content_type`:

```ruby
render file: filename, content_type: "application/rss"
```

#### 2.2.12.2 La opción :layout

Con la mayoría de las opciones de `render`, el contenido renderizado se muestra como parte del layout actual. Aprenderá más sobre los layput y cómo utilizarlos más adelante en esta guía.

Puede utilizar la opción `:layout` para indicar a Rails que utilice un archivo específico como el layout de la acción actual:

```ruby
render layout: "special_layout"
```

También puede indicarle a Rails que lo procese sin ningún layout en absoluto:

```ruby
render layout: false
```

#### 2.2.12.3 La opción :location

Puedes utilizar la opción `:location` para establecer el encabezado `HTTP Location`

```ruby
render xml: photo, location: photo_url(photo)
```

#### 2.2.12.4 La opción :status

Rails generará automáticamente una respuesta con el código de estado `HTTP` correcto \(en la mayoría de los casos, esto es 200 OK\). Puede usar la opción `:status` para cambiar esto:

```ruby
render status: 500
render status: :forbidden
```

Rails entiende los códigos numéricos de estado y los símbolos correspondientes que se muestran a continuación.

| Clase de Respuesta | Codigo de Status HTTP | Símbolo |
| :--- | :--- | :--- |
| Informacional | 100 | :continue |
|  | 101 | :switching\_protocols |
|  | 102 | :processing |
| Success | 200 | :ok |
|  | 201 | :created |
|  | 202 | :accepted |
|  | 203 | :non\_authoritative\_information |
|  | 204 | :no\_content |
|  | 205 | :reset\_content |
|  | 206 | :partial\_content |
|  | 207 | :multi\_status |
|  | 208 | :already\_reported |
|  | 226 | :im\_used |
| Redirection | 300 | :multiple\_choices |
|  | 301 | :moved\_permanently |
|  | 302 | :found |
|  | 303 | :see\_other |
|  | 304 | :not\_modified |
|  | 305 | :use\_proxy |
|  | 307 | :temporary\_redirect |
|  | 308 | :premanent\_redirect |
| Client Error | 400 | :bad\_request |
|  | 401 | :unauthorized |
|  | 402 | :payment\_required |
|  | 403 | :forbidden |
|  | 404 | :not\_found |
|  | 405 | :method\_not\_allowed |
|  | 406 | :not\_acceptable |
|  | 407 | :proxy\_authentication\_required |
|  | 408 | :request\_timeout |
|  | 409 | :conflict |
|  | 410 | :gone |
|  | 411 | :length\_required |
|  | 412 | :precondition\_failed |
|  | 413 | :payload\_too\_large |
|  | 414 | :uri\_too\_long |
|  | 415 | :unsupported\_media\_type |
|  | 416 | :range\_not\_satisfiable |
|  | 417 | :expectation\_failed |
|  | 422 | :unprocessable\_entity |
|  | 423 | :locked |
|  | 424 | :failed\_dependency |
|  | 426 | :upgrade\_required |
|  | 428 | :precondition\_required |
|  | 429 | :too\_many\_requests |
|  | 431 | :request\_header\_fields\_too\_large |
| Server Error | 500 | :internal\_server\_error |
|  | 501 | :not\_implemented |
|  | 502 | :bad\_gateway |
|  | 503 | :service\_unavailable |
|  | 504 | :gateway\_timeout |
|  | 505 | :http\_version\_not\_supported |
|  | 506 | :variant\_also\_negotiates |
|  | 507 | :insufficient\_storage |
|  | 508 | :loop\_detected |
|  | 510 | :not\_extended |
|  | 511 | :network\_authentication\_required |

> Si intenta procesar el contenido junto con un código de estado sin contenido \(100-199, 204, 205 o 304\), se eliminará de la respuesta.

#### 2.2.12.5 La opción :formats

Rails utiliza el formato especificado en la solicitud \(o `:html`de forma predeterminada\). Puede cambiar esto pasando la opción `:formats` con un símbolo o un array:

```ruby
render formats: :xml
render formats: [:json, :xml]
```

Si no existe una plantilla con el formato especificado, se genera un error `ActionView::MissingTemplate`.

### 2.2.13 Búsqueda de layouts

Para encontrar el layout actual, Rails primero busca un archivo en `app/views/layouts` con el mismo nombre de base que el controlador. Por ejemplo, las acciones de renderización de la clase `PhotosController` usarán `app/views/layouts/photos.html.erb` \(o `app/views/layouts/photos.builder`\). Si no existe un layout específico para el controlador, rails utilizará `app/views/layouts/application.html.erb` o `app/views/layouts/application.builder`. Si no existe un diseño `.erb`, rails utilizará un diseño `.builder` si existe. Rails también proporciona varias maneras de asignar con mayor precisión layouts específicos a controladores y acciones individuales.

#### 2.2.13.1 Especificación de layouts para controladores

Puede reemplazar las convenciones de layout predeterminadas en sus controladores mediante la declaración de un layout. Por ejemplo:

```ruby
class ProductsController < ApplicationController
  layout "inventory"
  #...
end
```

Con esta declaración, todas las vistas generadas por el `ProductsController` utilizarán `app/views/layouts/inventory.html.erb` como su layout.

Para asignar un layout específico para toda la aplicación, utilice una declaración de layout en su clase `ApplicationController`:

```ruby
class ApplicationController < ActionController::Base
  layout "main"
  #...
end
```

Con esta declaración, todas las vistas de toda la aplicación utilizarán `app/views/layouts/main.html.erb` para su layout.

#### 2.2.13.2 Elección de layouts en tiempo de ejecución

Puede utilizar un símbolo para aplazar la elección del layout hasta que se procese una solicitud:

```ruby
class ProductsController < ApplicationController
  layout :products_layout

  def show
    @product = Product.find(params[:id])
  end

  private
    def products_layout
      @current_user.special? ? "special" : "products"
    end

end
```

Ahora, si el usuario actual es un usuario especial, verá un layout especial cuando acceda a un producto.

Incluso puede utilizar un método inline, como un `Proc`, para determinar el layout. Por ejemplo, si pasa un objeto `Proc`, al bloque que le da el `Proc` se le dará la instancia del controlador, por lo que el layout se puede determinar sobre la base de la solicitud actual:

```ruby
class ProductsController < ApplicationController
  layout Proc.new { |controller| controller.request.xhr? ? "popup" : "application" }
end
```

#### 2.2.13.3 Layouts condicionales

Los layouts especificados a nivel de controlador admiten las opciones `:only` y `:except`. Estas opciones llevan un nombre de método o un array de nombres de métodos que corresponden a nombres de métodos dentro del controlador:

```ruby
class ProductsController < ApplicationController
  layout "product", except: [:index, :rss]
end
```

Con esta declaración, el layout del producto se utilizaría para todo, excepto los métodos `rss` e `index`.

#### 2.2.13.4 Herencia de layouts

Las declaraciones de layouts caen en cascada hacia abajo en la jerarquía y las declaraciones de layouts más específicas siempre anulan las declaraciones más generales. Por ejemplo:

application\_controller.rb

```ruby
class ApplicationController < ActionController::Base
  layout "main"
end
```

articles\_controller.rb

```ruby
class ArticlesController < ApplicationController
end
```

special\_articles\_controller.rb

```ruby
class SpecialArticlesController < ArticlesController
  layout "special"
end
```

old\_articles\_controller.rb

```ruby
class OldArticlesController < SpecialArticlesController
  layout false

  def show
    @article = Article.find(params[:id])
  end

  def index
    @old_articles = Article.older
    render layout: "old"
  end
  # ...
end
```

En esta aplicación:

* En general, las vistas se mostrarán en el layout principal
* `ArticlesController#index` usará el layout principal
* `SpecialArticlesController#index` usará el layout especial
* `OldArticlesController#show` no utilizará ningún layout en absoluto
* `OldArticlesController#index` utilizará el layout antiguo.

#### 2.2.13.5 Herencia de plantilla

Similar a la lógica de herencia de layout, si un template o partial no se encuentra en la ruta convencional, el controlador buscará una plantilla o parcial para procesar en su cadena de herencia. Por ejemplo:

```ruby
# in app/controllers/application_controller
class ApplicationController < ActionController::Base
end
 
# in app/controllers/admin_controller
class AdminController < ApplicationController
end
 
# in app/controllers/admin/products_controller
class Admin::ProductsController < AdminController
  def index
  end
end
```

El orden de búsqueda de la acción index de `admin/products#index `será:

* app/views/admin/products/
* app/views/admin/
* app/views/application/

Esto hace que `app/views/application/` sea un gran lugar para tus patials compartidos, que luego se pueden renderizar en tu `ERB` como tal:

```ruby
<%# app/views/admin/products/index.html.erb %>
<%= render @products || "empty_list" %>
 
<%# app/views/application/_empty_list.html.erb %>
There are no items in this list <em>yet</em>.
```

#### 2.2.14 Evitar errores de procesamiento doble

Tarde o temprano, la mayoría de los desarrolladores de Rails verán el mensaje de error "Can only render or redirect once per action". Si bien esto es molesto, es relativamente fácil de arreglar. Por lo general, esto ocurre debido a un malentendido fundamental de la forma en que funciona.

Por ejemplo, aquí hay un código que activará este error:

```ruby
def show
  @book = Book.find(params[:id])
  if @book.special?
    render action: "special_show"
  end
  render action: "regular_show"
end
```

Si `@book.special?` se evalúa como `true`, Rails iniciará el proceso de renderizado para volcar la variable `@book` en la vista `special_show`. Pero esto no detendrá el resto del código en la acción `show`, y cuando Rails llegue al final de la acción, comenzará a mostrar la vista `regular_show` y lanzará un error. La solución es simple: asegúrese de que sólo tiene una llamada para procesar o redirigir en una ruta de un solo código. Una cosa que puede ayudar es `return`. He aquí una versión parcheada del método:

```ruby
def show
  @book = Book.find(params[:id])
  if @book.special?
    render action: "special_show" and return
  end
  render action: "regular_show"
end
```

Asegúrese de usar `and return`  en lugar solo `return` porque `and return` no funcionará debido a la precedencia del operador en el lenguaje Ruby

Tenga en cuenta que el renderizado implícito realizado por ActionController detecta si `render` se ha llamado, por lo que lo siguiente funcionará sin errores:

```ruby
def show
  @book = Book.find(params[:id])
  if @book.special?
    render action: "special_show"
  end
end
```

Esto hará que se renderice `book` con `special?` establecido con la plantilla `special_show`, mientras que otros `books` se procesarán con la plantilla de `show` predeterminada.

