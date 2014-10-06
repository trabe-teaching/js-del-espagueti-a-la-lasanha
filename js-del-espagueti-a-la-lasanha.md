title: "JS: del espagueti a la lasaña, paso a paso"
output: index.html
controls: false
progress: false
theme: trabe-theme

-- cover
<script src="https://code.jquery.com/jquery-2.1.1.min.js"></script>

# JS: del espagueti<br/>a la lasaña,<br/>paso a paso

## http://ir.gl/jsesalas

-- image
![Who we are](images/who_we_are.png)

-- image

![Trabe](images/trabe.png)

<!-- a qué nos dedicamos, qué hacemos para quién, tecnologías, etc -->


--

## Técnicas para<br/>estructurar código JS
### (con ejemplos _ad hoc_)

--

## Ideas básicas

### JS no es DOM
### jQuery no es JS


-- code

## Los comienzos

```xml


<a onclick="return confirm('Are you sure?');"
   href="/delete">Delete everything</a>
```

-- code

## Primeros pasos

```xml


<script>
  confirmDelete = function() {
    return confirm('Are you sure?');
  }
</script>

<a onclick="confirmDelete" href="/delete">Delete everything</a>
```

-- code

## Segundos pasos

```xml


<a id="delete-link" href="/delete">Delete everything</a>

<script>
  // Order matters
  document.getElementById('delete-link').onclick = function() {
    return confirm('Are you sure?');
  }
</script>

```

--

## Infinitos pasos,<br/>todos mal (o regular)

### Browser wars (DOM APIs' hell)
### Polución del espacio global (modularidad cero)
### ¿Rendimiento? ¿eso qué es?

--

## Librerías to the rescue

### Prototype
### Dojo
### MooTools
### YUI
### pero sobre todo __jQuery__


-- code
## The way of the jQuery

```xml

<script>
$(function() {
  $('.delete-link').on('click', function(event) {
    if (!confirm('Are you sure?')) {
      event.preventDefault();
      alert("Saved by the bell!");
    }
  });
});
</script>

<a class="delete-link" href="/delete_all">Delete everything</a>
<a class="delete-link" href="/delete_some">Delete something</a>
```

--

## JS y jQuery correcto, pero...

### No es eficiente
### No es modular
### Acoplado al DOM<br/>(no resiste modificaciones)
### Un pelín espagueti


-- section

## Recetas para hacer lasaña
### (añadir una capa más)

--

## No usar el tag &lt;script&gt;<br/>con contenido

--

## Choose the most convenient

### Múltiples src<br>(devel only y ojo a las dependencias)
### Preprocesadores y empaquetado
### Gestores de dependencias<br>(carga dínamica)

--

## IIFEs y Module pattern
### No polucionar el espacio global
### Encapsular estado y publicar APIs

-- code
## IIFE + Module Pattern

```javascript
var counter = (function() {
  var counter = 0;

  var inc = function() {
    counter += 1;
  };

  var value = function() {
    return counter;
  }

  return {
    inc: inc,
    value: value,
  }
})();

counter.inc();
console.log(counter.value()); // 1
```

--

## No usar callbacks inline
### (dentro de un orden ;)

-- code
## Espagueti (anonymous) callbacks
```javascript


$('.link').on('click', function(event) {
  event.preventDefault();
  $.get('/data.html', function(html) {
    $('.target').html(html);
  };
});
```

-- code
## Callback functions

```javascript


 var loadCallback = function(html) {
   $('.target').html(html);
 };

 var clickCallback = function(event) {
   event.preventDefault();
   $.get('/data.html', loadCallback);
 };

 $('link').on('click', clickCallback);
```
--

## Usar promesas
### (no solo para llamadas AJAX)

-- code
## Promesa AJAX
```javascript


var clickCallback = function(event) {
  event.preventDefault();
  var promise = $.get('/data.html');
  promise.done(doneCallback);
}

var doneCallback = function(html) {
  $('.target').html(html);
}

$('.link').on('click', clickCallback);
```

--

## No veo la diferencia


