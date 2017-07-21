# 3-  Debugging con la gema byebug

Cuando su código se comporta de manera inesperada, puede intentar imprimirlo en los logs o en la consola para diagnosticar el problema. Desafortunadamente, hay momentos en que este tipo de seguimiento de errores no es eficaz para encontrar la causa raíz de un problema. Cuando realmente necesita viajar en su código fuente en ejecución, el depurador es su mejor compañero.

El depurador también puede ayudarte si quieres aprender sobre el código fuente de Rails pero no sabes por dónde empezar. Sólo tiene que depurar cualquier solicitud de su aplicación y utilizar esta guía para aprender a moverse en el código que ha escrito en el código Rails subyacente.

### 3.1 Configuración

Puede usar la gema byebug para establecer puntos de interrupción \(breakpoints\) y pasar a través del código en vivo en Rails. Para instalarlo, ejecute:

```ruby
$ gem install byebug
```

Dentro de cualquier aplicación Rails, puede invocar el depurador llamando al método byebug.

He aquí un ejemplo:

```ruby
class PeopleController < ApplicationController
  def new
    byebug
    @person = Person.new
  end
end
```

### 3.2 El Shell

Tan pronto como su aplicación llame al método byebug, el debugger se iniciará en una shell de debbug dentro de la ventana de terminal donde lanzó su servidor de aplicaciones, y se colocará en el prompt del debugger \(byebug\). Antes de la solicitud, se mostrará el código alrededor de la línea que está a punto de ejecutarse y la línea actual se marcará con '`=>`', así:

```ruby
[1, 10] in /PathTo/project/app/controllers/articles_controller.rb
    3:
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     byebug
=>  8:     @articles = Article.find_recent
    9:
   10:     respond_to do |format|
   11:       format.html # index.html.erb
   12:       format.json { render json: @articles }
 
(byebug)
```

Si llegaste allí por una solicitud del navegador, la pestaña del navegador que contiene la solicitud se congelará hasta que el depurador haya terminado y el trace haya terminado de procesar toda la solicitud.

Por ejemplo:

```ruby
=> Booting Puma
=> Rails 5.1.0 application starting in development on http://0.0.0.0:3000
=> Run `rails server -h` for more startup options
Puma starting in single mode...
* Version 3.4.0 (ruby 2.3.1-p112), codename: Owl Bowl Brawl
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://localhost:3000
Use Ctrl-C to stop
Started GET "/" for 127.0.0.1 at 2014-04-11 13:11:48 +0200
  ActiveRecord::SchemaMigration Load (0.2ms)  SELECT "schema_migrations".* FROM "schema_migrations"
Processing by ArticlesController#index as HTML
 
[3, 12] in /PathTo/project/app/controllers/articles_controller.rb
    3:
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     byebug
=>  8:     @articles = Article.find_recent
    9:
   10:     respond_to do |format|
   11:       format.html # index.html.erb
   12:       format.json { render json: @articles }
(byebug)
```

Ahora es el momento de explorar su aplicación. Un buen lugar para comenzar es pedirle al depurador ayuda. Tipea: help

```
(byebug) help
 
  break      -- Sets breakpoints in the source code
  catch      -- Handles exception catchpoints
  condition  -- Sets conditions on breakpoints
  continue   -- Runs until program ends, hits a breakpoint or reaches a line
  debug      -- Spawns a subdebugger
  delete     -- Deletes breakpoints
  disable    -- Disables breakpoints or displays
  display    -- Evaluates expressions every time the debugger stops
  down       -- Moves to a lower frame in the stack trace
  edit       -- Edits source files
  enable     -- Enables breakpoints or displays
  finish     -- Runs the program until frame returns
  frame      -- Moves to a frame in the call stack
  help       -- Helps you using byebug
  history    -- Shows byebug's history of commands
  info       -- Shows several informations about the program being debugged
  interrupt  -- Interrupts the program
  irb        -- Starts an IRB session
  kill       -- Sends a signal to the current process
  list       -- Lists lines of source code
  method     -- Shows methods of an object, class or module
  next       -- Runs one or more lines of code
  pry        -- Starts a Pry session
  quit       -- Exits byebug
  restart    -- Restarts the debugged program
  save       -- Saves current byebug session to a file
  set        -- Modifies byebug settings
  show       -- Shows byebug settings
  source     -- Restores a previously saved byebug session
  step       -- Steps into blocks or methods one or more times
  thread     -- Commands to manipulate threads
  tracevar   -- Enables tracing of a global variable
  undisplay  -- Stops displaying all or some expressions when program stops
  untracevar -- Stops tracing a global variable
  up         -- Moves to a higher frame in the stack trace
  var        -- Shows variables and its values
  where      -- Displays the backtrace
 
(byebug)
```

