---
id: forms
title: Formulários
permalink: docs/forms.html
prev: lists-and-keys.html
next: lifting-state-up.html
redirect_from:
  - "tips/controlled-input-null-value.html"
  - "docs/forms-zh-CN.html"
---

<<<<<<< HEAD
Os elementos de formulário HTML funcionam de maneira um pouco diferente de outros elementos DOM no React, porque os elementos de formulário mantêm naturalmente algum estado interno. Por exemplo, este formulário em HTML puro aceita um único nome:
=======
HTML form elements work a bit differently from other DOM elements in React, because form elements naturally keep some internal state. For example, this form in plain HTML accepts a single name:
>>>>>>> 25f756d8e3800afb032cb31ce3626d6134e31e38

```html
<form>
  <label>
    Nome:
    <input type="text" name="name" />
  </label>
  <input type="submit" value="Enviar" />
</form>
```

Esse formulário tem o comportamento padrão do HTML de navegar para uma nova página quando o usuário enviar o formulário. Se você quer esse comportamento no React, ele simplesmente funciona. Mas na maioria dos casos, é conveniente ter uma função JavaScript que manipula o envio de um formulário e tem acesso aos dados que o usuário digitou nos inputs. O modo padrão de fazer isso é com uma técnica chamada "componentes controlados" (controlled components).

## Componentes Controlados (Controlled Components) {#controlled-components}

