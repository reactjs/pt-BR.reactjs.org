---
id: optimizing-performance
title: Otimizando performance
permalink: docs/optimizing-performance.html
redirect_from:
  - "docs/advanced-performance.html"
---

Internamente, o React usa diversas técnicas inteligentes para minimizar o número de operações custosas de DOM que são necessárias para alterar a UI. Para muitas aplicações, utilizar React fará com que elas tenham uma rápida interface sem fazer muito esforço para otimizar performance. No entanto, existem diversas maneiras para acelerar sua aplicação React.

## Use a build de produção {#use-the-production-build}

Se você está fazendo benchmarking ou tendo problemas de performance em suas aplicações React, tenha certeza que você está testando com a build de produção.

Por padrão, o React inclui diversos avisos úteis. Esses avisos são muito úteis em desenvolvimento. Contudo, eles tornam o React maior e mais lento, então você precisa ter certeza que está usando a versão de produção quando faz a publicação de seu app.

Se você não tem certeza se seu processo de build está configurado corretamente, você pode verificar instalando a extensão [React Developer Tools para o Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi). Se você visitar um site que usa React em produção, o ícone terá uma cor de fundo escura:

<img src="../images/docs/devtools-prod.png" style="max-width:100%" alt="React DevTools em um site com a versão de produção do React">

Se você visitar um site com React em modo de desenvolvimento, o ícone terá um fundo vermelho:

<img src="../images/docs/devtools-dev.png" style="max-width:100%" alt="React DevTools em um site com a versão de desenvolvimento do React">

É esperado que você use o modo de desenvolvimento enquanto trabalha em seu app, e o modo de produção quando publicar ele para os usuários.

Você irá encontrar instruções para construir seu app para produção abaixo.

### Criando um app React (Create React App) {#create-react-app}