Para ver las diez líneas anteriores debe escribir `list- (or l-).`

```
(byebug) l-
 
[1, 10] in /PathTo/project/app/controllers/articles_controller.rb
   1  class ArticlesController < ApplicationController
   2    before_action :set_article, only: [:show, :edit, :update, :destroy]
   3
   4    # GET /articles
   5    # GET /articles.json
   6    def index
   7      byebug
   8      @articles = Article.find_recent
   9
   10      respond_to do |format|
```

De esta manera puede moverse dentro del archivo y ver el código por encima de la línea donde agregó la llamada byebug. Por último, para ver dónde se encuentra en el código de nuevo se puede escribir `list=`

```
(byebug) list=
 
[3, 12] in /PathTo/project/app/controllers/articles_controller.rb
    3:
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     byebug
=>  8:     @articles = Article.find_recent
    9:
   10:     respond_to do |format|
   11:       format.html # index.html.erb
   12:       format.json { render json: @articles }
(byebug)
```

### 3.3 El contexto

Cuando inicie la depuración de la aplicación, se colocará en diferentes contextos a medida que vaya a través de las diferentes partes del stack.

El depurador crea un contexto cuando se alcanza un punto de parada o un evento. El contexto tiene información sobre el programa suspendido que permite al depurador inspeccionar el stack del frame, evaluar variables desde la perspectiva del programa depurado y conocer el lugar donde se detiene el programa depurado.

En cualquier momento puede llamar al comando `backtrace` \(o su alias `where`\) para imprimir el backtrace de la aplicación. Esto puede ser muy útil para saber cómo llegaste donde estás. Si alguna vez se preguntó acerca de cómo obtuvo algún lugar en su código, entonces backtrace proporcionará la respuesta.

```
(byebug) where
--> #0  ArticlesController.index
      at /PathToProject/app/controllers/articles_controller.rb:8
    #1  ActionController::BasicImplicitRender.send_action(method#String, *args#Array)
      at /PathToGems/actionpack-5.1.0/lib/action_controller/metal/basic_implicit_render.rb:4
    #2  AbstractController::Base.process_action(action#NilClass, *args#Array)
      at /PathToGems/actionpack-5.1.0/lib/abstract_controller/base.rb:181
    #3  ActionController::Rendering.process_action(action, *args)
      at /PathToGems/actionpack-5.1.0/lib/action_controller/metal/rendering.rb:30
...

```

El frame actual está marcado con `-->`. Puede moverse a cualquier lugar que desee en este trace \(cambiando así el contexto\) utilizando el comando `frame n`, donde `n` es el número del frame especificado. Si lo hace, `byebug` mostrará su nuevo contexto.

```
(byebug) frame 2
 
[176, 185] in /PathToGems/actionpack-5.1.0/lib/abstract_controller/base.rb
   176:       # is the intended way to override action dispatching.
   177:       #
   178:       # Notice that the first argument is the method to be dispatched
   179:       # which is *not* necessarily the same as the action name.
   180:       def process_action(method_name, *args)
=> 181:         send_action(method_name, *args)
   182:       end
   183:
   184:       # Actually call the method associated with the action. Override
   185:       # this method if you wish to change how action methods are called,
(byebug)
```



