---
id: higher-order-components
title: Componentes de Ordem Superior
permalink: docs/higher-order-components.html
---

Um componente de ordem superior (HOC, do inglês Higher-Order Component) é uma técnica avançada do React para reutilizar a lógica de um componente. HOCs não são parte da API do React, per se. Eles são um padrão que surgiu da própria natureza de composição do React.

Concretamente, **um componente de ordem superior é uma função que recebe um componente e retorna um novo componente.**

```js
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

Enquanto um componente transforma props em UI, um componente de ordem superior transforma um componente em outro componente.

HOCs são comuns em bibliotecas externas ao React, como o [`connect`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) do Redux e o [`createFragmentContainer`](https://relay.dev/docs/v10.1.3/fragment-container/#createfragmentcontainer) do Relay.

Neste documento, nós vamos discutir porque componentes de ordem superior são úteis e como escrever o seu.

## Usar HOCs para características transversais. {#use-hocs-for-cross-cutting-concerns}

> **Nota**
>
> Anteriormente, nós recomendamos mixins como uma forma de lidar com características transversais. Desde então, percebemos que mixins criam mais problemas do que trazem valor. [Leia mais](/blog/2016/07/13/mixins-considered-harmful.html) sobre porque nós deixamos os mixins de lado e como você pode fazer a transição dos seus componentes existentes.

Componentes são a unidade primária de reutilização de código no React. Contudo, pode-se perceber que alguns padrões não se encaixam tão facilmente em componentes tradicionais.

Por exemplo, digamos que você tem um componente `CommentList` que subscreve-se a uma fonte externa de dados para renderizar uma lista de comentários:

```js
class CommentList extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      // "DataSource" é uma fonte de dados global
      comments: DataSource.getComments()
    };
  }

  componentDidMount() {
    // Subscreve-se às mudanças
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    // Limpa o listener
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    // Atualiza o state do componente sempre que a fonte de dados muda
    this.setState({
      comments: DataSource.getComments()
    });
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    );
  }
}
```

Depois, você escreve um componente que se subscreve a um blog post, o qual segue um padrão similar:

```js
class BlogPost extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      blogPost: DataSource.getBlogPost(props.id)
    };
  }

  componentDidMount() {
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id)
    });
  }

  render() {
    return <TextBlock text={this.state.blogPost} />;
  }
}
```

`CommentList` e `BlogPost` não são idênticos - eles chamam métodos diferentes de `DataSource` e renderizam saídas diferentes. Mas muito de suas implementações é igual:

- Em `mount`, adicione um change listener para `DataSource`.
- Dentro do listener, chame `setState` sempre que a fonte de dados mudar.
- Em `unmount`, remova o change listener.

Você pode imaginar que em uma aplicação grande, esse mesmo padrão de subscrever-se a `DataSource` e chamar `setState` irá ocorrer várias vezes. Nós queremos uma abstração que permita-nos definir essa lógica em um único lugar e compartilhá-la com vários componentes. Isso é onde componentes de ordem superior se destacam.

Nós podemos escrever uma função que cria componentes, como `CommentList` e `BlogPost`, que subscrevem-se a `DataSource`. A função irá aceitar como um dos seus argumentos um componente filho que recebe o dado assinado como uma prop. Vamos chamar a função de `withSubscription`:

```js
const CommentListWithSubscription = withSubscription(
  CommentList,
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost,
  (DataSource, props) => DataSource.getBlogPost(props.id)
);
```

O primeiro parâmetro é um componente encapsulado. O segundo parâmetro acessa os dados que estamos interessados, dado um `DataSource` e as props atuais.

Quando `CommentListWithSubscription` e `BlogPostWithSubscription` são renderizados, `CommentList` e `BlogPost` receberão uma prop `data` com os dados mais recentes obtidos de `DataSource`:

```js
// A função recebe um componente...
function withSubscription(WrappedComponent, selectData) {
  // ...e retorna outro componente...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ... que lida com a subscrição...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ... e renderiza o componente encapsulado com os dados novos!
      // Note que nós passamos diretamente qualquer prop adicional
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

Note que um HOC não modifica o componente de entrada, nem utiliza herança para copiar seu comportamento. Em vez disso, um HOC *compõe* o componente original ao *envolvê-lo* (wrapping) em um componente container. Um HOC é uma função pura sem efeitos colaterais.

E é isso! O componente encapsulado recebe todas as props do container, junto de uma nova prop, `data`, o qual a utiliza para renderizar sua saída. O HOC não se preocupa com o como ou o porquê de seus dados serem usados, e o componente encapsulado não se preocupa de onde os dados vieram.

Por `withSubscription` ser uma função normal, você pode adicionar quantos argumentos quiser. Por exemplo, você pode querer fazer o nome da prop `data` ser configurável, para continuar a isolar o HOC do componente encapsulado. Ou você pode aceitar um argumento que configura `shouldComponentUpdate`, ou um que configura a fonte de dados. Todos esses casos são possíveis porque o HOC tem controle total sobre como o componente é definido.

Como componentes, o contrato entre `withSubscription` e o componente encapsulado é completamente baseado em props. Isso faz com que seja fácil trocar um HOC por outro, desde que eles providenciem as mesmas props para o componente encapsulado. Isso pode ser útil se você mudar de bibliotecas para obtenção de dados, por exemplo.

## Não alterar o componente original. Usar composição. {#dont-mutate-the-original-component-use-composition}

Resista à tentação de modificar o prototype de um componente (ou alterá-lo de qualquer outra forma) dentro de um HOC.

```js
function logProps(InputComponent) {
  InputComponent.prototype.componentDidUpdate = function(prevProps) {
    console.log('Current props: ', this.props);
    console.log('Previous props: ', prevProps);
  };
  // O fato de estarmos retornando a entrada original é uma dica de que ela sofreu mutação.
  return InputComponent;
}

// EnhancedComponent criará logs sempre que uma prop for recebida
const EnhancedComponent = logProps(InputComponent);
```

Existem alguns problemas nisso. Primeiro, o componente de entrada não pode ser reutilizado separadamente do componente melhorado. Mais crucialmente, se você aplicar outro HOC para `EnhancedComponent` que *também* altera `componentDidUpdate`, a funcionalidade do primeiro HOC será sobrescrita! Esse HOC também não funcionará com componentes funcionais, os quais não possuem métodos de ciclo de vida.

Realizar mutações em HOCs podem causar "vazamentos" - o consumidor deve saber como eles são implementados para evitar conflitos com outros HOCs.

Em vez de mutações, HOCs devem utilizar composição, encapsulando o componente de entrada em um componente container:

```js
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('Current props: ', this.props);
      console.log('Previous props: ', prevProps);
    }
    render() {
      // Encapsula o componente de entrada em um container, sem alterá-lo. Excelente!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

Esse HOC possui a mesma funcionalidade que a sua versão com mutação e evita o potencial de ocorrer conflitos. Ele funciona igualmente bem com componentes funcionais e controlados. E por ser uma função pura, pode ser combinado com outros HOCs, ou até com si mesmo.

Você deve ter notado similaridades entre HOCs e um padrão chamado **componentes container**. Componentes container são parte de uma estratégia de separação de responsabilidade entre preocupações de alto nível e baixo nível. Containers gerenciam coisas como subscrições e state, e passam props para componentes que lidam com coisas como renderização da UI. HOCs utilizam containers como parte de sua implementação. Você pode pensar em HOCs como definições de componentes container com parâmetros.

## Convenção: Passar props não relacionadas para o componente encapsulado {#convention-pass-unrelated-props-through-to-the-wrapped-component}

HOCs adicionam características a um componente. Eles não devem alterar drasticamente o seu contrato. É esperado que um componente retornado de um HOC tenha uma interface similar ao do componente encapsulado.

HOCs devem passar props que não são relacionadas às suas preocupações específicas. A maioria dos HOCs possuem um método render que se parece com algo assim:

```js
render() {
  // Filtra props extras que são específicas a esse HOC e não devem ser
  // passadas para o componente encapsulado.
  const { extraProp, ...passThroughProps } = this.props;

  // Injeta props no componente encapsulado.
  // Estes geralmente são valores do state ou métodos da instância.
  const injectedProp = someStateOrInstanceMethod;

  // Passa as props para o componente encapsulado.
  return (
    <WrappedComponent
      injectedProp={injectedProp}
      {...passThroughProps}
    />
  );
}
```

Essa convenção ajuda a garantir que HOCs são tão flexíveis e reutilizáves quanto possível.

## Convenção: Maximizando composabilidade {#convention-maximizing-composability}

Nem todos os HOCs são iguais. Às vezes eles aceitam apenas um único argumento, o componente encapsulado:

```js
const NavbarWithRouter = withRouter(Navbar);
```

Normalmente, HOCs aceitam argumentos adicionais. Neste exemplo do Relay, um objeto de configuração é usado para especificar uma dependência de dados de um componente:

```js
const CommentWithRelay = Relay.createContainer(Comment, config);
```

A assinatura mais comum para HOCs se parece com isso:

```js
// `connect` do React Redux
const ConnectedComment = connect(commentSelector, commentActions)(CommentList);
```

*O Que?!* Se você quebrar isso em partes, é mais fácil de ver o que está acontecendo.

```js
// connect é uma função que retorna outra função
const enhance = connect(commentListSelector, commentListActions);
// A função retornada é um HOC, que retorna um componente que está conectado
// à store do Redux
const ConnectedComment = enhance(CommentList);
```

Em outras palavras, `connect` é uma função de ordem superior que retorna um componente de ordem superior!

Essa forma pode parecer confusa ou desnecessária, mas ela possui uma propriedade útil. HOCs com um único argumento, como o retornado pela função `connect`, possuem a assinatura `Componente => Componente`. Funções cujo tipo de saída é igual ao tipo da entrada são muito fáceis de se compor.

```js
// Em vez de fazer isso...
const EnhancedComponent = withRouter(connect(commentSelector)(WrappedComponent))

// ... Você pode usar uma conveniência de composição de funções
// compose(f, g, h) é o mesmo que (...args) => f(g(h(...args)))
const enhance = compose(
  // Ambos são HOCs com um único argumento
  withRouter,
  connect(commentSelector)
)
const EnhancedComponent = enhance(WrappedComponent)
```

(Essa mesma propriedade também permite que `connect` e outros enhancer-style HOCs sejam usados como decoradores, uma proposta experimental do JavaScript.)

A função de conveniência `compose` é disponibilizada por várias bibliotecas de terceiros, incluindo lodash (como o [`lodash.flowRight`](https://lodash.com/docs/#flowRight)), [Redux](https://redux.js.org/docs/api/compose.html), e [Ramda](https://ramdajs.com/docs/#compose).

## Convenção: Envolver o nome de exibição para melhor depuração {#convention-wrap-the-display-name-for-easy-debugging}

<<<<<<< HEAD
Os componentes container criados por HOCs aparecem no [React Developer Tools](https://github.com/facebook/react-devtools) como qualquer outro componente. Para facilitar a depuração, escolha um nome de exibição que comunique que o componente é o resultado de um HOC.
=======
The container components created by HOCs show up in the [React Developer Tools](https://github.com/facebook/react/tree/master/packages/react-devtools) like any other component. To ease debugging, choose a display name that communicates that it's the result of a HOC.
>>>>>>> 25f756d8e3800afb032cb31ce3626d6134e31e38

A técnica mais comum é encapsular o nome de exibição do componente encapsulado. Então, se o seu componente de ordem superior se chama `withSubscription`, e o nome de exibição do componente encapsulado é `CommentList`, use o nome de exibição `WithSubscription(CommentList)`:

```js
function withSubscription(WrappedComponent) {
  class WithSubscription extends React.Component {/* ... */}
  WithSubscription.displayName = `WithSubscription(${getDisplayName(WrappedComponent)})`;
  return WithSubscription;
}

function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}
```

## Ressalvas {#caveats}

Componentes de ordem superior vêm com algumas ressalvas que não são imediatamente óbvias se você for novo no React.

### Não usar HOCs dentro do método render {#dont-use-hocs-inside-the-render-method}

O algoritmo de comparação do React (chamado [Reconciliação](/docs/reconciliation.html)) usa a identidade do componente para determinar se ele deve atualizar a sub-árvore existente ou jogá-la fora e montar uma nova. Se o componente que foi retornado do `render` é idêntico (`===`) ao componente do render anterior, o React atualizará a sub-árvore comparando-a recursivamente com uma nova. Se elas não forem iguais, a sub-árvore anterior será completamente desmontada.

Normalmente, você não precisa se preocupar com isso. Mas isso é importante em HOCs porque não se pode aplicar um HOC em um componente dentro do método render de um componente:

```js
render() {
  // Uma nova versão de EnhancedComponent é criada toda vez que render for chamado
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // Isso faz com que a sub-árvore seja completamente
  // desmontada/remontada todas as vezes!
  return <EnhancedComponent />;
}
```

O problema aqui não é só na performance - remontar um componente causa a perda de todo o seu state e todos os seus filhos.

Em vez disso, aplique HOCs fora da definição do componente para que o componente resultante seja criado apenas uma vez. Então, sua identidade será consistente pelas renderizações. De qualquer forma, isso geralmente é o que você quer.

Nesses casos raros em que é preciso aplicar um HOC dinamicamente, isso também pode ser feito dentro dos métodos de ciclo de vida do componente, ou no seu construtor.

### Métodos estáticos devem ser propagados {#static-methods-must-be-copied-over}

As vezes é útil definir um método estático em um componente do React. Por exemplo, containers do Relay expõem um método estático `getFragment` para facilitar a composição de fragmentos do GraphQL.

Mas, quando se aplica um HOC a um componente, o componente original é encapsulado com um componente container. Isso significa que o novo componente não possui nenhum dos métodos estáticos do componente original.

```js
// Defina um método estático
WrappedComponent.staticMethod = function() {/*...*/}
// Agora aplique um HOC
const EnhancedComponent = enhance(WrappedComponent);

