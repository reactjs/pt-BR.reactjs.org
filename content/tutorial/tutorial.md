---
id: tutorial
title: "Tutorial: Introdução ao React"
layout: tutorial
sectionid: tutorial
permalink: tutorial/tutorial.html
redirect_from:
  - "docs/tutorial.html"
  - "docs/why-react.html"
  - "docs/tutorial-ja-JP.html"
  - "docs/tutorial-ko-KR.html"
  - "docs/tutorial-zh-CN.html"
---

Este tutorial não assume nenhum conhecimento existente do React.

## Antes de começarmos o tutorial {#before-we-start-the-tutorial}

Vamos construir um pequeno jogo durante este tutorial. **Você pode ficar tentado a ignorá-lo porque você não vai construir jogos - mas dê uma chance.** As técnicas que você aprenderá no tutorial são fundamentais para criar qualquer aplicativo React e o domínio delas lhe dará um profundo entendimento de React.

> Dica
>
> Este tutorial foi criado para pessoas que preferem **aprender fazendo**. Se você preferir aprender conceitos do zero, confira nosso [step-by-step guide](/docs/hello-world.html). Você pode achar este tutorial e o guia complementares um ao outro.

O tutorial está dividido em várias seções:

* [Configuração do Tutorial](#setup-for-the-tutorial) lhe dará **um ponto de partida** para seguir o tutorial.
* [Visão geral](#overview) te ensinará **os fundamentos** do React: componentes, propriedades (_props_) e estado (_state_).
* [Completando o Jogo](#completing-the-game) te ensinará **as técnicas mais comuns** para desenvolver com React.
* [Adicionando Time Travel (viagem no tempo)](#adding-time-travel) lhe dará **uma visão mais profunda** dos pontos fortes exclusivos do React.

Você não precisa completar todas as seções de uma vez para entender tudo que o tutorial tem a oferecer. Tente chegar o mais longe possível, mesmo que seja uma ou duas seções.

### O que estamos construindo? {#what-are-we-building}

Neste tutorial, mostraremos como criar um jogo interativo de jogo-da-velha com React.

Você pode ver o que vamos construir aqui: **[Resultado Final](https://codepen.io/gaearon/pen/gWWZgR?editors=0010)**. Se o código não fizer sentido para você ou se você não estiver familiarizado com a sintaxe do código, não se preocupe! O objetivo deste tutorial é ajudar você a entender o React e sua sintaxe.

Recomendamos que você confira o jogo tic-tac-toe (jogo da velha) antes de continuar com o tutorial. Uma das características que você notará é que existe uma lista numerada à direita do tabuleiro do jogo. Esta lista fornece um histórico de todas as jogadas que ocorreram no jogo e é atualizada à medida que o jogo avança.

Você pode fechar o jogo da velha assim que estiver familiarizado com ele. Começaremos a partir de um modelo mais simples neste tutorial. Nosso próximo passo é prepará-lo para que você possa começar a desenvolver o jogo.

### Pré-requisitos {#prerequisites}

Vamos presumir que você já tenha alguma familiaridade com HTML e JavaScript. Porém, você deve ser capaz de acompanhá-lo mesmo que esteja vindo de uma linguagem de programação diferente. Também vamos presumir que você também já esteja familiarizado com conceitos de programação. Tais como: funções, objetos, matrizes em menor escala e classes.

Se você precisa revisar JavaScript, recomendamos ler [este guia](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/A_re-introduction_to_JavaScript). Observe que também estamos usando alguns recursos do ES6 - uma versão recente do JavaScript. Neste tutorial, estamos usando [arrow function](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Functions/Arrow_functions), [classes](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Classes), [`let`](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Statements/let), e declarações [`const`](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Statements/const). Você pode usar o [Babel REPL](babel://es5-syntax-example) para verificar o código ES6 compilado.

## Configuração para o tutorial {#setup-for-the-tutorial}

Há duas maneiras de concluir este tutorial: você pode escrever o código em seu navegador ou configurar um ambiente de desenvolvimento local em seu computador.

### Opção de Configuração 1: Desenvolvendo o Código no Navegador {#setup-option-1-write-code-in-the-browser}

Esta é a maneira mais rápida de começar!

Primeiro, abra este **[Código Inicial](https://codepen.io/gaearon/pen/oWWQNa?editors=0010)** em uma nova guia. A nova aba deve exibir um tabuleiro de tic-tac-toe (jogo da velha) vazio e o código React. Nós estaremos editando o código React neste tutorial.

Agora você pode ignorar a segunda opção de configuração e ir para a seção [Visão geral](#overview) para obter uma visão geral do React.

### Opção de instalação 2: ambiente de desenvolvimento local {#setup-option-2-local-development-environment}

Isto é completamente opcional e não é necessário para este tutorial!

<br>

<details>

<summary><b>Opcional: instruções para serem seguidas na sua máquina usando seu editor de texto preferido</b></summary>

Essa opção de configuração requer mais trabalho mas permite que você complete o tutorial utilizando um editor de sua preferência. Aqui estão os passos a serem seguidos:

1. Certifique-se de ter uma versão recente do [Node.js](https://nodejs.org/pt-br/) instalada.
2. Siga as [instruções de instalação do create-react-app](/docs/create-a-new-react-app.html#create-react-app) para criar um novo projeto.

```bash
npx create-react-app my-app
```

3. Apague todos os arquivos na pasta `src/` do novo projeto

> Nota:
>
> **não exclua a pasta `src` inteira. Apenas os arquivos originais dentro dela.** Substituiremos os arquivos de origem padrão por exemplos deste projeto na próxima etapa.

```bash
cd my-app
cd src

# Se você estiver usando um Mac ou Linux:
rm -f *

# Ou, se você estiver no Windows:
del *

# E então, volte para a pasta do projeto
cd ..
```

4. Adicione um arquivo chamado `index.css` na pasta` src/`com [este código CSS](https://codepen.io/gaearon/pen/oWWQNa?editors=0100).

5. Adicione um arquivo chamado `index.js` na pasta` src/`com [este código JS](https://codepen.io/gaearon/pen/oWWQNa?editors=0010).

6. Adicione estas três linhas ao topo do `index.js` na pasta `src/`:

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
```

Agora, se você executar o `npm start` na pasta do projeto e abrir `http://localhost:3000` no navegador, você verá um campo vazio de jogo da velha.

Recomendamos seguir [estas instruções](https://babeljs.io/docs/editors/) para configurar o realce de sintaxe para seu editor.

</details>

### Me Ajudem. Estou com Dificuldades! {#help-im-stuck}

Se você não conseguir proseguir no tutorial por algum motivo, confira os [recursos de suporte da comunidade](/community/support.html). Em particular, o [Reactiflux Chat](https://discord.gg/reactiflux) é uma ótima maneira de obter ajuda rapidamente. Se você não receber uma resposta ou se permanecer preso, registre um problema e nós ajudaremos você.

## Visão geral {#overview}

Agora que tudo está configurado, vamos obter uma visão geral do React!

### O que é React? {#what-is-react}

O React é uma biblioteca JavaScript declarativa, eficiente e flexível para criar interfaces com o usuário. Ele permite compor UIs complexas a partir de pequenos e isolados códigos chamados "componentes".

React possui alguns tipos diferentes de componentes, mas começaremos com subclasses `React.Component`:

```javascript
class ShoppingList extends React.Component {
  render() {
    return (
      <div className="shopping-list">
        <h1>Lista de compras para {this.props.name}</h1>
        <ul>
          <li>Instagram</li>
          <li>WhatsApp</li>
          <li>Oculus</li>
        </ul>
      </div>
    );
  }
}

// Exemplo de uso: <ShoppingList name="Mark" />
```

Nós falaremos sobre formatos engraçados de tags que possuem formato de XML em breve. Usamos componentes para dizer ao React o que queremos ver na tela. Quando nossos dados forem alterados, o React atualizará e renderizará novamente com eficiência nossos componentes.

Aqui, o ShoppingList é um **componente React de classe** ou **component React do tipo classe**. Um componente recebe parâmetros, chamados `props` (abreviação de propriedades), e retorna uma hierarquia de elementos para exibir através do método `render`.

O método `render` retorna uma *descrição* do que você deseja ver na tela. React recebe a descrição e exibe o resultado. Em particular, `render` retorna um **elemento React**, que é uma descrição simplificada do que renderizar. A maioria dos desenvolvedores do React usa uma sintaxe especial chamada "JSX", que facilita a escrita desses elementos. A sintaxe `<div />` é transformada em tempo de compilação para `React.createElement ('div')`. O exemplo acima é equivalente a:

```javascript
return React.createElement('div', {className: 'shopping-list'},
  React.createElement('h1', /* ... filhos de h1 ... */),
  React.createElement('ul', /* ... filhos de ul ... */)
);
```

[Veja a versão completa.](babel://tutorial-expanded-version)

Se você está curioso, o `createElement()` é descrito em mais detalhes na [referência da API](/docs/react-api.html#createelement), mas não iremos usá-lo neste tutorial. Em vez disso, continuaremos usando o JSX.

O JSX vem com todo o poder do JavaScript. Você pode colocar *quaisquer* expressões JavaScript dentro de chaves no JSX. Cada elemento React é um objeto JavaScript que você pode armazenar em uma variável ou passar em seu código.

O componente `ShoppingList` acima apenas renderiza componentes internos do DOM como `<div />` e ` <li />`. Mas você também pode compor e renderizar componentes React personalizados. Por exemplo, agora podemos nos referir a toda a lista de compras escrevendo `<ShoppingList />`. Cada componente React é encapsulado e pode operar de forma independente; Isso permite que você construa interfaces complexas a partir de componentes simples.

### Inspecionando o Código Inicial {#inspecting-the-starter-code}

Se você for trabalhar no tutorial **em seu navegador,** abra esse código em uma nova guia: **[Código Inicial](https://codepen.io/gaearon/pen/oWWQNa?editors=0010 )** Se você vai trabalhar no tutorial **localmente,** abra `src/index.js` em sua pasta de projeto (você já criou este arquivo durante a [configuração](#setup-option-2-local-development-environment)).

Este Código Inicial é a base do que estamos construindo. Fornecemos o estilo CSS para que você só precise se concentrar no aprendizado do React e na programação do jogo da velha.

Ao inspecionar o código, você notará que temos três componentes React:

* Quadrado(Square)
* Tabuleiro(Board)
* Jogo(Game)

O componente Square renderiza um único `<button>` e o Board renderiza 9 squares. O componente Game renderiza um Board com valores que modificaremos mais tarde. Atualmente não há componentes interativos.

### Passando dados através de props {#passing-data-through-props}

Para aquecer, vamos tentar passar alguns dados do nosso componente Board para o nosso componente Square.

É altamente recomendável digitar o código manualmente, enquanto você está trabalhando no tutorial e não usando copiar/colar. Isso ajudará você a desenvolver a memória muscular e um melhor entendimento.

No método `renderSquare` do Board, altere o código para passar um prop chamado `value` para o Square:

```js{3}
class Board extends React.Component {
  renderSquare(i) {
    return <Square value={i} />;
  }
}
```

Altere o método `render` do Square para mostrar esse valor substituindo `{/ * TODO * /}` por `{this.props.value}`:

```js{5}
class Square extends React.Component {
  render() {
    return (
      <button className="square">
        {this.props.value}
      </button>
    );
  }
}
```

Antes:

![React Devtools](../images/tutorial/tictac-empty.png)

Depois: Você deve ver um número em cada quadrado na saída renderizada.

![React Devtools](../images/tutorial/tictac-numbers.png)

**[Ver o código completo até este momento](https://codepen.io/gaearon/pen/aWWQOG?editors=0010)**

Parabéns! Você acabou de passar um "prop" de um componente pai Board para um componente filho Square. Passar props é a forma como os dados fluem em aplicações React, de pais para filhos.

### Fazendo um componente interativo {#making-an-interactive-component}

Vamos preencher o componente Square com um "X" quando clicamos nele.
Primeiro, altere a tag `button` que é retornada na função `render()` do componente Square para isto:

```javascript{4}
class Square extends React.Component {
  render() {
    return (
      <button className="square" onClick={function() { alert('click'); }}>
        {this.props.value}
      </button>
    );
  }
}
```

Se você clicar em um quadrado agora, deverá ver um alerta no seu navegador.

>Nota
>
>Para salvar a digitação e evitar o [comportamento confuso de `this`](https://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/),vamos usar a sintaxe [arrow function](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Functions/Arrow_functions) para manipuladores de eventos:
>
>```javascript{4}
>class Square extends React.Component {
>  render() {
>    return (
>      <button className="square" onClick={() => alert('click')}>
>        {this.props.value}
>      </button>
>    );
>  }
>}
>```
>
>Note que com `onClick = {() => alert ('click')}`, estamos passando *uma função* como prop `onClick`. O React só chamará essa função depois de um clique. Esquecendo `() =>` e escrevendo somente `onClick = {alert ('click')}` é um erro comum, e dispararia o alerta toda vez que o componente fosse renderizado novamente.

Como próximo passo, queremos que o componente Square "lembre" que foi clicado e preencha com um "X". Para "lembrar" as coisas, os componentes usam o **estado (_state_)**.

Os componentes React podem ter estado (_state_) configurando `this.state` em seus construtores. `this.state` deve ser considerado como privado para o componente React que o definiu. Vamos armazenar o valor atual do Square em `this.state` e alterá-lo quando o Square for clicado.

Primeiro, adicionaremos um construtor à classe para inicializar o estado:

```javascript{2-7}
class Square extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: null,
    };
  }

  render() {
    return (
      <button className="square" onClick={() => alert('click')}>
        {this.props.value}
      </button>
    );
  }
}
```

>Nota
>
>Em [classes JavaScript](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Classes), você sempre precisa chamar `super` ao definir o construtor de uma subclasse. Todas os componentes de classe React que possuem um método `constructor` devem iniciá-lo com uma chamada `super (props)`.

Agora vamos mudar o método `render` do componente Square para exibir o valor do estado (_state_) atual quando clicado:

* Substitua `this.props.value` por` this.state.value` dentro da tag `<button>`.
* Substitua o `onClick={...}` event handler por `onClick={() => this.setState({value: 'X'})}`.
* Coloque `className` e` onClick` props em linhas separadas para melhor legibilidade.

Após estas mudanças, a tag `<button>` que é retornada pelo método `render` do Square deve se parecer com isto:

```javascript{12-13,15}
class Square extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: null,
    };
  }

  render() {
    return (
      <button
        className="square"
        onClick={() => this.setState({value: 'X'})}
      >
        {this.state.value}
      </button>
    );
  }
}
```

Ao chamar `this.setState` a partir de um manipulador `onClick` no método `render` do componente Square, nós dizemos ao React para renderizar novamente aquele Square sempre que seu` <button>` for clicado. Após a atualização, o `this.state.value` do Square será `'X'`, então vamos ver o `X` no tabuleiro do jogo. Se você clicar em qualquer quadrado, um `X` deve aparecer.

Quando você chama `setState` em um componente, o React atualiza automaticamente os componentes filhos dentro dele também.

**[Ver o código completo até este momento](https://codepen.io/gaearon/pen/VbbVLg?editors=0010)**

### Developer Tools {#developer-tools}

A extensão React Devtools para [Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en) e [Firefox](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/) permite inspecionar uma árvore de componentes React com as ferramentas de desenvolvedor do seu navegador.

<img src="../images/tutorial/devtools.png" alt="React Devtools" style="max-width: 100%">

O React DevTools permite que você verifique as props e o estado (_state_) de seus componentes React.

Depois de instalar o React DevTools, você pode clicar com o botão direito do mouse em qualquer elemento da página, clicar em "Inspecionar" para abrir as ferramentas de desenvolvedor, e as guias React ("⚛️ Components" e "⚛️ Profiler") aparecerá como a última guia à direita.  Use "⚛️ Components" para inspecionar a árvore de componentes.

**No entanto, observe que há algumas etapas extras para a extensão funcionar com o CodePen:**

1. Faça o login ou registre-se e confirme seu e-mail (necessário para evitar spam).
2. Clique no botão "Fork".
3. Clique em "Change View" e escolha "Debug mode".
4. Na nova aba que se abre, o devtools deve agora ter uma aba React.

## Completando o jogo {#completing-the-game}

Agora nós temos os blocos básicos para contrução do nosso jogo da velha. Para completar o jogo, nós precisamos preencher os "X"s e os "O"s no tabuleiro e de alguma maneira necessitamos definir o vencedor.

### Movendo o state para cima {#lifting-state-up}

Atualmente, cada componente Quadrado (Square) mantém o estado do jogo. Para verificar o vencedor, nós vamos manter o valor de cada um dos 9 quadrados em uma posição.

Podemos pensar que o Tabuleiro (Board) poderia apenas perguntar para cada Quadrado pelo seu estado. Apesar desse modelo ser possível no React, nós o desencorajamos, pois, o código se torna difícil de ser compreendido, suscetível à erros e difícil de refatorar. Ao invés disso, a melhor opção é guardar o estado do jogo no componente pai (Tabuleiro) ao invés de cada Quadrado. O componente do tabuleiro pode dizer para cada Quadrado o que pode ser exibido via prop, [assim como fizemos quando passamos o número de cada Quadrado](#passing-data-through-props).

**Para coletar dados de múltiplos filhos (children), ou para fazer dois filhos se comunicarem entre si, você precisa declarar um estado compartilhado em seu componente pai. O componente pai pode passar o estado de volta para os filhos através do uso de propriedades (props); isso mantém os componentes filhos em sincronia com os seus irmãos e também com o pai.**

Criar estado em um componente Pai é bem comum quando componentes React são refatorados -- Vamos aproveitar essa oportunidade para tentar o conceito, na prática.

Vamos adicionar um construtor no Tabuleiro e definir que seu estado inicial irá ter um array com 9 posições preenchidas por nulo (null). Esses 9 nulls corresponderão aos 9 quadrados: 

```javascript{2-7}
class Board extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      squares: Array(9).fill(null),
    };
  }

  renderSquare(i) {
    return <Square value={i} />;
  }

  render() {
    const status = 'Next player: X';

    return (
      <div>
        <div className="status">{status}</div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    );
  }
}
```

Quando preenchermos o tabuleiro mais tarde, ele ficará parecido com isto:

```javascript
[
  'O', null, 'X',
  'X', 'X', 'O',
  'O', null, null,
]
```

O método `renderSquare` do Tabuleiro atualmente está definido como:

```javascript
  renderSquare(i) {
    return <Square value={i} />;
  }
```

No começo, nós [passamos o seu valor (value) como prop](#passing-data-through-props) para o Tabuleiro mostrar números de 0 a 8 em cada Quadrado. Em outro passo anterior, nós trocamos os números pela letra "X" [determinado no próprio estado do Quadrado](#making-an-interactive-componente). Isso porque atualmente o Quadrado ignora o valor (`value`) recebido do Tabuleiro.

Iremos agora utilizar novamente o mesmo mecanismo de propriedades. Vamos modificar o Tabuleiro para instruir cada Quadrado individualmente qual é o valor correto (`'X'`, `'O'` ou `null`). Nós já temos definidos o array de `quadrados` no construtor do Tabuleiro e iremos modificar o método `renderSquare` para definir o valor a partir do estado:

```javascript{2}
  renderSquare(i) {
    return <Square value={this.state.squares[i]} />;
  }
```

**[Veja o código até este momento](https://codepen.io/gaearon/pen/gWWQPY?editors=0010)**

Cada Square vai receber a proriedade `value` que vai ser `'X'`, `'O'`, ou `null` para quadrados vazios.

Agora, precisamos mudar o que acontece quando um Quadrado é clicado. O componente Tabuleiro agora mantém quais quadrados são preenchidos. Precisamos criar uma maneira para cada Quadrado atualizar o state do Tabuleiro. O state é considerado privado ao componente em que é definido, ou seja, nós não podemos atualizar o state do Tabuleiro diretamente do Quadrado.

Para manter a privacidade do state do Tabuleiro, nós vamos passar a função responsável do Tabuleiro para o Quadrado. Essa função irá ser chamada assim que o Quadrado for clicado. Nós então mudaremos o método `renderSquare` no Tabuleiro para:

```javascript{5}
  renderSquare(i) {
    return (
      <Square
        value={this.state.squares[i]}
        onClick={() => this.handleClick(i)}
      />
    );
  }
```

>Nota
>
>Nós quebramos o retorno do elemento em várias linhas para melhorar a legibilidade e adicionamos parentesis para que o JavaScript não insira ponto e virgula após o `return` e quebre o código

Agora nós iremos passar duas props do Tabuleiro para o Quadrado: `value` e `onClick`. A propriedade `onClick` é uma função que será chamada quando o Quadrado for clicado. Nós manteremos as seguintes mudanças no componente Quadrado:

* Substituir `this.state.value` por `this.props.value` no método `render`;
* Substituir `this.setState()` por `this.props.onClick()` no método `render`;
* Deletar o `constructor` do Quadrado, já que não manteremos mais o state do jogo nele;

Após essas mudanças, o componente Quadrado se parecerá com isto:

```javascript{1,2,6,8}
class Square extends React.Component {
  render() {
    return (
      <button
        className="square"
        onClick={() => this.props.onClick()}
      >
        {this.props.value}
      </button>
    );
  }
}
```

Quando um Quadrado for clicado, a função `onClick` provida pelo Tabuleiro será chamada. Aqui está uma revisão de como isso acontece:

1. A propriedade `onClick` do DOM embutida no componente `<button>` diz ao React para criar um evento de escuta (event listener).
2. Quando o botão é clicado, o React irá chamar a função o manipulador de eventos `onClick` definido no método `render()` do Quadrado.
3. Esse manipulador de eventos chamará a função recebida através da propriedade `onClick` que foi criada no Tabuleiro (`this.props.onClick()`).
4. Como o Tabuleiro passou `onClick={() => this.handleClick(i)}` para o Quadrado, a função `this.handleClick(i)` será chamada quando o Quadrado for clicado.
5. Como nós não definimos a função `handleClick()` ainda, nosso código quebrará.

>Nota
>
>O atributo `onClick` dos elementos `<button>` no DOM possuem um significado especial para o React, pois ele é um componente nativo. Para componentes customizados como o Square, o nome é por sua conta. Nós poderíamos renomear a propriedade `onClick` do componente Square para `handleClick`. Em React, no entanto, a convenção é usar nomes `on[Event]` para propriedades que representam eventos e `handle[Event]` para metodos que manipulam os eventos.

Quando tentamos clicar em um Square, um erro ocorrerá, pois, não definimos a função `handleClick` ainda. O adicionaremos agora na classe Board:

```javascript{9-13}
class Board extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      squares: Array(9).fill(null),
    };
  }

  handleClick(i) {
    const squares = this.state.squares.slice();
    squares[i] = 'X';
    this.setState({squares: squares});
  }

  renderSquare(i) {
    return (
      <Square
        value={this.state.squares[i]}
        onClick={() => this.handleClick(i)}
      />
    );
  }

  render() {
    const status = 'Next player: X';

    return (
      <div>
        <div className="status">{status}</div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    );
  }
}
```

**[Veja o código até este momento](https://codepen.io/gaearon/pen/ybbQJX?editors=0010)**

Após essas mudanças, seremos capazes novamente de clicar nos Squares para preenche-los. Entretanto, agora o state é guardado no componente Board ao invés de em cada Square. Quando o state do Board for alterado, os componentes Square serão re-renderizados automaticamente. Manter o state de todos os quadrados no componente Board nos permitirá determinar o vencedor no futuro.

Como o componente Square não mantém mais state, os componentes Square receberão os valores do Board e o informarão quando forem clicados. Em "termos React", os Squares são agora **componentes controlados** (**controlled components**). O Board terá controle total sobre eles.

Note como na função `handleClick`, nós chamamos `.slice()` para criar uma cópia do array de quadrados para o modificar ao invés de faze-lo no array existente. Explicaremos o motivo quando criarmos uma copia do array de `quadrados` na próxima sessão.

### Por que Imutabilidade é Importante {#why-immutability-is-important}

<<<<<<< HEAD
No código do exemplo anterior nós sugerimos que você usasse o método `.slice()` para criar uma cópia do array de `quadrados` ao invés de modificar o existente. Iremos agora discutir imutabilidade e porque ela é importante de se aprender.
=======
In the previous code example, we suggested that you use the `.slice()` method to create a copy of the `squares` array to copy instead of modifying the existing array. We'll now discuss immutability and why immutability is important to learn.
>>>>>>> 446345cfba91b62546f46f88fad893937a826cdc

Geralmente existem duas maneiras de se alterar dados. A primeira é *mutar* o dado alterando diretamente seu valor. A segunda maneira é substituir o dado antigo por uma nova cópia com as alterações desejadas.

#### Mudando dados com mutação {#data-change-with-mutation}
```javascript
var player = {score: 1, name: 'Jeff'};
player.score = 2;
// Agora o player é {score: 2, name: 'Jeff'}
```

#### Mudando dados sem mutação {#data-change-without-mutation}
```javascript
var player = {score: 1, name: 'Jeff'};

var newPlayer = Object.assign({}, player, {score: 2});
// Agora o player não sofreu alteração, mas o newPlayer é {score: 2, name: 'Jeff'}

// Ou então se você estiver usando a sintaxe "object spread", você pode escrever:
// var newPlayer = {...player, score: 2};
```

O resultado final será o mesmo, mas por não mutar (ou alterar os dados subjacentes) diretamente, nós ganhamos vários benefícios descritos abaixo

#### Complexidade das features se tornam mais simples {#complex-features-become-simple}

Imutabilidades faz a complexidade das features se tornarem bem mais simples de serem implementadas. Mais tarde neste tutorial, implementaremos uma feature de "máquina do tempo" que nos permitirá revisar o histórico do jogo da velha e "voltar" as jogadas anteriores.
Essa funcionalidade não está ligada somente ao jogo -- uma habilidade de desfazer e refazer certas ações é um requisito comum em aplicações. Evitar mutação nos permite manter o histórico das versões anteriores do jogo intacta e reutiliza-las mais tarde.

#### Detectar Mudanças {#detecting-changes}

Detectar mudanças e objetos mutados é difícil, pois, eles são modificados diretamente. Essa detecção requer um objeto mutado para ser comparado com as cópias das suas próprias versões anteriores e a árvore inteira do object para ser cruzada.

Detectar mudanças em objetos imutáveis é consideravelmente fácil. Se ele for imutável que está sendo referenciado for diferente do anterior, concluímos que o objeto foi alterado.

#### Determinar Quando Re-renderizar no React {#determining-when-to-re-render-in-react}

O principal benefício da imutabilidade é que ela ajuda a construir _componentes puros_ em React. Dados imutáveis podem facilmente determinar se foram feitas mudanças, que ajudarão a decidir quando um componente precisa ser re-renderizado.

Você pode aprender mais sobre `shouldComponentUpdate` e como construir *componentes puros* lendo o artigo [Otimizando Performance](/docs/optimizing-performance.html#examples).

### Componentes de Função {#function-components}

Nós vamos agora mudar o Square para ser um **componente de função**.

Em React, **componentes de função** são os mais simples de serem escritos, contém apenas um método `render` e não possuem seu próprio state. Ao invés de definir uma classe que extende de `React.Component`, nós podemos escrever uma função que recebe `props` como entrada e retorna o que deverá ser renderizado. Esse tipo de componente é menos tedioso de escrever do que classes e muitos componentes podem ser expressados desta maneira.

Troque a classe Square por esta função:

```javascript
function Square(props) {
  return (
    <button className="square" onClick={props.onClick}>
      {props.value}
    </button>
  );
}
```

Nos modificamos `this.props` para `props` nas duas vezes que ela aparece.

**[Veja o código completo até aqui](https://codepen.io/gaearon/pen/QvvJOv?editors=0010)**

>Nota
>
>Quando modificamos Square para ser um componente funcional, também modificamos `onClick={() => this.props.onClick()}` para uma versão mais curta: `onClick={props.onClick}` (note a ausência dos parentêses em *ambos* os lados).

### Trocando Turnos {#taking-turns}

Agora precisamos consertar um defeito óbvio em nosso Jogo da Velha: os "O"s não podem ser marcados no tabuleiro.

Vamos definir a primeira jogadas para ser "X" por padrão. Podemos definir esse padrão modificando o state inicial no construtor do nosso tabuleiro (Board)

```javascript{6}
class Board extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      squares: Array(9).fill(null),
      xIsNext: true,
    };
  }
```

Sempre que um jogador fizer uma jogada, `xIsNext` (um boolean) será trocado para determinar qual jogador será o próximo e o state do jogo será  salvo. Nós atualizaremos a função `handleClick` do Board para trocar o valor de `xIsNext`:

```javascript{3,6}
  handleClick(i) {
    const squares = this.state.squares.slice();
    squares[i] = this.state.xIsNext ? 'X' : 'O';
    this.setState({
      squares: squares,
      xIsNext: !this.state.xIsNext,
    });
  }
```

Com esse mudança,"X"s e "O"s podem trocar os turnos. Tente!

Também vamos modificar o texto de "status" na função `render` do Board para que ela passe a exibir quem jogará o próximo turno.

```javascript{2}
  render() {
    const status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');

    return (
      // o restante não tem alterações
```

Depois de fazer essas mudanças, você deverá ter esse componente do Board:

```javascript{6,11-16,29}
class Board extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      squares: Array(9).fill(null),
      xIsNext: true,
    };
  }

  handleClick(i) {
    const squares = this.state.squares.slice();
    squares[i] = this.state.xIsNext ? 'X' : 'O';
    this.setState({
      squares: squares,
      xIsNext: !this.state.xIsNext,
    });
  }

  renderSquare(i) {
    return (
      <Square
        value={this.state.squares[i]}
        onClick={() => this.handleClick(i)}
      />
    );
  }

  render() {
    const status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');

    return (
      <div>
        <div className="status">{status}</div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    );
  }
}
```

**[Veja o código completo até aqui](https://codepen.io/gaearon/pen/KmmrBy?editors=0010)**

### Declarando um Vencedor {#declaring-a-winner}

Agora que mostramos quem jogará o próximo turno, também deveríamos mostrar quando o jogo foi vencido e que não há mais turnos a serem jogados. Copie essa função auxiliar e cole-a no final do arquivo:

```javascript
function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

Dado um array de 9 quadrados, esta função irá verificar se há um vencedor e retornará `'X'`, `'O'` ou `null` conforme apropriado

Chamaremos `calculateWinner(squares)` na função `render` do Board para checar se um jogador venceu. Caso tenha vencido, podemos mostrar um texto como "Winner: X" ou "Winner: O". Vamos substituir a declaração de `status` na função `render` com esse código:

```javascript{2-8}
  render() {
    const winner = calculateWinner(this.state.squares);
    let status;
    if (winner) {
      status = 'Winner: ' + winner;
    } else {
      status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
    }

    return (
      // o restante não tem alterações
```

Agora podemos modificar a função `handleClick` do Board para retornar antes, ignorando o click, caso alguém tenha vencido o jogo ou se o quadrado (square) já esteja ocupado:

```javascript{3-5}
  handleClick(i) {
    const squares = this.state.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i] = this.state.xIsNext ? 'X' : 'O';
    this.setState({
      squares: squares,
      xIsNext: !this.state.xIsNext,
    });
  }
```

**[Veja o código completo até aqui](https://codepen.io/gaearon/pen/LyyXgK?editors=0010)**

Parabéns! Você agora tem um Jogo da Velha funcionando! E também acaba de aprender o básico de React. Então, *você* é provavelmente o verdadeiro vencedor aqui.

## Adicionando a Viagem no Tempo (Time Travel){#adding-time-travel}

Como um último exercício, vamos tornar possível fazer uma "volta no tempo" até as jogadas anteriores que aconteceram no jogo.

### Armazenando um Histórico de Jogadas {#storing-a-history-of-moves}

Se nós tivéssemos modificado o array `squares`, a implementação da volta no tempo seria muito difícil.

No entanto, nós utilizamos `slice()` para criar uma nova cópia do array `squares` após cada jogada e [tratamos ele como imutável](#why-immutability-is-important). Isso nos permitirá o armazenamento de cada versão anterior do array `squares` e que possamos navegar entre os turnos que já tenham acontecido. 

Vamos armazenar os arrays `squares` anteriores em um outro array chamado `history`. O array `history` representa todos os estados do tabuleiro, da primeira à última jogada, e tem uma forma parecida com essa:

```javascript
history = [
  // Antes da primeira jogada
  {
    squares: [
      null, null, null,
      null, null, null,
      null, null, null,
    ]
  },
  // Depois da primeira jogada
  {
    squares: [
      null, null, null,
      null, 'X', null,
      null, null, null,
    ]
  },
  // Depois da segunda jogada
  {
    squares: [
      null, null, null,
      null, 'X', null,
      null, null, 'O',
    ]
  },
  // ...
]
```

Agora precisamos decidir a qual componente pertencerá o state do `history`.

### Trazendo o State pra Cima, Novamente {#lifting-state-up-again}

Queremos que o componente Game, o de mais alto nível, mostre uma lista com as jogadas anteriores. Para poder fazer isso, ele precisará acessar o `history`, então, temos que trazer o state `history` para cima, colocando-o no componente de mais alto nível, o componente Game.

Colocar o state `history` no componente Game, nos permite remover o state `squares` de seu componente filho, Board. Assim como ["trouxemos para cima"](#trazendo-state-para-cima) o state do componente Square para o componente Board, agora estamos trazendo o state do componente Board para o componente de mais alto nível, Game. Isso dá ao componente Game total controle sobre os dados do Board e permite que instrua o Board a renderizar turnos anteriores a partir do `history`.

Primeiro, iremos configurar o state inicial para o componente Game em seu construtor:

```javascript{2-10}
class Game extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      history: [{
        squares: Array(9).fill(null),
      }],
      xIsNext: true,
    };
  }

  render() {
    return (
      <div className="game">
        <div className="game-board">
          <Board />
        </div>
        <div className="game-info">
          <div>{/* status */}</div>
          <ol>{/* TODO */}</ol>
        </div>
      </div>
    );
  }
}
```

Em seguida, faremos com que o componente Board receba as props `squares` e `onClick` do componente Game. Agora, uma vez que temos apenas um manipulador de clique no Board para vários Squares, vamos precisar passar a localização de cada Square para o manipulador `onClick` para indicar qual Square foi clicado. Aqui estão os passos necessários para transformar o componente Board:

* Deletar o `contructor` do Board.
* Substituir `this.state.squares[i]` por `this.props.squares[i]` na função `renderSquare` do Board.
* Substituir `this.handleClick(i)` por `this.props.onClick(i)` na função `renderSquare` do Board.

O componente Board agora ficou assim:

```javascript{17,18}
class Board extends React.Component {
  handleClick(i) {
    const squares = this.state.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i] = this.state.xIsNext ? 'X' : 'O';
    this.setState({
      squares: squares,
      xIsNext: !this.state.xIsNext,
    });
  }

  renderSquare(i) {
    return (
      <Square
        value={this.props.squares[i]}
        onClick={() => this.props.onClick(i)}
      />
    );
  }

  render() {
    const winner = calculateWinner(this.state.squares);
    let status;
    if (winner) {
      status = 'Winner: ' + winner;
    } else {
      status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
    }

    return (
      <div>
        <div className="status">{status}</div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    );
  }
}
```

Vamos atualizar a função `render` do componente Game para utilizar a entrada mais recente do histórico (history) para determinar e exibir o status do jogo.

```javascript{2-11,16-19,22}
  render() {
    const history = this.state.history;
    const current = history[history.length - 1];
    const winner = calculateWinner(current.squares);

    let status;
    if (winner) {
      status = 'Winner: ' + winner;
    } else {
      status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
    }

    return (
      <div className="game">
        <div className="game-board">
          <Board
            squares={current.squares}
            onClick={(i) => this.handleClick(i)}
          />
        </div>
        <div className="game-info">
          <div>{status}</div>
          <ol>{/* TODO */}</ol>
        </div>
      </div>
    );
  }
```

Uma vez que o componente Game agora está renderizando o status do jogo, nós podemos remover o código correspondente do método `render` do componente Board. Depois de refatorar, a função `render` do Board fica assim:

```js{1-4}
  render() {
    return (
      <div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    );
  }
```

Finalmente, precisamos mover o método `handleClick` do componente Board para o componente Game. Nós também precisamos modificar `handleClick` pois o state do componente Game está estruturado de maneira diferente. No componente Game, dentro do método `handleClick`, nós concatenamos novas entradas do histórico de jogadas em `history`.

```javascript{2-4,10-12}
  handleClick(i) {
    const history = this.state.history;
    const current = history[history.length - 1];
    const squares = current.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i] = this.state.xIsNext ? 'X' : 'O';
    this.setState({
      history: history.concat([{
        squares: squares,
      }]),
      xIsNext: !this.state.xIsNext,
    });
  }
```

>Nota
>
>Ao contrário do método de arrays `push()`, que você talvez possa estar mais familiarizado, o método `concat()` não modifica o array original, por isso preferimos utilizá-lo. 

Nesse ponto, o componente Board necessita apenas dos métodos `renderSquare` e `render`. O state do jogo e o método `handleClick` devem estar no componente Game.

**[Veja o código completo até aqui](https://codepen.io/gaearon/pen/EmmOqJ?editors=0010)**

### Mostrando as Jogadas Anteriores{#showing-the-past-moves}

Uma vez que estamos gravando o histórico do Jogo da Velha, agora podemos mostrá-lo para o jogador como uma lista de jogadas anteriores.

Aprendemos anterioremente que os elementos React são objetos JavaScript de primeira classe; podemos passá-los livremente por nossas aplicações. Para renderizar múltiplos itens em React, podemos utilizar um array de elementos React.

Em JavaScript, arrays possuem um [método `map()`](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Array/map) que é normalmente utilizado para mapear uma fonte de dados para outra fonte de dados, por exemplo:

```js
const numbers = [1, 2, 3];
const doubled = numbers.map(x => x * 2); // [2, 4, 6]
```

Utilizando o método `map`, nós podemos mapear nosso histórico de jogadas para elementos React, representando botões na tela, e mostrar uma lista de botões que "pulam" para os jogadas anteriores.

Vamos fazer um `map` sobre o `history` no método `render` do componente Game:

```javascript{6-15,34}
  render() {
    const history = this.state.history;
    const current = history[history.length - 1];
    const winner = calculateWinner(current.squares);

    const moves = history.map((step, move) => {
      const desc = move ?
        'Go to move #' + move :
        'Go to game start';
      return (
        <li>
          <button onClick={() => this.jumpTo(move)}>{desc}</button>
        </li>
      );
    });

    let status;
    if (winner) {
      status = 'Winner: ' + winner;
    } else {
      status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
    }

    return (
      <div className="game">
        <div className="game-board">
          <Board
            squares={current.squares}
            onClick={(i) => this.handleClick(i)}
          />
        </div>
        <div className="game-info">
          <div>{status}</div>
          <ol>{moves}</ol>
        </div>
      </div>
    );
  }
```

**[Veja o código completo nessa etapa](https://codepen.io/gaearon/pen/EmmGEa?editors=0010)**

Para cada jogada no histórico do Jogo da Velha, nós criamos um item de lista `<li>` que contém um botão `<button>`. O botão tem um manipulador `onClick` que chama um método chamado `this.jumpTo()`. Nós ainda não implementamos o método `jumpTo()`. Por agora, nós devemos ver uma lista das jogadas que já ocorreram no jogo e um aviso no console do developer tools que diz: 

>  Warning:
>  Each child in an array or iterator should have a unique "key" prop. Check the render method of "Game".

em português:

>  Aviso:
>  Cada filho de um array ou iterator deve ter uma prop "key" única. Confira o método render de "Game"

Vamos discutir sobre o que o aviso acima significa:

### Definindo uma Key (Chave) {#picking-a-key}

Quando renderizamos uma lista, o React armazena algumas informações sobre cada item da lista renderizada. Quando atualizamos uma lista, o React precisa determinar o que mudou. Nós poderiamos ter adicionado, removido, rearranjado ou atualizado os itens da lista. 

Imagine uma transição de

```html
<li>Alexa: 7 tasks left</li>
<li>Ben: 5 tasks left</li>
```

para

```html
<li>Ben: 9 tasks left</li>
<li>Claudia: 8 tasks left</li>
<li>Alexa: 5 tasks left</li>
```

Além das contagens atualizadas, um humano lendo isso provavelmente iria dizer que nós trocamos a ordem de Alexa e Ben e inserimos Claudia entre eles. No entanto, React é um programa de computador e não sabe qual foi nossa intenção. Pelo fato do React não ter como saber nossas intenções, precisamos especificar uma propriedade *key* (chave) para cada item da lista para diferenciá-los entre si. Uma opção poderia ser a utilização das strings `alexa`, `ben`, `claudia`. Se tivéssemos mostrando dados a partir de um banco de dados, os ids de Alexa, Ben e Claudia no banco poderiam ser utilizados como as chaves. 

```html
<li key={user.id}>{user.name}: {user.taskCount} tasks left</li>
```

Quando uma lista é re-renderizada, o React pega cada chave e busca nos itens da lista anterior por uma chave correspondente. Se a lista atual tiver uma chave que ainda não existia, React cria um componente. Se na lista atual tiver faltando uma chave que já existia na lista anterior, React destrói o componente anterior. Se as duas chaves combinarem, o componente correspondente é movido. As chaves informam ao React sobre a identidade de cada componente, o que permite que ele mantenha o estado entre re-renderizações. Se a chave de um componente mudar, o componente será destruído e recriado com um novo estado (state).

`key` é uma propriedade especial e reservada do React (juntamente com `ref`, uma funcionalidade mais avançada). Quando um elemento é criado, React extrai a propriedade `key` e armazena como uma chave diretamente no elemento retornado. Ainda que pareça que `key` pertença a `props`, `key` não pode ser referenciado utilizando `this.props.keys`. React automaticamente utiliza `key` para decidir quais componentes atualizar. Um componente não pode acessar sua `key`.

**É fortemente recomendado que você defina adequadamente suas chaves sempre que construir listas dinâmicas**. Se não tiver uma chave apropriada, você talvez deva considerar restruturar seus dados para tê-la.

Se nenhuma chave for especificada, React vai mostrar um aviso e utilizar, por padrão, o índice do array como chave. Utilizar o índice do array como a chave é problemático quando se tenta reordenar os itens de uma lista ou inserir/remover itens. Passar `key={i}` explicitamente silencia o aviso, mas continua com os mesmos problemas dos índices do array e por isso não é recomendado na maioria dos casos.

Chaves não precisam ser globalmente únicas; elas precisam ser únicas apenas entre os componentes e seus irmãos (siblings).

### Implementando a Viagem no Tempo (Time Travel) {#implementing-time-travel}

No histórico do Jogo da Velha, cada jogada anterior tem um único ID associado a ela: é o número sequencial da jogada. As jogadas nunca são reordenadas, apagadas, ou inseridas no meio, então é seguro utilizar o index da jogada como a chave.

No método `render` do componente Game, nós podemos adicionar a chave como `<li key={move}>` e o aviso do React sobre as chaves deve desaparecer.

```js{6}
    const moves = history.map((step, move) => {
      const desc = move ?
        'Go to move #' + move :
        'Go to game start';
      return (
        <li key={move}>
          <button onClick={() => this.jumpTo(move)}>{desc}</button>
        </li>
      );
    });
```

**[Veja o código completo até aqui](https://codepen.io/gaearon/pen/PmmXRE?editors=0010)**

Clicar em quaisquer dos botões da lista vai causar um erro pois o método `jumpTo` não está definido. Antes de implementá-lo, vamos adicionar `stepNumber` ao state do componente Game para indicar qual passo estamos visualizando no momento.

Primeiro, adicione `stepNumber: 0` ao state inicial no `contructor` do componente Game. 

```js{8}
class Game extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      history: [{
        squares: Array(9).fill(null),
      }],
      stepNumber: 0,
      xIsNext: true,
    };
  }
```

Em seguida, definiremos o método `jumpTo` no componente Game para atualizar aquele `stepNumber`. Também definimos `xIsNext` para `true` caso o número que estejamos atribuindo a `stepNumber` seja par: 

```javascript{5-10}
  handleClick(i) {
    // esse método não mudou
  }

  jumpTo(step) {
    this.setState({
      stepNumber: step,
      xIsNext: (step % 2) === 0,
    });
  }

  render() {
    // esse método não mudou
  }
```

Agora faremos algumas modificações no método `handleClick` do componente Game, que é disparado quando você clica em um quadradado do tabuleiro (square).

O state `stepNumber` que adicionamos reflete a jogada mostrada ao usuário nesse momento. Após fazermos uma nova jogada, precisamos atualizar `stepNumber` adicionando `stepNumber: history.length` como parte do argumento de `this.setState`. Isso certifica que não ficaremos presos mostrando a mesma jogada após uma novo ter sido feita. 

Também iremos substituir a leitura de `this.state.history` por `this.state.history.slice(0, this.state.stepNumber + 1)`. Isso certifica que se nós "voltarmos no tempo", e então fizermos uma nova jogada a partir daquele ponto, descartamos todo o histórico do "futuro" que agora se tornaria incorreto.

```javascript{2,13}
  handleClick(i) {
    const history = this.state.history.slice(0, this.state.stepNumber + 1);
    const current = history[history.length - 1];
    const squares = current.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i] = this.state.xIsNext ? 'X' : 'O';
    this.setState({
      history: history.concat([{
        squares: squares
      }]),
      stepNumber: history.length,
      xIsNext: !this.state.xIsNext,
    });
  }
```

Por fim, modificaremos o método `render` do componente Game para deixar de renderizar sempre a última jogada e passar a renderizar apenas a jogada selecionada atualmente, de acordo com `stepNumber`:

```javascript{3}
  render() {
    const history = this.state.history;
    const current = history[this.state.stepNumber];
    const winner = calculateWinner(current.squares);

    // o resto não foi modificado
```

Se clicarmos em qualquer passo no histórico do jogo, o tabuleiro do Jogo da Velha deve atualizar imediatamente para mostrar como ficou depois que aquele passo ocorreu.

**[Veja o código completo nessa etapa](https://codepen.io/gaearon/pen/gWWZgR?editors=0010)**

### Recapitulando {#wrapping-up}

Parabéns! Você criou um jogo que:

* Te permite jogar o Jogo da Velha,
* Indica quando um dos jogadores ganhou o jogo,
* Armazena um histórico do jogo à medida que ele avança,
* Permite aos jogadores revisarem o histórico do jogo e verem versões anteriores do tabuleiro.

Belo trabalho! Esperamos que agora você esteja sentindo como se tivesse uma boa noção de como React funciona.

Dê uma olhada on resultado final aqui: **[Resultado Final](https://codepen.io/gaearon/pen/gWWZgR?editors=0010)**.

Se você tiver algum tempo extra e quiser praticar suas habilidades no React, aqui estão algumas ideias de melhorias que você poderia adicionar a seu Jogo da Velha, listadas em ordem crescente de dificuldade.

1. Mostrar a localização de cada jogada no formato (col,row), para cada jogada no histórico.
2. Estilizar com negrito o item da lista de jogadas que está selecionado no momento.
3. Reescrever o componente Board para utilizar 2 loops para fazer os quadrados, em vez de deixá-los hardcoded.
4. Adicionar um botão de toggle que lhe permita ordenar os jogadas em ordem ascendente ou descendente.
5. Quando alguém ganhar, destaque os 3 quadrados que causaram a vitória.
6. Quando ninguém ganhar, exiba uma mensagem informando que o resultado foi um empate.

Ao longo dessa tutorial, abordamos conceitos de React incluindo elementos, componentes, props e state. Para uma explicação mais detalhada de cada um desses tópicos, confira [o restante da documentação](/docs/hello-world.html). Para aprender mais sobre definição de componentes, confira a [API de Referência do `React.Component`](/docs/react-component.html).
