---
id: lists-and-keys
title: Listas e Chaves
permalink: docs/lists-and-keys.html
prev: conditional-rendering.html
next: forms.html
---

Primeiro, vamos rever como transformamos listas em JavaScript.

Dado o código abaixo, nós usamos a função [`map()`](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Array/map) para receber um array de `números` e dobrar o valor de cada um deles. Atribuímos o novo array retornado pela função `map()` para a variável `doubled` e imprime no console:

```javascript{2}
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((number) => number * 2);
console.log(doubled);
```

Esse código imprime `[2, 4, 6, 8, 10]` no console.

No React, transformar arrays em listas de [elementos](/docs/rendering-elements.html) é praticamente idêntico a isso.

### Renderizando Múltiplos Componentes {#rendering-multiple-components}

Você pode criar coleções de elementos e [adicioná-los no JSX](/docs/introducing-jsx.html#embedding-expressions-in-jsx) usando chaves `{}`.

Abaixo, iteramos pelo array `numbers` usando a função [`map()`](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Array/map) do JavaScript. Retornamos um elemento `<li>` para cada item. Finalmente, atribuímos o array de elementos resultante para `listItems`:

```javascript{2-4}
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);
```

Adicionamos todo o array `listItems` dentro de um elemento `<ul>` e [renderizamos ele no DOM](/docs/rendering-elements.html#rendering-an-element-into-the-dom):

```javascript{2}
ReactDOM.render(
  <ul>{listItems}</ul>,
  document.getElementById('root')
);
```

[**Experimente no CodePen**](https://codepen.io/gaearon/pen/GjPyQr?editors=0011)

Esse código mostra uma lista não ordenada de números entre 1 e 5.

### Componente de Lista Básico {#basic-list-component}

Geralmente você irá renderizar listas dentro de um [componente](/docs/components-and-props.html).

Podemos refatorar o exemplo anterior em um componente que aceita um array de `números` e retorna uma lista de elementos.

```javascript{3-5,7,13}
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

Ao executar esse código, você receberá um aviso que uma chave deve ser definida para os itens da lista. `key` é um atributo string especial que você precisa definir ao criar listas de elementos. Iremos analisar os motivos pelos quais isso é importante na próxima seção.

Vamos atribuir uma `key` aos itens da nossa lista dentro de `numbers.map()` e resolver o valor da chave que está em falta.

```javascript{4}
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

[**Experimente no CodePen**](https://codepen.io/gaearon/pen/jrXYRR?editors=0011)

## Chaves {#keys}

As chaves ajudam o React a identificar quais itens sofreram alterações, foram adicionados ou removidos. As chaves devem ser atribuídas aos elementos dentro do array para dar uma identidade estável aos elementos:

```js{3}
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```

A melhor forma de escolher uma chave é usar uma string que identifica unicamente um item da lista dentre os demais. Na maioria das vezes você usaria IDs de seus dados como chave:

```js{2}
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```

Quando você não possui nenhum ID estável para os itens renderizados, você pode usar o índice do item como chave em último recurso:

```js{2,3}
const todoItems = todos.map((todo, index) =>
  // Apenas faça isso caso os itens não possuam IDs estáveis
  <li key={index}>
    {todo.text}
  </li>
);
```

Não recomendamos o uso de índices para chave se a ordem dos itens pode ser alterada. Isso pode impactar de forma negativa o desempenho e poderá causar problemas com o estado do componente. Leia o artigo escrito por Robin Pokorny para [uma explicação aprofundada nos impactos negativos de se usar um índice como chave](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318). Se você não atribuir uma chave de forma explícita para os itens de uma lista, então o React irá utilizar os índices como chave por padrão.

Aqui você poderá ver [uma explicação aprofundada sobre o porquê o uso das chaves é necessário](/docs/reconciliation.html#recursing-on-children) caso você esteja interessado em aprender mais sobre isso.

### Extraindo Componentes com Chaves {#extracting-components-with-keys}

As chaves apenas fazem sentido no contexto do array que está encapsulando os itens.

Por exemplo, se você [extrai](/docs/components-and-props.html#extracting-components) um componente `ListItem`, você deve deixar a chave nos elementos `<ListItem />` ao invés de deixar no elemento `<li>` dentro de `ListItem`.

**Exemplo: Uso Incorreto de Chaves**

```javascript{4,5,14,15}
function ListItem(props) {
  const value = props.value;
  return (
    // Errado! Não há necessidade de definir a chave aqui:
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Errado! A chave deveria ser definida aqui:
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

**Exemplo: Uso Correto de Chaves**

```javascript{2,3,9,10}
function ListItem(props) {
  // Correto! Não há necessidade de definir a chave aqui:
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Correto! A chave deve ser definida dentro do array.
    <ListItem key={number.toString()} value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

[**Experimente no CodePen**](https://codepen.io/gaearon/pen/ZXeOGM?editors=0010)

Por via de regra, os elementos dentro de uma função `map()` devem especificar chaves.

### Chaves devem ser Únicas apenas entre Elementos Irmãos {#keys-must-only-be-unique-among-siblings}

<<<<<<< HEAD
Chaves usadas nos arrays devem ser únicas entre seus elementos irmãos. Contudo elas não precisam ser únicas globalmente. Podemos usar as mesmas chaves ao criar dois arrays diferentes:
=======
Keys used within arrays should be unique among their siblings. However, they don't need to be globally unique. We can use the same keys when we produce two different arrays:
>>>>>>> 25f756d8e3800afb032cb31ce3626d6134e31e38

```js{2,5,11,12,19,21}
function Blog(props) {
  const sidebar = (
    <ul>
      {props.posts.map((post) =>
        <li key={post.id}>
          {post.title}
        </li>
      )}
    </ul>
  );
  const content = props.posts.map((post) =>
    <div key={post.id}>
      <h3>{post.title}</h3>
      <p>{post.content}</p>
    </div>
  );
  return (
    <div>
      {sidebar}
      <hr />
      {content}
    </div>
  );
}

const posts = [
  {id: 1, title: 'Hello World', content: 'Welcome to learning React!'},
  {id: 2, title: 'Installation', content: 'You can install React from npm.'}
];
ReactDOM.render(
  <Blog posts={posts} />,
  document.getElementById('root')
);
```

[**Experimente no CodePen**](https://codepen.io/gaearon/pen/NRZYGN?editors=0010)

As chaves servem como uma dica para o React. Mas elas não são passadas para os componentes. Se você precisar do mesmo valor em um componente, defina ele explicitamente como uma `prop` com um nome diferente:

```js{3,4}
const content = posts.map((post) =>
  <Post
    key={post.id}
    id={post.id}
    title={post.title} />
);
```

No exemplo acima, o componente `Post` pode acessar `props.id`. Mas, não pode acessar `props.key`.

### Incluindo map() no JSX {#embedding-map-in-jsx}

Nos exemplos acima declaramos uma variável `listItems` separada e adicionamos ela no JSX:

```js{3-6}
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <ListItem key={number.toString()}
              value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

O JSX permite [incluir qualquer expressão](/docs/introducing-jsx.html#embedding-expressions-in-jsx) dentro de chaves, então podemos adicionar o resultado do `map()` diretamente:

```js{5-8}
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />
      )}
    </ul>
  );
}
```

[**Experimente no CodePen**](https://codepen.io/gaearon/pen/BLvYrB?editors=0010)

Às vezes isso resulta em um código mais limpo. Mas esse padrão também pode ser confuso. Como em JavaScript, depende de você decidir se vale a pena extrair uma variável para aumentar a legibilidade. Lembre-se que se o corpo da função `map()` tiver muitos níveis, poderá ser um bom momento para [extrair um componente](/docs/components-and-props.html#extracting-components).