![](http://rack.0.mshcdn.com/media/ZgkyMDEzLzA2LzEzLzM3L0FkdmVudHVyZVRpLjM4MzhmLmdpZgpwCXRodW1iCTEyMDB4OTYwMD4/16390650/874/Adventure-Time.gif)

--

## Hay diferencias
### el evento es un suceso
### la promesa es un proceso

--

## Independencia del momento
### el evento no la tiene
### la promesa sí
### llamada vs whatsapp

-- code
## ¿seudocódigo? de inicialización + evento

```javascript



service.initialize = function() {
  async_code_block {
    trigger('init');
  }
};

service.initialize();

// ... unknown amount of time passes ...

service.on('init', callback); // maybe called
```

-- code
## ¿seudocódigo? de inicialización + promesa

```javascript



service.initialize = function() {
  this.promise = new Promise();
  async_code_block {
    this.promise.resolve();
  }
}

service.initialize();

// ... unknown amount of time passes ...

service.promise.done(callback); // always called

```


--

## Más diferencias
### las promesas (procesos) pueden encadenarse y paralelizarse

-- code
## Peticiones paralelas: the shitty way
```javascript

var loaded = 0, data1, data2;

var doneCallback = function() {
  if (loaded === 2) {
    console.log(data1, data2);
  }
}

$.get('/first_data.json', function(data) {
  data1 = data;
  loaded += 1;
  doneCallback();
})

$.get('/second_data.json', function(data) {
  data2 = data;
  loaded += 1;
  doneCallback();
})
```
-- code
## Peticiones paralelas: the cool way

```javascript



var doneCallback = function(data1, data2) {
  console.log(data1, data2);
})

var promise1 = $.get('/first_data.json');
var promise2 = $.get('/second_data.json');

$.when(promise1, promise2).then(doneCallback);
```

--

## Delegación de eventos
### Los eventos del DOM<br/>fluyen jerárquicamente

-- code

## Handlers and callbacks

```javascript

// n handlers, n callbacks
$('a').on('click', function() {
  // ...
});


var callback = function(event) {
  // ...
}

// n handlers, 1 callback
$('a').on('click', callback);

// 1 handler, 1 callback
$('body').on('click', 'a', callback);
```
--

## Desacoplarnos<br/>del DOM
### (desacoplarse del markup)

--

## Un ejemplo
### Componente que cuenta clicks<br/>(¡qué original!)

-- code
## El HTML
```xml



<p class="counter">0</p>
<a class="countable" href="#">Link 1</a>
<a class="countable" href="#">Link 2</a>


```
-- code
## The fast way: funciona, pero...

```javascript



$(function() {
  var count = 0,
      $counter = $('.counter');

  $('.countable').on('click', function() {
    count += 1;
    $counter.html(count);
  });
});

```

--
## Seleccionar el nodo adecuado
### (adecuadamente)

-- code
## En busca del selector perdido

```javascript


$(function() {
  $('#countable').on ... // Mal

  $('.countable').on ... // Regular

  $('.js-countable').on ... // Algo mejor

  $('[data-role~=countable]').on ...  // Perito!
});

```
-- code
## El HTML usando data-xxx

```xml




<p data-role="counter">0</p>
<a data-role="countable" href="#">Link 1</a>
<a data-role="countable" href="#">Link 2</a>
```

-- code
## JS usando data-xxx

```javascript



$(function() {
  var count = 0,
      $counter = $('[data-role~=counter]');

  $('[data-role=countable]').on('click', function() {
    count += 1;
    $counter.html(count);
  });
});

```
--

## Separemos la lógica<br/>en un componente
### No nos aisla del DOM,<br/>pero nos ayudará luego :)
### Y hará nuestro código reusable :D


-- code
## Un componente
```javascript

var Counter = function($counter, $links) {
  this.count = 0;
  this.$counter = $counter;
  this.$links = $links;

  var that = this;

  $links.on('click', function() {
    that.count += 1;
    that.$counter(that.count);
  };
};

$(function() {
  new Counter($('[data-role~=counter]'),
    $('[data-role~=countable]'));
});

```

--

## Seguimos un poco<br/>pegados al DOM
### añadamos otra capa

-- code
## Un wrapper para encapsular la manipulación DOM
```javascript



var CounterDom = function($node) {
  this.$node = $node;
};

CounterDom.prototype.setCount = function(count) {
  this.$node.html(count);
}
```

-- code
## Usando el wrapper

```javascript

var Counter = function(counterDom, $links) {
  this.count = 0;
  this.counterDom = counterDom;
  this.$links = $links;

  var that = this;

  $links.on('click', function() {
    that.count += 1;
    that.counterDom.setCount(this.count);
  };
};

$(function() {
  var counterDom = new CounterDom($('[data-role~=counter]'));
  new Counter(counterDom, $('[data-role~=countable]'));
});
```

-- code
## Un wrapper para encapsular eventos del DOM

```javascript



var CountableDom = function($node) {
  this.$node = $node;
};

CountableDom.prototype.onInteraction = function(callback) {
  this.$node.on('click', function(event) { callback(); }
}
```

-- code
## Usando el wrapper

```javascript

var Counter = function(counterDom, countableDom) {
  this.count = 0;
  this.counterDom = counterDom;
  this.countableDom = countableDom;

  var that = this;

  countableDom.onInteraction(function() {
    that.count += 1;
    that.counterDom.setCount(this.count);
  });
};

$(function() {
  var counterDom = new CounterDom($('[data-role~=counter]'));
  var countableDom = new CountableDom($('[data-role~=countable]'));
  new Counter(counterDom, countableDom);
});
```
--

## Ventajas
### No depender de jQuery ;)
### Testear sin depender del DOM
### DOM intercambiable

-- code sligthly-smaller
## Test al rico mock de piña para el niño y la niña
```javascript
describe('Counter', function() {

  var countableDomMock = {
    onInteraction : function(callback) {
      this.cb = callback;
    }
  }

  var counterDomMock = {
    setCount: function(value) {
      this.value = value;
    }
  }

  counter = new Counter(counterDomMock, countableDomMock);

  it('counts clicks', function() {
    countableDomMock.cb(); // mock interaction
    expect(counterDomMock.value).toEqual(1);
  });
});
```

-- code
## Checkboxes en lugar de enlaces

```xml

<p data-role="counter">0</p>
<label>
  <input type="checkbox" data-role="countable"/> Check 1
</label>
<label>
  <input type="checkbox" data-role="countable"/> Check 2
</label>
```

```javascript


var CountableDom = function($node) {
  this.$node = $node;
};

CountableDom.prototype.onInteraction = function(callback) {
  this.$node.on('change', function(event) { callback(); });
};
```

--

## Relanzar eventos
### Evento DOM => Evento Custom
### Alternativa para<br/>desacoplarse de los eventos

-- code
## Relanzando eventos
```javascript

var CountableDom = function($node) {
    this.$node = $node;
    this.$node.on('click', function() {
      $node.trigger('interaction');
    });
};

CountableDom.prototype.on = function() {
  this.$node.on.apply(Array.prototype.slice.call(arguments));
}

var Counter = function(counterDom, countableDom) {
  // ...

  countableDom.on('interaction', function() {
    // ...
  });
};

```

--

## One more step
### Aislar el modelo

-- code
## Modelo: ¿POJsO?

```javascript


var Count = function(initialValue) {
  this.count = initialValue;
};

Count.prototype.inc = function() {
  this.count += 1;
}

Count.prototype.value = function() {
  return this.count;
}
```

-- code
## Usando el modelo

```javascript

var Counter = function(count, counterDom, countableDom) {
  this.count = count;
  // ...

  countableDom.onInteraction(function() {
    that.count.increment();
    that.counterDom.setCount(that.count.value());
  });
};

$(function() {
  var count = new Count(0);
  var counterDom = new CounterDom($('[data-role~=counter]'));
  var countableDom = new CountableDom($('[data-role~=countable]'));
  new Counter(count, counterDom, countableDom);
});
```


-- two-columns

* Count
* XXXDom
* Counter
* MODEL
* VIEW
* CONTROLLER

--

## MVC FTW!

![](http://media.giphy.com/media/pa37AAGzKXoek/giphy.gif)

--

## Bus de eventos
### **WARNING**: handle with care

-- humongous-code

window.eventBus = $({});

-- code
## El modelo propaga por el bus sus cambios
```javascript



Count.prototype.inc = function() {
  this.count += 1;
  eventBus.trigger('count.inc', [ this.value() ]);
};

eventBus.on('count.inc', function(event, count) {
  console.log(count);
};
```

--

### **WARNING**: handle with care

<img src="http://www.vibewow.com/wp-content/uploads/2014/09/25.gif" width="550" />

--

## Observar el modelo
### get / set
### Object.observe (ECMAScript 6)

--

## Parametrización
### pasar datos del markup al JS
### data-xxx y JSON

-- code
## data-confirm-message

```javascript
$.fn.confirmable = function() {
  this.each(function() {
    var $el = $(this);
    $el.on('click', function(event) {
      var message = $el.data('confirm-message');
      if (!confirm(message)) { /* .. */ }
    });
  });
};

$(function() {
  $('[data-role~=confirmable]').confirmable()
});
```

```xml

<a data-role="confirmable"
   data-confirm-message="Are you sure?"
   href="/delete">Delete everything</a>
```

-- code
## Valor inicial del contador

```xml


<p data-role="counter" data-initial-count="10">0</p>
```

```javascript


CounterDom.prototype.getInitialCount = function() {
  this.$node.data('initial-count');
}

var Counter = function(counterDom, countableDom) {
  this.count = counterDom.getInitialCount();

  // ...
}

```

--

## No escribir HTML en el JS
### Usar plantillas

-- code
## HTML "a piñon"

```javascript




$.get('/data.json').done(function(data) {
  $placeHolder.html('<div><b>' + data.value + '</b></div>');
});

```

-- code
## Plantillas (Handlebars, por ejemplo)

```xml



<script id="my-template" type="text/x-handlebars-template">
  <div><b>{{ value }}</b></div>
</script>
```

```javascript



$.get('/data.json').done(function(data) {
  var source = $('#my-template').html(),
      template = Handlebars.compile(source);

  $placeHolder.html(template({value: data.value}));
});
```

--

## __Beware__
### Mejor no tener las<br/>plantillas en el HTML

--

## Plantillas JS

### Handlebars
### Mustache
### Eco
### Jade
### ...

-- section

## Técnicas Top Chef
### El detalle lo dejamos<br/>para el próximo capítulo

--

## Esteroides JS

### Underscore
### LoDash
### CoffeeScript, TypeScript ...
### ...

--

## Frameworks MVC

### Angular
### Ember
### React
### Backbone
### ...

--

## Dependencias

### AMD
### CommonJS
### Harmony

--

## Tools

### Node/Npm
### Bower vs Browserify
### Gulp vs Grunt
### Yeoman vs Lineman

--

## ECMAScript 6

### let
### fat arrow
### classes
### interpolation
### promises
### modules
### ...

http://www.2ality.com/2014/08/es6-today.html

-- section

# ¿Preguntas?

--
# ¡Viva la lasaña!

![](http://img.pandawhale.com/28306-Garfield-lasagna-gif-EzgK.gif)