Se seu projeto é construído com [Create React App](https://github.com/facebookincubator/create-react-app), execute:

```
npm run build
```

Isto irá criar uma build de produção para seu app na pasta `build/` de seu projeto.

Lembre que isto é somente necessário antes de publicar para produção. Para desenvolvimento normal, use `npm start`.

### Builds de único arquivo {#single-file-builds}

Nós oferecemos versões de produção prontas do React e React DOM com arquivos únicos:

```html
<script src="https://unpkg.com/react@17/umd/react.production.min.js"></script>
<script src="https://unpkg.com/react-dom@17/umd/react-dom.production.min.js"></script>
```

Lembre que somente arquivos React terminados com `.production.min.js` são adequados para produção.

### Brunch {#brunch}

Para uma build de produção do Brunch mais eficiente, instale o plugin [`terser-brunch`](https://github.com/brunch/terser-brunch):

```
# Se você usa npm
npm install --save-dev terser-brunch

# Se você usa Yarn
yarn add --dev terser-brunch
```

Então, para criar uma build de produção, adicione o argumento `-p` no comando `build`:

```
brunch build -p
```

Lembre que você somente precisa fazer isso para builds de produção. Você não deve passar o argumento `-p` ou aplicar esse plugin em desenvolvimento, porque ele irá esconder avisos úteis do React e fará as builds mais lentas.

### Browserify {#browserify}

Para uma build de produção do Browserify mais eficiente, instale alguns plugins:

```
# Se você usa npm
npm install --save-dev envify terser uglifyify

# Se você usa Yarn
yarn add --dev envify terser uglifyify
```

Para criar uma build de produção, tenha certeza que você adicionou esses transforms **(a ordem faz diferença):**

* O [`envify`](https://github.com/hughsk/envify) assegura que o ambiente que a build está configurado é o correto. Torne ele global (`-g`).
* O [`uglifyify`](https://github.com/hughsk/uglifyify) remove os imports de desenvolvimento. Torna ele global também (`-g`).
* Finalmente, o bundle gerado é enviado para o [`terser`](https://github.com/terser-js/terser) para enxutar ([entenda o porquê](https://github.com/hughsk/uglifyify#motivationusage)).

Por exemplo:

```
browserify ./index.js \
  -g [ envify --NODE_ENV production ] \
  -g uglifyify \
  | terser --compress --mangle > ./bundle.js
```

Lembre que você somente precisar fazer isso para builds de produção. Você não deve aplicar esses plugins em desenvolvimento porque eles vão esconder avisos úteis do React, e farão as builds mais lentas.

### Rollup {#rollup}

Para uma build de produção do Rollup mais eficiente, instale alguns plugins:

```bash
# Se você usa npm
npm install --save-dev rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-terser

# Se você usa Yarn
yarn add --dev rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-terser
```

Para criar uma build de produção, tenha certeza que você adicionou esses plugins, **(a ordem faz diferença)**

* O [`replace`](https://github.com/rollup/rollup-plugin-replace) assegura que o ambiente em que a build está configurado é o correto.
* O [`commonjs`](https://github.com/rollup/rollup-plugin-commonjs) fornece suporte para CommonJS no Rollup.
* O [`terser`](https://github.com/TrySound/rollup-plugin-terser) comprime e enxuta o bundle final.

```js
plugins: [
  // ...
  require('rollup-plugin-replace')({
    'process.env.NODE_ENV': JSON.stringify('production')
  }),
  require('rollup-plugin-commonjs')(),
  require('rollup-plugin-terser')(),
  // ...
]
```

Para um exemplo completo de setup [veja esse gist](https://gist.github.com/Rich-Harris/cb14f4bc0670c47d00d191565be36bf0).

Lembre que você somente precisa fazer isso para builds de produção. Você não deve aplicar o `terser` ou o `replace` com o valor de `'production'`em desenvolvimento porque eles vão esconder avisos úteis do React, e farão as builds mais lentas.

### webpack {#webpack}

>**Observação:**
>
>Se você está usando Create React App, por favor siga [as instruções acima](#create-react-app).<br>
>Esta seção é somente relevante se você configura o webpack diretamente.

O Webpack v4+ irá diminuir seu código por padrão no modo de produção.

```js
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  mode: 'production',
  optimization: {
    minimizer: [new TerserPlugin({ /* opções adicionais aqui */ })],
  },
};
```

Você pode aprender mais sobre isso na [documentação do webpack](https://webpack.js.org/guides/production/).

Lembre que você somente precisa fazer isso para builds de produção. Você não deve aplicar `TerserPlugin` em desenvolvimento porque ele vai esconder avisos úteis do React, e farão as builds mais lentas.

## Analisando componentes com o DevTools Profiler {#profiling-components-with-the-devtools-profiler}

`react-dom` 16.5+ e `react-native` 0.57+ fornecem melhorias nas capacidades de analise em modo de desenvolvimento com o React DevTools Profiler.

Uma visão geral do Profiler pode ser encontrada nesse artigo ["Introducing the React Profiler"](/blog/2018/09/10/introducing-the-react-profiler.html).
Um vídeo com o passo a passo do profiler também está [disponível no YouTube](https://www.youtube.com/watch?v=nySib7ipZdk).

Se você ainda não tem o React DevTools instalado, você pode encontrá-lo aqui:

- [Extensão para Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)
- [Extensão para Firefox](https://addons.mozilla.org/en-GB/firefox/addon/react-devtools/)
- [Pacote separado](https://www.npmjs.com/package/react-devtools)

> Observação
>
> Uma analise de uma build de produção do `react-dom` está disponível como `react-dom/profiling`.
> Leia mais sobre como usar esse pacote no [fb.me/react-profiling](https://fb.me/react-profiling)

> Observação
>
> Antes do React 17, usamos o padrão [User Timing API](https://developer.mozilla.org/en-US/docs/Web/API/User_Timing_API) para criar o perfil de componentes com a guia de desempenho do chrome.
> Para um passo a passo mais detalhado, confira [este artigo de Ben Schwarz](https://calibreapp.com/blog/react-performance-profiling-optimization).

## Virtualizando Longas Listas {#virtualize-long-lists}

<<<<<<< HEAD
Se sua aplicação renderiza longas listas de informação (milhares ou centenas de linhas), nós recomendamos usar uma técnica conhecida como "windowing". Esta técnica somente renderiza um pequeno conjunto de suas linhas e pode reduzir drasticamente o tempo que ele leva para re-renderizar os componentes bem como o número de nós criados no DOM.
=======
If your application renders long lists of data (hundreds or thousands of rows), we recommend using a technique known as "windowing". This technique only renders a small subset of your rows at any given time, and can dramatically reduce the time it takes to re-render the components as well as the number of DOM nodes created.
>>>>>>> 25f756d8e3800afb032cb31ce3626d6134e31e38

[react-window](https://react-window.now.sh/) e [react-virtualized](https://bvaughn.github.io/react-virtualized/) são as bibliotecas de windowing mais populares. Eles fornecem diversos componentes reutilizáveis para exibir listas, grids e informações tabulares. Você pode também pode criar seu próprio componente de windowing, como [o Twitter fez](https://medium.com/@paularmstrong/twitter-lite-and-high-performance-react-progressive-web-apps-at-scale-d28a00e780a3), se você quer algo mais específico para sua aplicacão.

## Evite recompilação {#avoid-reconciliation}

O React cria e mantém sua representação interna da UI renderizada. Ele inclui os elementos do React que você retorna dos seus componentes. Essa representação evita que o React crie nós no DOM e acesse os existes sem necessidade, além do que essas operações podem ser mais lentas do que operações em objetos JavaScript. Algumas vezes esse processo é referenciado como "virtual DOM", mas ele funciona da mesma forma no React Native.

Quando uma propriedade ou estado de um componente é alterado, o React decide se uma atualização do DOM atual é necessária comparando o novo elemento retornado com o antigo. Quando eles não forem iguais, o React irá alterar o DOM.

Embora o React somente altere os nós de DOM alterados, o re-rendering ainda leva algum tempo. Em muitos casos isso não é um problema, mas se a lentidão é perceptível, você pode aumentar velocidade dele sobrescrevendo a função de lifecycle `shouldComponentUpdate`, na qual é chamada antes do processo de re-rendering começar. A implementação padrão dessa função retorna `true`, deixando o React performar a alteração:

```javascript
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

Se você sabe que em algumas situações seu componente não precisa ser alterado, você pode retornar `false` no `shouldComponentUpdate` ao invés, para pular o todo o processo de renderização, incluindo a chamada de `render()` nesse componente e seus filhos:

Na maioria dos casos, ao invés de escrever `shouldComponentUpdate()` na mão, você pode herdar do [`React.PureComponent`](/docs/react-api.html#reactpurecomponent). Ele equivale a implementação do `shouldComponentUpdate()` com uma comparação rasa entre as anteriores e novas propriedades e estados 

## shouldComponentUpdate em Ação {#shouldcomponentupdate-in-action}

Abaixo podemos ver uma sub-árvore de componentes. Para cada uma, `SCU` define o que o `shouldComponentUpdate` retorna, e `vDOMEq` indica se os elementos renderizados pelo React são equivalentes. Finalmente, o círculo de cores indica se o componente tinha de ser reconciliado ou não.

<figure><img src="../images/docs/should-component-update.png" style="max-width:100%" /></figure>

Já que `shouldComponentUpdate` retornou `false` na sub-árvore iniciada no C2, React não tentou renderizar C2, e por consequência não invocou `shouldComponentUpdate` no C4 e C5.

Para C1 e C3, `shouldComponentUpdate` retornou `true`, então o React teve que descer até as folhas para checá-los. Para o C6 `shouldComponentUpdate` retornou `true`, e já que os elementos renderizados não são iguais, o React teve que alterar o DOM.

O último caso interessante é o C8. React teve que renderizar este componente, mas já que os elementos que ele retornou eram iguais aos previamente renderizados, ele não teve que alterar o DOM. 

Note que o React somente teve de fazer mutações no DOM para o C6, no qual era inevitável. Para C8, ele abortou comparando os elementos React renderizados, e para a sub-árvore do C2 e C7, ele nem mesmo teve que comparar os elementos pois abortou no `shouldComponentUpdate`, e `render` não foi chamado.

## Exemplos {#examples}

Se seu componente muda quando as variáveis `props.color` ou `state.count` mudam, você poderia ter um `shouldComponentUpdate` que checa isso:

```javascript
class CounterButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

Nesse código, `shouldComponentUpdate` só está checando se houve alguma mudança no `props.color` ou `state.count`. Se esses valores não são alterados, o componente não é alterado. Se seu componente ficou mais complexo, você pode usar um padrão similar fazendo uma comparação rasa (shallow comparison) entre todos os fields de `props` e `state` para determinar se o componente deve ser atualizado. Esse padrão é comum o suficiente para que o React forneça um helper para usar essa lógica - apenas herde de `React.PureComponent`. Então, essa é uma maneira mais simples de alcançar a mesma coisa:

```js
class CounterButton extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

Na maior parte das vezes, você pode usar `React.PureComponent` em vez de escrever seu próprio `shouldComponentUpdate`. Ele somente faz comparações rasas, então você não pode usá-lo caso as props ou state tenham sido alteradas de uma  maneira que a comparação rasa não iria detectar.

Isso pode ser um problema com estruturas mais complexas. Por exemplo, vamos dizer que você quer um componente `ListOfWords` para renderizar uma lista de palavras separas por vírgulas, com um componente pai `WordAdder` que deixa você clicar em um botão para adicionar uma palavra para a lista. Esse código *não* faz o trabalho corretamente:

```javascript
class ListOfWords extends React.PureComponent {
  render() {
    return <div>{this.props.words.join(',')}</div>;
  }
}

class WordAdder extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      words: ['marklar']
    };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // Essa parte é um padrão ruim e causa um bug
    const words = this.state.words;
    words.push('marklar');
    this.setState({words: words});
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick} />
        <ListOfWords words={this.state.words} />
      </div>
    );
  }
}
```

O problema é que `PureComponent` vai fazer um comparação simples entre o valores antigos e novos de `this.props.words`. Já que esse código muta a lista de `words` no método `handleClick` do `WordAdder`, os antigos e novos valores de `this.props.words` serão comparados como iguais, mesmo que as atuais palavras da lista tenham mudado. A `ListOfWords` não irá alterar ainda que haja novas palavras que deveriam ser renderizadas.

## O Poder de Não Mutar Dados {#the-power-of-not-mutating-data}

A maneira mais simples desse problema não acontecer é evitar mutar valores que são usados como propriedades ou estado. Por exemplo, o método `handleClick` abaixo poderia ser reescrito usando `concat` como:

```javascript
handleClick() {
  this.setState(state => ({
    words: state.words.concat(['marklar'])
  }));
}
```

ES6 suporta a [sintaxe de espalhamento](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Spread_syntax) no qual pode fazer isso mais fácil. Se você está usando Creact React App, esta sintaxe é disponível por padrão.

```js
handleClick() {
  this.setState(state => ({
    words: [...state.words, 'marklar'],
  }));
};
```

Você pode também reescrever o código que muta os objetos para evitar mutação, em uma maneira similar. Por exemplo, vamos dizer que nós temos um objeto chamado `colormap` e nós queremos escrever uma função que muda `colormap.right` para `'blue'`. Você poderia escrever:

```js
function updateColorMap(colormap) {
  colormap.right = 'blue';
}
```

Para escrever isso sem mutar o objeto original, nós poderíamos usar o método [Object.assign](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Object/assign):

```js
function updateColorMap(colormap) {
  return Object.assign({}, colormap, {right: 'blue'});
}
```

`updateColorMap` agora retorna um novo objeto, ao invés de mutar o valor o antigo. `Object.assign` é ES6 e requer um polyfill.

[Object spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) facilita a atualização de objetos sem mutação:

```js
function updateColorMap(colormap) {
  return {...colormap, right: 'blue'};
}
```

Este recurso foi adicionado ao JavaScript no ES2018.

Se você está usando Create React App, ambos `Object.assign` e a sintaxe de espalhador de objeto estão disponíveis por padrão.

Quando você lida com objetos profundamente aninhados, atualizá-los de maneira imutável pode parecer complicado. Se você enfrentar esse problema, consulte [Immer](https://github.com/mweststrate/immer) or [immutability-helper](https://github.com/kolodny/immutability-helper). Essas bibliotecas permitem escrever código altamente legível sem perder os benefícios da imutabilidade.