Em HTML, elementos de formulário como `<input>`, `<textarea>` e `<select>` normalmente mantêm seu próprio estado e o atualiza baseado na entrada do usuário. Em React, o estado mutável é normalmente mantido na propriedade state dos componentes e atualizado apenas com [`setState()`](/docs/react-component.html#setstate).

Podemos combinar os dois fazendo o estado React ser a "única fonte da verdade". Assim, o componente React que renderiza um formulário também controla o que acontece nesse formulário nas entradas subsequentes do usuário. Um input cujo o valor é controlado pelo React dessa maneira é chamado de "componente controlado" (controlled component).

Por exemplo, se quisermos que o exemplo anterior registre o nome quando ele for enviado, podemos escrever o formulário como um componente controlado:

```javascript{4,10-12,21,24}
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('Um nome foi enviado: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Nome:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Enviar" />
      </form>
    );
  }
}
```

[**Experimente no CodePen**](https://codepen.io/gaearon/pen/VmmPgp?editors=0010)

Como o atributo `value` é definido no nosso `<input type="text">`, o valor exibido sempre será o mesmo de `this.state.value`, fazendo com que o estado do React seja a fonte da verdade. Como o `handleChange` é executado a cada tecla pressionada para atualizar o estado do React, o valor exibido será atualizado conforme o usuário digita.

Com um componente controlado, o valor da entrada é sempre direcionado pelo estado React. Embora isso signifique que você precisa digitar um pouco mais de código, agora também pode passar o valor para outros elementos da interface do usuário ou redefini-lo de outros manipuladores de eventos.

## Tag textarea {#the-textarea-tag}

Em HTML, o texto de um elemento `<textarea>` é definido por seus filhos:

```html
<textarea>
  Apenas algum texto em uma área de texto
</textarea>
```

Em React, em vez disso, o `<textarea>` usa um atributo `value`. Desta forma, um formulário usando um `<textarea>` pode ser escrito de forma muito semelhante a um formulário que usa um input de linha única:

```javascript{4-6,12-14,26}
class EssayForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: 'Por favor, escreva uma dissertação sobre o seu elemento DOM favorito.'
    };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('Uma dissertação foi enviada: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Dissertação:
          <textarea value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Enviar" />
      </form>
    );
  }
}
```

Observe que `this.state.value` é inicializado no construtor, para que o textarea comece com algum texto.

## Tag select {#the-select-tag}

Em HTML, `<select>` cria uma lista suspensa (drop-down). Por exemplo, esse HTML cria uma lista suspensa de sabores:

```html
<select>
  <option value="laranja">Laranja</option>
  <option value="limao">Limão</option>
  <option selected value="coco">Coco</option>
  <option value="manga">Manga</option>
</select>
```

Note que a opção "coco" é selecionada por padrão, por causa do atributo `selected`. Em React, em vez de usar este atributo `selected`, usa-se um atributo `value` na raiz da tag `select`. Isso é mais conveniente em um componente controlado, porque você só precisa atualizá-lo em um só lugar. Por exemplo:

```javascript{4,10-12,24}
class FlavorForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: 'coco'};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('Seu sabor favorito é: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Escolha seu sabor favorito:
          <select value={this.state.value} onChange={this.handleChange}>
            <option value="laranja">Laranja</option>
            <option value="limao">Limão</option>
            <option value="coco">Coco</option>
            <option value="manga">Manga</option>
          </select>
        </label>
        <input type="submit" value="Enviar" />
      </form>
    );
  }
}
```

[**Experimente no CodePen**](https://codepen.io/gaearon/pen/JbbEzX?editors=0010)

No geral, isso faz com que as tags `<input type="text">`, `<textarea>` e `<select>` funcionem de forma muito semelhante - todos eles aceitam um atributo `value` que você pode usar para implementar um componente controlado.

> Nota
>
> Você pode passar um array para o atributo `value`, permitindo que você selecione várias opções em uma tag `select`:
>
>```js
><select multiple={true} value={['B', 'C']}>
>```

## Tag de entrada de arquivo (file input) {#the-file-input-tag}

Em HTML, o `<input type="file">` permite ao usuário escolher um ou mais arquivos de seu dispositivo para serem enviados para um servidor ou manipulados por JavaScript através da [File API](https://developer.mozilla.org/pt-BR/docs/Web/API/File/Using_files_from_web_applications).

```html
<input type="file" />
```

Como seu valor é de somente leitura, ele é um componente **não controlado** do React. Esses são discutidos junto a outros componentes não controlados [mais adiante na documentação](/docs/uncontrolled-components.html#the-file-input-tag).

## Manipulando Múltiplos Inputs {#handling-multiple-inputs}

Quando você precisa manipular múltiplos inputs controlados, você pode adicionar um atributo `name` a cada elemento e deixar a função manipuladora escolher o que fazer com base no valor de `event.target.name`.

Por exemplo:

```javascript{15,18,28,37}
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          Estão indo:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <br />
        <label>
          Número de convidados:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```

[**Experimente no CodePen**](https://codepen.io/gaearon/pen/wgedvV?editors=0010)

Observe como usamos a sintaxe ES6 [nomes de propriedades computados](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Inicializador_Objeto#Nomes_de_propriedades_computados) para atualizar a chave de estado correspondente ao nome de entrada fornecido:

```js{2}
this.setState({
  [name]: value
});
```

É equivalente a este código no ES5:

```js{2}
var partialState = {};
partialState[name] = value;
this.setState(partialState);
```

Além disso, como o `setState()` automaticamente [mescla um estado parcial ao estado atual](/docs/state-and-lifecycle.html#state-updates-are-merged), nós podemos chamá-lo apenas com as partes alteradas.

## Valor Nulo em um Input Controlado {#controlled-input-null-value}

A especificação de uma prop `value` em um [componente controlado](/docs/forms.html#controlled-components) impede que o usuário altere a entrada, a menos que você deseje. Se você especificou uma prop `value`, mas o input ainda é editável, você pode ter acidentalmente definido o `value` como `undefined` ou `null`.

O código a seguir demonstra isso. (O input é bloqueada no início, mas torna-se editável após um tempo.)

```javascript
ReactDOM.render(<input value="hi" />, mountNode);

setTimeout(function() {
  ReactDOM.render(<input value={null} />, mountNode);
}, 1000);

```

## Alternativas para Componentes Controlados {#alternatives-to-controlled-components}

Às vezes pode ser tedioso usar componentes controlados, porque você precisa escrever um manipulador de eventos para cada maneira que seus dados podem mudar e canalizar todo o estado do input através de um componente React. Isso pode se tornar particularmente irritante quando você está convertendo uma base de código preexistente para o React ou integrando um aplicativo React com uma biblioteca que não seja baseado em React. Nessas situações, talvez você queira verificar os [componentes não controlados](/docs/uncontrolled-components.html), uma técnica alternativa para implementar formulários de entrada.

## Soluções Completas {#fully-fledged-solutions}

Se você está procurando por uma solução completa, incluindo validação, manter o controle dos campos visualizados e lidar com o envio de formulários, o [Formik](https://jaredpalmer.com/formik) é uma das escolhas mais populares. No entanto, ele é construído sobre os mesmos princípios de componentes controlados e gerenciamento de estado - portanto, não negligencie o aprendizado desses conceitos.