// O EnhancedComponent não possui métodos estáticos
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

Para resolver isso, você pode copiar os métodos no container antes de retorná-lo:

```js
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // Deve-se saber exatamente qual método copiar :(
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```

Porém, isso requer que você saiba exatamente quais métodos precisam ser copiados. A biblioteca [hoist-non-react-statics](https://github.com/mridgway/hoist-non-react-statics) pode ser usada para copiar automaticamente todos os métodos estáticos que não são do React:

```js
import hoistNonReactStatic from 'hoist-non-react-statics';
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
```

Outra solução possível é exportar o método estático do próprio componente, separadamente.

```js
// Em vez de...
MyComponent.someFunction = someFunction;
export default MyComponent;

// ... exporte o método separadamente...
export { someFunction };

// ... e no módulo que está consumindo, importe ambos
import MyComponent, { someFunction } from './MyComponent.js';
```

### Refs não são passadas diretamente {#refs-arent-passed-through}

Enquanto a convenção para componentes de ordem superior é passar diretamente todas as props para o componente encapsulado, isso não funciona para refs. Isso acontece porque `ref` não é exatamente uma prop - tal como `key`, ela é tratada de maneira especial pelo React. Se uma ref for adicionado a um elemento cujo componente é o resultado de um HOC, a ref referenciará à instância mais externa do componente container, não ao componente encapsulado.

A solução para esse problema é usar a API `React.forwardRef` (introduzida com o React 16.3). [Saiba mais sobre ela na seção de encaminhamento de refs](/docs/forwarding-refs.html).
