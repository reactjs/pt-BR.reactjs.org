---
id: integrating-with-other-libraries
title: Integrando com outras Bibliotecas
permalink: docs/integrating-with-other-libraries.html
---

React pode ser utilizado em qualquer aplicação web. Ele pode ser embutido em outras aplicações e, com um pouco de cuidado, outras aplicações podem ser incorporadas ao React. Este guia vai examinar alguns dos mais comuns usos de caso, focando na integracão com [jQuery](https://jquery.com/) e [Backbone](http://backbonejs.org/), mas as mesmas idéias podem ser aplicadas para integrar componentes com qualquer código existente.

## Integrando com Plugins de Manipulacão do DOM {#integrating-with-dom-manipulation-plugins}

React desconhece qualquer alteração feita no DOM fora do React. Ele determina as atualizacões com base na sua própria representacão interna, e se os mesmos nós do DOM forem manipulados por outras bibliotecas, React fica confuso e não sabe como proceder.

Isto não significa que é impossível ou mesmo necessariamente difícil de combinar React com outras maneiras de afetar o DOM, você apenas precisa estar atento ao que cada um está fazendo.

A maneira mais fácil de evitar conflitos é evitando com que o componente React atualize. Você pode fazer isto, renderizando elementos que o React não tem motivos para atualizar, como uma `<div />` vazia.

### Como Abordar o Problema {#how-to-approach-the-problem}

Para demonstrar isto, vamos esboçar um wrapper para um plugin jQuery genérico.

Nós vamos adicionar um [ref](/docs/refs-and-the-dom.html) para o elemento root no DOM. Dentro do `componentDidMount`, nós vamos ter a referência desse elemento para passar para o plugin jQuery.

Para evitar com que o React toque no DOM depois de montado, nós vamos retornar uma `<div />` vazia para o método `render()`. O elemento `<div />`, não possui propriedades ou filhos, assim o React não tem razão para atualiza-lo, deixando o plugin jQuery livre para gerenciar esta parte do DOM:

```js{3,4,8,12}
class SomePlugin extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.somePlugin();
  }

  componentWillUnmount() {
    this.$el.somePlugin('destroy');
  }

  render() {
    return <div ref={el => this.el = el} />;
  }
}
```

Note que nós definimos ambos `componentDidMount` e `componentWillUnmount` [métodos do ciclo de vida](/docs/react-component.html#the-component-lifecycle). Muitos plugins jQuery adicionam listeners de eventos para o DOM, e é importante removê-los no `componentWillUnmount`. Se o plugin não fornece um método para limpeza, você vai provavelmente ter que criar o seu próprio, lembrando de remover qualquer listener do plugin registrado para evitar vazamento de memória.

### Integrando com o plugin jQuery Chosen {#integrating-with-jquery-chosen-plugin}

Para um exemplo mais concreto desses conceitos, vamos escrever um wrapper mínimo para o plugin [Chosen](https://harvesthq.github.io/chosen/), which augments `<select>` inputs.

>**Nota:**
>
> Apenas porque isto é possível, não significa que este é a melhor maneira para apps React. Nós encorajamos você a utilizar componentes React quando você puder. Componentes React são fáceis de reutilizar em aplicacões React, e muitas vezes fornecem mais controle sobre seu comportamento e aparência.

Primeiro, vamos olhar o que Chosen faz no DOM.

Se você chama-lo em um nó DOM `<select>`, ele lê os atributos do nó DOM original, esconde-os com um estilo inline, e então adiciona um nó DOM separado com sua própria representacão visual, logo após o `<select>`. Em seguida, ele dispara um evento do jQuery, para notificar sobre as alteracões.

Vamos supor que esta é a API onde nós estamos nos esforcando para com o nosso `<Chosen>` React componente:

```js
function Example() {
  return (
    <Chosen onChange={value => console.log(value)}>
      <option>vanilla</option>
      <option>chocolate</option>
      <option>strawberry</option>
    </Chosen>
  );
}
```

Nós vamos implementá-lo como um [componente não controlado](/docs/uncontrolled-components.html) pela simplicidade.

Primeiro, vamos criar um component vazio com um método `render()` onde nós retornamos o `<select>` em volta de uma `<div>`

```js{4,5}
class Chosen extends React.Component {
  render() {
    return (
      <div>
        <select className="Chosen-select" ref={el => this.el = el}>
          {this.props.children}
        </select>
      </div>
    );
  }
}
```

Note como envolvemos o `<select>` em uma `<div>` extra. Isto é necessário porque Chosen vai adicionar um outro elemento DOM logo após o `<select>` que passamos para ele. Contudo, no que diz respeito ao React, `<div>` sempre tem apenas um único filho. Isto é como nós garantimos que as atualizacões do React não vão conflitar com o nó extra do DOM adicionado pelo Chosen. É importante que se você modificar o DOM fora do fluxo do React, você deve garantir que React não tem um motivo para acessar esses nós do DOM.

Em seguida, vamos implementar os métodos do ciclo de vida. Nós vamos precisar inicializar Chosen com a referência para o `<select>` no `componentDidMount`, e destruí-lo no `componentWillUnmount`.

```js{2,3,7}
componentDidMount() {
  this.$el = $(this.el);
  this.$el.chosen();
}

componentWillUnmount() {
  this.$el.chosen('destroy');
}
```

[**Experimente no Codepen**](http://codepen.io/gaearon/pen/qmqeQx?editors=0010)

Note que React não atribui nenhum significado especial para o campo `this.el`. Isto apenas funciona porque nós atribuimos anteriormente um valor para este campo, com um `ref` no método `render()`:

```js
<select className="Chosen-select" ref={el => this.el = el}>
```

Isto é suficiente para renderizar o nosso componente, mas também queremos ser notificados quando os valores mudarem. Para fazer isto, vamos ouvir os eventos de `change` do jQuery no `<select>` controlado pelo Chosen.

Nós não vamos passar `this.props.onChange` diretamente para o Chosen porque as propriedades do componente, podem mudar ao longo do tempo e isto inclui os handlers de evento. Como alternativa, vamos declarar um método `handleChange()` que chama o `this.props.onChange` e subscreve o evento `change` do jQuery:

```js{5,6,10,14-16}
componentDidMount() {
  this.$el = $(this.el);
  this.$el.chosen();

  this.handleChange = this.handleChange.bind(this);
  this.$el.on('change', this.handleChange);
}

componentWillUnmount() {
  this.$el.off('change', this.handleChange);
  this.$el.chosen('destroy');
}

handleChange(e) {
  this.props.onChange(e.target.value);
}
```

[**Experimente no CodePen**](http://codepen.io/gaearon/pen/bWgbeE?editors=0010)

Finally, there is one more thing left to do. In React, props can change over time. For example, the `<Chosen>` component can get different children if parent component's state changes. This means that at integration points it is important that we manually update the DOM in response to prop updates, since we no longer let React manage the DOM for us.

Chosen's documentation suggests that we can use jQuery `trigger()` API to notify it about changes to the original DOM element. We will let React take care of updating `this.props.children` inside `<select>`, but we will also add a `componentDidUpdate()` lifecycle method that notifies Chosen about changes in the children list:

```js{2,3}
componentDidUpdate(prevProps) {
  if (prevProps.children !== this.props.children) {
    this.$el.trigger("chosen:updated");
  }
}
```

This way, Chosen will know to update its DOM element when the `<select>` children managed by React change.

The complete implementation of the `Chosen` component looks like this:

```js
class Chosen extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.chosen();

    this.handleChange = this.handleChange.bind(this);
    this.$el.on('change', this.handleChange);
  }
  
  componentDidUpdate(prevProps) {
    if (prevProps.children !== this.props.children) {
      this.$el.trigger("chosen:updated");
    }
  }

  componentWillUnmount() {
    this.$el.off('change', this.handleChange);
    this.$el.chosen('destroy');
  }
  
  handleChange(e) {
    this.props.onChange(e.target.value);
  }

  render() {
    return (
      <div>
        <select className="Chosen-select" ref={el => this.el = el}>
          {this.props.children}
        </select>
      </div>
    );
  }
}
```

[**Try it on CodePen**](http://codepen.io/gaearon/pen/xdgKOz?editors=0010)

## Integrating with Other View Libraries {#integrating-with-other-view-libraries}

React can be embedded into other applications thanks to the flexibility of [`ReactDOM.render()`](/docs/react-dom.html#render).

Although React is commonly used at startup to load a single root React component into the DOM, `ReactDOM.render()` can also be called multiple times for independent parts of the UI which can be as small as a button, or as large as an app.

In fact, this is exactly how React is used at Facebook. This lets us write applications in React piece by piece, and combine them with our existing server-generated templates and other client-side code.

### Replacing String-Based Rendering with React {#replacing-string-based-rendering-with-react}

A common pattern in older web applications is to describe chunks of the DOM as a string and insert it into the DOM like so: `$el.html(htmlString)`. These points in a codebase are perfect for introducing React. Just rewrite the string based rendering as a React component.

So the following jQuery implementation...

```js
$('#container').html('<button id="btn">Say Hello</button>');
$('#btn').click(function() {
  alert('Hello!');
});
```

...could be rewritten using a React component:

```js
function Button() {
  return <button id="btn">Say Hello</button>;
}

ReactDOM.render(
  <Button />,
  document.getElementById('container'),
  function() {
    $('#btn').click(function() {
      alert('Hello!');
    });
  }
);
```

From here you could start moving more logic into the component and begin adopting more common React practices. For example, in components it is best not to rely on IDs because the same component can be rendered multiple times. Instead, we will use the [React event system](/docs/handling-events.html) and register the click handler directly on the React `<button>` element:

```js{2,6,9}
function Button(props) {
  return <button onClick={props.onClick}>Say Hello</button>;
}

function HelloButton() {
  function handleClick() {
    alert('Hello!');
  }
  return <Button onClick={handleClick} />;
}

ReactDOM.render(
  <HelloButton />,
  document.getElementById('container')
);
```

[**Try it on CodePen**](http://codepen.io/gaearon/pen/RVKbvW?editors=1010)

You can have as many such isolated components as you like, and use `ReactDOM.render()` to render them to different DOM containers. Gradually, as you convert more of your app to React, you will be able to combine them into larger components, and move some of the `ReactDOM.render()` calls up the hierarchy.

### Embedding React in a Backbone View {#embedding-react-in-a-backbone-view}

[Backbone](http://backbonejs.org/) views typically use HTML strings, or string-producing template functions, to create the content for their DOM elements. This process, too, can be replaced with rendering a React component.

Below, we will create a Backbone view called `ParagraphView`. It will override Backbone's `render()` function to render a React `<Paragraph>` component into the DOM element provided by Backbone (`this.el`). Here, too, we are using [`ReactDOM.render()`](/docs/react-dom.html#render):

```js{1,5,8,12}
function Paragraph(props) {
  return <p>{props.text}</p>;
}

const ParagraphView = Backbone.View.extend({
  render() {
    const text = this.model.get('text');
    ReactDOM.render(<Paragraph text={text} />, this.el);
    return this;
  },
  remove() {
    ReactDOM.unmountComponentAtNode(this.el);
    Backbone.View.prototype.remove.call(this);
  }
});
```

[**Try it on CodePen**](http://codepen.io/gaearon/pen/gWgOYL?editors=0010)

It is important that we also call `ReactDOM.unmountComponentAtNode()` in the `remove` method so that React unregisters event handlers and other resources associated with the component tree when it is detached.

When a component is removed *from within* a React tree, the cleanup is performed automatically, but because we are removing the entire tree by hand, we must call this method.

## Integrating with Model Layers {#integrating-with-model-layers}

While it is generally recommended to use unidirectional data flow such as [React state](/docs/lifting-state-up.html), [Flux](http://facebook.github.io/flux/), or [Redux](http://redux.js.org/), React components can use a model layer from other frameworks and libraries.

### Using Backbone Models in React Components {#using-backbone-models-in-react-components}

The simplest way to consume [Backbone](http://backbonejs.org/) models and collections from a React component is to listen to the various change events and manually force an update.

Components responsible for rendering models would listen to `'change'` events, while components responsible for rendering collections would listen for `'add'` and `'remove'` events. In both cases, call [`this.forceUpdate()`](/docs/react-component.html#forceupdate) to rerender the component with the new data.

In the example below, the `List` component renders a Backbone collection, using the `Item` component to render individual items.

```js{1,7-9,12,16,24,30-32,35,39,46}
class Item extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange() {
    this.forceUpdate();
  }

  componentDidMount() {
    this.props.model.on('change', this.handleChange);
  }

  componentWillUnmount() {
    this.props.model.off('change', this.handleChange);
  }

  render() {
    return <li>{this.props.model.get('text')}</li>;
  }
}

class List extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange() {
    this.forceUpdate();
  }

  componentDidMount() {
    this.props.collection.on('add', 'remove', this.handleChange);
  }

  componentWillUnmount() {
    this.props.collection.off('add', 'remove', this.handleChange);
  }

  render() {
    return (
      <ul>
        {this.props.collection.map(model => (
          <Item key={model.cid} model={model} />
        ))}
      </ul>
    );
  }
}
```

[**Try it on CodePen**](http://codepen.io/gaearon/pen/GmrREm?editors=0010)

### Extracting Data from Backbone Models {#extracting-data-from-backbone-models}

The approach above requires your React components to be aware of the Backbone models and collections. If you later plan to migrate to another data management solution, you might want to concentrate the knowledge about Backbone in as few parts of the code as possible.

One solution to this is to extract the model's attributes as plain data whenever it changes, and keep this logic in a single place. The following is [a higher-order component](/docs/higher-order-components.html) that extracts all attributes of a Backbone model into state, passing the data to the wrapped component.

This way, only the higher-order component needs to know about Backbone model internals, and most components in the app can stay agnostic of Backbone.

In the example below, we will make a copy of the model's attributes to form the initial state. We subscribe to the `change` event (and unsubscribe on unmounting), and when it happens, we update the state with the model's current attributes. Finally, we make sure that if the `model` prop itself changes, we don't forget to unsubscribe from the old model, and subscribe to the new one.

Note that this example is not meant to be exhaustive with regards to working with Backbone, but it should give you an idea for how to approach this in a generic way:

```js{1,5,10,14,16,17,22,26,32}
function connectToBackboneModel(WrappedComponent) {
  return class BackboneComponent extends React.Component {
    constructor(props) {
      super(props);
      this.state = Object.assign({}, props.model.attributes);
      this.handleChange = this.handleChange.bind(this);
    }

    componentDidMount() {
      this.props.model.on('change', this.handleChange);
    }

    componentWillReceiveProps(nextProps) {
      this.setState(Object.assign({}, nextProps.model.attributes));
      if (nextProps.model !== this.props.model) {
        this.props.model.off('change', this.handleChange);
        nextProps.model.on('change', this.handleChange);
      }
    }

    componentWillUnmount() {
      this.props.model.off('change', this.handleChange);
    }

    handleChange(model) {
      this.setState(model.changedAttributes());
    }

    render() {
      const propsExceptModel = Object.assign({}, this.props);
      delete propsExceptModel.model;
      return <WrappedComponent {...propsExceptModel} {...this.state} />;
    }
  }
}
```

To demonstrate how to use it, we will connect a `NameInput` React component to a Backbone model, and update its `firstName` attribute every time the input changes:

```js{4,6,11,15,19-21}
function NameInput(props) {
  return (
    <p>
      <input value={props.firstName} onChange={props.handleChange} />
      <br />
      My name is {props.firstName}.
    </p>
  );
}

const BackboneNameInput = connectToBackboneModel(NameInput);

function Example(props) {
  function handleChange(e) {
    props.model.set('firstName', e.target.value);
  }

  return (
    <BackboneNameInput
      model={props.model}
      handleChange={handleChange}
    />
  );
}

const model = new Backbone.Model({ firstName: 'Frodo' });
ReactDOM.render(
  <Example model={model} />,
  document.getElementById('root')
);
```

[**Try it on CodePen**](http://codepen.io/gaearon/pen/PmWwwa?editors=0010)

This technique is not limited to Backbone. You can use React with any model library by subscribing to its changes in the lifecycle methods and, optionally, copying the data into the local React state.
