---
id: concurrent-mode-suspense
title: Suspense para Busca de Dados (Experimental)
permalink: docs/concurrent-mode-suspense.html
prev: concurrent-mode-intro.html
next: concurrent-mode-patterns.html
---

<style>
.scary > blockquote {
  background-color: rgba(237, 51, 21, 0.2);
  border-left-color: #ed3315;
}
</style>

<div class="scary">

>Cuidado:
>
<<<<<<< HEAD
>Esta página descreve **recursos experimentais que [ainda não estão disponíveis](/docs/concurrent-mode-adoption.html) em uma versão estável**. Não confie nas versões experimentais do React em aplicativos de produção. Esses recursos podem mudar significativamente e sem aviso antes de se tornarem parte do React.
>
>Esta documentação é destinada a adotantes precoces e pessoas curiosas. **Se você é novo no React, não se preocupe com esses recursos** -- você não precisa aprendê-los agora. Por exemplo, se você estiver procurando por um tutorial de busca de dados que funcione hoje, leia [este artigo](https://www.robinwieruch.de/react-hooks-fetch-data/).
=======
>This page was about experimental features that aren't yet available in a stable release. It was aimed at early adopters and people who are curious.
>
>Much of the information on this page is now outdated and exists only for archival purposes. **Please refer to the [React 18 Alpha announcement post](/blog/2021/06/08/the-plan-for-react-18.html
) for the up-to-date information.**
>
>Before React 18 is released, we will replace this page with stable documentation.
>>>>>>> 25f756d8e3800afb032cb31ce3626d6134e31e38

</div>

O React 16.6 adicionou um componente `<Suspense>` que permite você "esperar" para que algum código seja carregado e especifique declarativamente um estado de carregamento (como um spinner) enquanto esperamos:

```jsx
const ProfilePage = React.lazy(() => import('./ProfilePage')); // Carregado quando necessário

// Mostrar um spinner enquanto o perfil está carregando
<Suspense fallback={<Spinner />}>
  <ProfilePage />
</Suspense>
```

Suspense para Busca de Dados é um novo recurso que permite usar `<Suspense>` também para **declarativamente "esperar" por qualquer outra coisa, incluindo dados.** Esta página se concentra no caso de uso de busca de dados, mas também pode aguardar imagens, scripts ou outro recurso assíncrono.

- [O Que É Suspense, Exatamente?](#what-is-suspense-exactly)
  - [O Que Suspense Não É](#what-suspense-is-not)
  - [O Que O Suspense Permite Que Você Faça](#what-suspense-lets-you-do)
- [Usando Suspense na Prática](#using-suspense-in-practice)
  - [E Se Eu Não Uso Relay?](#what-if-i-dont-use-relay)
  - [Para Autores de Biblioteca](#for-library-authors)
- [Abordagens Tradicionais vs Suspense](#traditional-approaches-vs-suspense)
  - [Abordagem 1: Busca-na-Renderização (sem usar Suspense)](#approach-1-fetch-on-render-not-using-suspense)
  - [Abordagem 2: Busca-Então-Renderiza (sem usar Suspense)](#approach-2-fetch-then-render-not-using-suspense)
  - [Abordagem 3: Renderização-Conforme-Você-Busca (usando Suspense)](#approach-3-render-as-you-fetch-using-suspense)
- [Comece a Buscar Cedo](#start-fetching-early)
  - [Ainda Estamos Descobrindo Isso](#were-still-figuring-this-out)
- [Suspense e Condições de Concorrência](#suspense-and-race-conditions)
  - [Condições de Concorrência com useEffect](#race-conditions-with-useeffect)
  - [Condições de Concorrência com componentDidUpdate](#race-conditions-with-componentdidupdate)
  - [O Problema](#the-problem)
  - [Resolvendo Condições de Concorrência com Suspense](#solving-race-conditions-with-suspense)
- [Tratando Erros](#handling-errors)
- [Próximos Passos](#next-steps)

## O Que É Suspense, Exatamente? {#what-is-suspense-exactly}

O Suspense permite que seus componentes "esperem" por algo antes que eles possam renderizar. [Neste exemplo](https://codesandbox.io/s/frosty-hermann-bztrp), dois componentes aguardam uma chamada de API assíncrona para buscar alguns dados:

```js
const resource = fetchProfileData();

function ProfilePage() {
  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails />
      <Suspense fallback={<h1>Loading posts...</h1>}>
        <ProfileTimeline />
      </Suspense>
    </Suspense>
  );
}

function ProfileDetails() {
  // Tenta ler as informações do usuário, embora ele ainda não tenha sido carregado
  const user = resource.user.read();
  return <h1>{user.name}</h1>;
}

function ProfileTimeline() {
  // Tenta ler as postagens, embora elas ainda não tenham sido carregadas
  const posts = resource.posts.read();
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```

**[Experimente no CodeSandbox](https://codesandbox.io/s/frosty-hermann-bztrp)**

Esta demo é um teaser. Não se preocupe se ainda não faz sentido. Falaremos mais sobre como isso funciona abaixo. Lembre-se de que o Suspense é mais um *mecanismo*, e APIs específicas como `fetchProfileData()` ou `resource.posts.read()` no exemplo acima não são muito importantes. Se você estiver curioso, poderá encontrar as definições deles no [sandbox de demonstração](https://codesandbox.io/s/frosty-hermann-bztrp).

Suspense não é uma biblioteca de busca de dados. É um **mecanismo para as bibliotecas de busca de dados** para comunicar o React que *os dados que um componente está lendo ainda não estão prontos*. O React pode esperar que esteja pronto e atualizar a UI. No Facebook, usamos o Relay e sua [nova integração com Suspense](https://relay.dev/docs/en/experimental/step-by-step). Esperamos que outras bibliotecas como Apollo possam fornecer integrações semelhantes.

A longo prazo, pretendemos que o Suspense se torne a principal maneira de ler dados assíncronos dos componentes -- não importa de onde esses dados sejam provenientes.

### O Que Suspense Não É {#what-suspense-is-not}

O Suspense é significativamente diferente das abordagens existentes para esses problemas, portanto, ler sobre isso pela primeira vez geralmente leva a conceitos errôneos. Vamos esclarecer os mais comuns:

 * **Não é uma implementação de busca de dados.** Ele não pressupõe que você use GraphQL, REST ou qualquer outro formato de dado específico, biblioteca, transporte ou protocolo.

 * **Não é um cliente pronto para uso.** Você não pode "substituir" `fetch` ou Relay com Suspense. Mas você pode usar uma biblioteca integrada com o Suspense (por exemplo, [as novas APIs do Relay](https://relay.dev/docs/en/experimental/api-reference)).

 * **Ele não acopla a busca de dados à camada de visualização.** Ele ajuda a orquestrar a exibição dos states de carregamento na sua UI, mas não vincula sua lógica de rede aos componentes do React.

### O Que O Suspense Permite Que Você Faça {#what-suspense-lets-you-do}

Então, qual é o sentido do Suspense? Existem algumas maneiras de responder a isso:

* **Permite que as bibliotecas de busca de dados se integrem profundamente ao React.** Se uma biblioteca de busca de dados implementa o suporte ao Suspense, usá-lo nos componentes do React parece muito natural.

* **Permite orquestrar states de carregamento projetados intencionalmente.** Ele não diz _como_ os dados são buscados, mas permite controlar de perto a sequência de carregamento visual do seu aplicativo.

* **Ajuda a evitar condições de concorrência.** Mesmo com o `await`, o código assíncrono geralmente está sujeito a erros. O Suspense parece mais com a leitura de dados de forma *síncrona* — como se já estivesse carregado.

## Usando Suspense na Prática {#using-suspense-in-practice}

No Facebook, até agora, usamos apenas a integração do Relay com o Suspense em produção. **Se você está procurando um guia prático para começar hoje, [confira o Guia do Relay](https://relay.dev/docs/en/experimental/step-by-step)!** Ele demonstra padrões que já funcionaram bem para nós em produção.

**As demos de código desta página usam uma implementação de API "fake" no lugar do Relay.** Isso os torna mais fáceis de entender se você não estiver familiarizado com o GraphQL, mas eles não mostrarão o "caminho certo" para criar um aplicativo com o Suspense. Esta página é mais conceitual e tem como objetivo ajudá-lo a entender *por que* o Suspense funciona de uma certa maneira e quais problemas ele resolve.

### E Se Eu Não Uso Relay? {#what-if-i-dont-use-relay}

Se você não usa o Relay hoje, pode ser necessário aguardar antes de experimentar o Suspense no seu aplicativo. Até agora, é a única implementação que testamos em produção e confiamos.

Nos próximos meses, muitas bibliotecas aparecerão com diferentes usos das APIs do Suspense. **Se você prefere aprender quando as coisas estão mais estáveis, pode preferir ignorar esse trabalho por enquanto e voltar quando o ecossistema do Suspense estiver mais maduro.**

Você também pode escrever sua própria integração para uma biblioteca de busca de dados, se desejar.

### Para Autores de Bibliotecas {#for-library-authors}

Esperamos ver muitas experiências na comunidade com outras bibliotecas. Há uma coisa importante a ser observada para os autores de bibliotecas de busca de dados.

Embora seja tecnicamente possível, o Suspense **não** se destina atualmente como uma maneira de começar a buscar dados quando um componente é renderizado. Em vez disso, permite que os componentes expressem que estão "aguardando" os dados que *já estão sendo buscados*. **[Criando Excelentes Experiências de Usuário com o Modo de Concorrência e Suspense](/blog/2019/11/06/building-great-user-experiences-with-concurrent-mode-and-suspense.html)descreve por que isso é importante e como implementar esse padrão na prática.**

A menos que você tenha uma solução que ajude a evitar cascatas, sugerimos que você prefira APIs que favorecem ou reforçam a busca antes da renderização. Para um exemplo concreto, você pode ver como [API de Relay Suspense](https://relay.dev/docs/en/experimental/api-reference#usepreloadedquery) impõe o pré-carregamento. Nossas mensagens sobre isso não foram muito consistentes no passado. O Suspense para Busca de Dados ainda é experimental, portanto, você pode esperar que nossas recomendações mudem com o tempo, à medida que aprendemos mais sobre o uso em produção e entendemos melhor o espaço do problema.

## Abordagens Tradicionais vs Suspense {#traditional-approaches-vs-suspense}

Poderíamos introduzir o Suspense sem mencionar as abordagens populares de busca de dados. No entanto, isso torna mais difícil ver quais problemas o Suspense resolve, por que vale a pena resolvê-los e como o Suspense é diferente das soluções existentes.

Em vez disso, veremos o Suspense como um próximo passo lógico em uma sequência de abordagens:

* **Busca-na-renderização (por exemplo, `fetch` em `useEffect`):** Comece a renderizar componentes. Cada um desses componentes pode acionar a busca de dados em seus efeitos e métodos de ciclo de vida. Essa abordagem geralmente leva a "cascatas".
* **Busca-Então-Renderiza (por exemplo, Relay sem Suspense):** Comece a buscar todos os dados para a próxima tela o mais cedo possível. Quando os dados estiverem prontos, renderize a nova tela. Não podemos fazer nada até que os dados cheguem.
* **Renderização-conforme-você-busca (por exemplo, Relay com Suspense):** Comece a buscar todos os dados necessários para a próxima tela o mais cedo possível e comece a renderizar a nova tela *imediatamente — antes de recebermos uma resposta da rede*. À medida que os dados entram, o React tenta renderizar novamente os componentes que ainda precisam dos dados até que estejam prontos.

>Nota
>
>Isso é um pouco simplificado e, na prática, as soluções tendem a usar uma mistura de diferentes abordagens. Ainda assim, vamos analisá-los isoladamente para contrastar melhor seus prós e contras.

Para comparar essas abordagens, implementaremos uma página de perfil com cada uma delas.

### Abordagem 1: Busca-na-Renderização (sem usar Suspense) {#approach-1-fetch-on-render-not-using-suspense}

Uma maneira comum de buscar dados nos aplicativos React hoje é usar um efeito:

```js
// Utilizando componente de função:
useEffect(() => {
  fetchSomething();
}, []);

// Ou, utilizando componente de classe:
componentDidMount() {
  fetchSomething();
}
```

Chamamos essa abordagem de "busca-na-renderização" porque ela não começa a buscar até que o componente seja renderizado na tela. Isso leva a um problema conhecido como "cascata".

Considere estes components `<ProfilePage>` e `<ProfileTimeline>`:

```js{4-6,22-24}
function ProfilePage() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser().then(u => setUser(u));
  }, []);

  if (user === null) {
    return <p>Loading profile...</p>;
  }
  return (
    <>
      <h1>{user.name}</h1>
      <ProfileTimeline />
    </>
  );
}

function ProfileTimeline() {
  const [posts, setPosts] = useState(null);

  useEffect(() => {
    fetchPosts().then(p => setPosts(p));
  }, []);

  if (posts === null) {
    return <h2>Loading posts...</h2>;
  }
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```

**[Experimente no CodeSandbox](https://codesandbox.io/s/fragrant-glade-8huj6)**

Se você executar esse código e olhar os logs no console, notará que a sequência é:

1. Começamos a buscar detalhes do usuário
2. Nós esperamos...
3. Terminamos a busca dos detalhes do usuário
4. Começamos a buscar postagens
5. Nós esperamos...
6. Terminamos a busca das postagens

Se a busca de detalhes do usuário demorar três segundos, *começaremos* a buscar as postagens após três segundos! Isso é uma "cascata": uma *sequência* não intencional que deveria ter sido paralelizada.

Cascatas são comuns no código que busca dados na renderização. É possível resolver, mas à medida que o produto cresce, muitas pessoas preferem usar uma solução que proteja contra esse problema.

### Abordagem 2: Busca-Então-Renderiza (sem usar Suspense) {#approach-2-fetch-then-render-not-using-suspense}

As bibliotecas podem impedir cascatas, oferecendo uma maneira mais centralizada de buscar dados. Por exemplo, o Relay resolve esse problema movendo as informações sobre os dados que um componente precisa para analisar estaticamente *fragments*, que mais tarde são compostos em uma única consulta.

Nesta página, não assumimos o conhecimento de Relay, portanto não o usaremos neste exemplo. Em vez disso, escreveremos algo semelhante manualmente combinando nossos métodos de busca de dados:

```js
function fetchProfileData() {
  return Promise.all([
    fetchUser(),
    fetchPosts()
  ]).then(([user, posts]) => {
    return {user, posts};
  })
}
```

Neste exemplo, `<ProfilePage>` espera as duas requisições, mas inicia-os em paralelo:

```js{1,2,8-13}
// Comece a buscar o mais cedo possível
const promise = fetchProfileData();

function ProfilePage() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState(null);

  useEffect(() => {
    promise.then(data => {
      setUser(data.user);
      setPosts(data.posts);
    });
  }, []);

  if (user === null) {
    return <p>Loading profile...</p>;
  }
  return (
    <>
      <h1>{user.name}</h1>
      <ProfileTimeline posts={posts} />
    </>
  );
}

// O filho não aciona mais a busca
function ProfileTimeline({ posts }) {
  if (posts === null) {
    return <h2>Loading posts...</h2>;
  }
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```

**[Experimente no CodeSandbox](https://codesandbox.io/s/wandering-morning-ev6r0)**

A sequência de eventos agora fica assim:

1. Começamos a buscar detalhes do usuário
2. Começamos a buscar postagens
3. Nós esperamos...
4. Terminamos a busca dos detalhes do usuário
5. Terminamos a busca das postagens

Resolvemos a "cascata" de rede anterior, mas acidentalmente introduzimos uma diferente. Esperamos que *todos* os dados retornem com `Promise.all()` dentro de `fetchProfileData`, portanto, agora não podemos renderizar os detalhes do perfil até que as postagens também sejam buscadas. Temos que esperar pelos dois.

Obviamente, isso é possível de ser corrigido neste exemplo em particular. Podemos remover a chamada `Promise.all()` e aguardar as duas Promises separadamente. No entanto, essa abordagem fica progressivamente mais difícil à medida que a complexidade da nossa árvore de dados e componentes aumenta. É difícil escrever componentes confiáveis quando partes arbitrárias da árvore de dados podem estar ausentes ou obsoletas. Portanto, buscar todos os dados para a nova tela e *depois* renderizar é frequentemente uma opção mais prática.

### Abordagem 3: Renderização-Conforme-Você-Busca (usando Suspense) {#approach-3-render-as-you-fetch-using-suspense}

Na abordagem anterior, buscamos dados antes de chamarmos `setState`:

1. Começamos a buscar
2. Terminamos a busca
3. Começamos a renderização

Com o Suspense, ainda começamos a buscar primeiro, mas nós invertemos de posição os dois últimos passos:

1. Começamos a buscar
2. **Começamos a renderização**
3. **Terminamos a busca**

**Com o Suspense, não esperamos a resposta voltar antes de começarmos a renderizar.** De fato, começamos a renderizar *praticamente imediatamente* após iniciar a requisição na rede:

```js{2,17,23}
// Isto não é uma Promise. É um objeto especial da nossa integração com Suspense.
const resource = fetchProfileData();

function ProfilePage() {
  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails />
      <Suspense fallback={<h1>Loading posts...</h1>}>
        <ProfileTimeline />
      </Suspense>
    </Suspense>
  );
}

function ProfileDetails() {
  // Tenta ler as informações do usuário, embora ele ainda não tenha sido carregado
  const user = resource.user.read();
  return <h1>{user.name}</h1>;
}

function ProfileTimeline() {
  // Tenta ler as postagens, embora elas ainda não tenham sido carregadas
  const posts = resource.posts.read();
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```

**[Experimente no CodeSandbox](https://codesandbox.io/s/frosty-hermann-bztrp)**

Aqui está o que acontece quando renderizamos `<ProfilePage>` na tela:

1. Já iniciamos as requisições em `fetchProfileData()`. Isso nos deu um "resource" especial em vez de uma Promise. Em um exemplo realista, ele seria fornecido pela integração do Suspense com a nossa biblioteca de dados, como o Relay.
2. O React tenta renderizar `<ProfilePage>`. Retorna `<ProfileDetails>` e `<ProfileTimeline>` como filhos.
3. O React tenta renderizar `<ProfileDetails>`. Ele chama `resource.user.read()`. Como nenhum dos dados foi buscado ainda, esse componente "suspende". O React ignora ele e tenta renderizar outros componentes na árvore.
4. O React tenta renderizar `<ProfileTimeline>`. Ele chama `resource.posts.read()`. Novamente, ainda não há dados, portanto esse componente também "suspende". O React também ignora ele e tenta renderizar outros componentes na árvore.
5. Não há mais nada para tentar renderizar. Como o `<ProfileDetails>` foi suspenso, o React mostra o fallback do `<Suspense>` mais próximo acima dele na árvore: `<h1>Loading profile...</h1>`. Nós terminamos por enquanto.

Este objeto `resource` representa os dados que ainda não existem, mas que podem eventualmente ser carregados. Quando chamamos `read()`, obtemos os dados ou o componente "suspende".

**À medida que mais dados fluem, o React tenta novamente a renderização e cada vez pode progredir "mais fundo".** Quando o `resource.user` é buscado, o componente `<ProfileDetails>` será renderizado com êxito e não precisaremos mais do fallback `<h1>Loading profile...</h1>`. Eventualmente, obteremos todos os dados e não haverá fallbacks na tela.

Isso tem uma implicação interessante. Mesmo se usarmos um cliente GraphQL que coleta todos os requisitos de dados em uma única requisição, *o streaming da resposta nos permite mostrar mais conteúdo mais rapidamente*. Como renderizamos-*à-medida-que-buscamos* (ao contrário de *depois* de buscar), se `user` aparece na resposta antes dos `posts`, poderemos "desbloquear" o boundary externo do `<Suspense>` antes que a resposta termine. Podemos ter esquecido isso antes, mas mesmo a solução busca-então-renderiza continha uma cascata: entre buscar e renderizar. O Suspense não sofre inerentemente dessa cascata, e bibliotecas como o Relay tiram proveito disso.

Observe como eliminamos as verificações `if (...)` "carregando" de nossos componentes. Isso não apenas remove código repetitivo, mas também simplifica fazermos alterações rápidas no design. Por exemplo, se quisermos que os detalhes e as postagens do perfil sempre "apareçam" juntos, poderemos excluir o boundary do `<Suspense>` entre eles. Ou poderíamos torná-los independentes um do outro, atribuindo a cada um *seu próprio* boundary `<Suspense>`. O Suspense nos permite alterar a granularidade de nossos states de carregamento e orquestrar seu sequenciamento sem alterações invasivas em nosso código.

## Comece a Buscar Cedo {#start-fetching-early}

Se você estiver trabalhando em uma biblioteca de busca de dados, há um aspecto crucial do Render-as-You-Fetch que você não deseja perder. **Iniciamos a busca _antes_ da renderização.** Veja este exemplo de código mais de perto:

```js
// Comece a buscar cedo!
const resource = fetchProfileData();

// ...

function ProfileDetails() {
  // Tente ler as informações do usuário
  const user = resource.user.read();
  return <h1>{user.name}</h1>;
}
```

**[Experimente no CodeSandbox](https://codesandbox.io/s/frosty-hermann-bztrp)**

Note que a chamada `read()` neste exemplo não *inicia* a busca. Ele apenas tenta ler os dados **que já estão sendo buscados**. Essa diferença é crucial para criar aplicativos rápidos com o Suspense. Não queremos atrasar o carregamento de dados até que um componente comece a renderizar. Como autor da biblioteca de busca de dados, você pode impor isso, tornando impossível obter um objeto `resource` sem também iniciar uma busca. Todas as demonstrações nesta página usando nossa "API falsa" impõem isso.

Você pode objetar que a busca "no nível superior", como neste exemplo, é impraticável. O que faremos se navegarmos para a página de outro perfil? Podemos querer buscar com base em adereços. A resposta é **queremos começar a buscar os manipuladores de eventos**. Aqui está um exemplo simplificado de navegação entre as páginas do usuário:

```js{1,2,10,11}
// Primeira busca: o mais rápido possível
const initialResource = fetchProfileData(0);

function App() {
  const [resource, setResource] = useState(initialResource);
  return (
    <>
      <button onClick={() => {
        const nextUserId = getNextId(resource.userId);
        // Próxima busca: quando o usuário clicar
        setResource(fetchProfileData(nextUserId));
      }}>
        Next
      </button>
      <ProfilePage resource={resource} />
    </>
  );
}
```

**[Experimente no CodeSandbox](https://codesandbox.io/s/infallible-feather-xjtbu)**

Com essa abordagem, podemos **buscar código e dados em paralelo**. Quando navegamos entre as páginas, não precisamos esperar o código de uma página carregar para começar a carregar seus dados. Podemos começar a buscar código e dados ao mesmo tempo (durante o clique no link), proporcionando uma experiência de usuário muito melhor.

Isso coloca uma questão de como sabemos *o que* buscar antes de renderizar a próxima tela. Existem várias maneiras de resolver isso (por exemplo, integrando a busca de dados mais próxima à sua solução de roteamento). Se você trabalha em uma biblioteca de busca de dados, [Criando Excelentes Experiências de Usuário com o Modo de Concorrência e Suspense](/blog/2019/11/06/building-great-user-experiences-with-concurrent-mode-and-suspense.html) apresenta um mergulho profundo sobre como fazer isso e por que é importante.

### Ainda Estamos Descobrindo Isso {#were-still-figuring-this-out}

O próprio Suspense como mecanismo é flexível e não possui muitas restrições. O código do produto precisa ser mais restrito para garantir que não haja cascatas, mas existem diferentes maneiras de fornecer essas garantias. Algumas perguntas que estamos explorando atualmente incluem:

* Buscar cedo pode ser complicado de expressar. Como tornamos mais fácil evitar cascatas?
* Quando buscamos dados para uma página, a API pode incentivar a inclusão de dados para transições instantâneas *dela*?
* Qual é o tempo de vida de uma resposta? O cache deve ser global ou local? Quem gerencia o cache?
* Os proxies podem ajudar a expressar APIs carregadas lentamente sem inserir chamadas `read()` por todo o lado?
* Como seria a composição de consultas GraphQL equivalentes para dados arbitrários com Suspense?

O Relay tem suas próprias respostas para algumas dessas perguntas. Certamente, existe mais do que uma maneira única de fazê-lo, e estamos empolgados em ver que novas idéias a comunidade React apresentará.

## Suspense e Condições de Concorrência {#suspense-and-race-conditions}

As condições de concorrência são erros que ocorrem devido a suposições incorretas sobre a ordem em que nosso código pode ser executado. A busca de dados no Hook `useEffect` ou nos métodos de ciclo de vida de classe como `componentDidUpdate` geralmente os leva a eles. O Suspense também pode ajudar aqui — vamos ver como.

Para demonstrar o problema, adicionaremos um componente `<App>` de nível superior que renderiza nosso `<ProfilePage>` com um botão que permite **alternar entre perfis diferentes**:

```js{9-11}
function getNextId(id) {
  // ...
}

function App() {
  const [id, setId] = useState(0);
  return (
    <>
      <button onClick={() => setId(getNextId(id))}>
        Next
      </button>
      <ProfilePage id={id} />
    </>
  );
}
```

Vamos comparar como diferentes estratégias de busca de dados lidam com esse requisito.

### Condições de Concorrência com `useEffect` {#race-conditions-with-useeffect}

Primeiro, tentaremos uma versão do nosso exemplo original de "busca no efeito". Vamos modificá-lo para passar um parâmetro `id` das props de `<ProfilePage>` para `fetchUser(id)` e `fetchPosts(id)`:

```js{1,5,6,14,19,23,24}
function ProfilePage({ id }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser(id).then(u => setUser(u));
  }, [id]);

  if (user === null) {
    return <p>Loading profile...</p>;
  }
  return (
    <>
      <h1>{user.name}</h1>
      <ProfileTimeline id={id} />
    </>
  );
}

function ProfileTimeline({ id }) {
  const [posts, setPosts] = useState(null);

  useEffect(() => {
    fetchPosts(id).then(p => setPosts(p));
  }, [id]);

  if (posts === null) {
    return <h2>Loading posts...</h2>;
  }
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```

**[Experimente no CodeSandbox](https://codesandbox.io/s/nervous-glade-b5sel)**

Observe como também mudamos as dependências do efeito de `[]` para `[id]` — porque queremos que o efeito seja executado novamente quando o `id` mudar. Caso contrário, não buscaremos novamente os dados.

Se tentarmos esse código, a princípio pode parecer que ele funcione. No entanto, se aleatorizarmos o tempo de atraso em nossa implementação da "fake API" e pressionarmos o botão "Next" com rapidez suficiente, veremos nos logs do console que algo está saindo muito errado. **Às vezes, as requisições dos perfis anteriores podem "voltar" depois que já mudamos o perfil para outro ID -- e, nesse caso, elas podem sobrescrever o novo state com uma resposta obsoleta para um ID diferente.**

É possível corrigir esse problema (você pode usar a função de limpeza de efeito para ignorar ou cancelar requisições obsoletas), mas é pouco intuitivo e difícil de depurar.

### Condições de Concorrência com `componentDidUpdate` {#race-conditions-with-componentdidupdate}

Pode-se pensar que este é um problema específico para `useEffect` ou Hooks. Talvez se portarmos esse código para classes ou usarmos uma sintaxe conveniente como `async` / `await`, isso resolverá o problema?

Vamos tentar isso:

```js
class ProfilePage extends React.Component {
  state = {
    user: null,
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.fetchData(this.props.id);
    }
  }
  async fetchData(id) {
    const user = await fetchUser(id);
    this.setState({ user });
  }
  render() {
    const { id } = this.props;
    const { user } = this.state;
    if (user === null) {
      return <p>Loading profile...</p>;
    }
    return (
      <>
        <h1>{user.name}</h1>
        <ProfileTimeline id={id} />
      </>
    );
  }
}

class ProfileTimeline extends React.Component {
  state = {
    posts: null,
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.fetchData(this.props.id);
    }
  }
  async fetchData(id) {
    const posts = await fetchPosts(id);
    this.setState({ posts });
  }
  render() {
    const { posts } = this.state;
    if (posts === null) {
      return <h2>Loading posts...</h2>;
    }
    return (
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.text}</li>
        ))}
      </ul>
    );
  }
}
```

**[Experimente no CodeSandbox](https://codesandbox.io/s/trusting-clarke-8twuq)**

Este código é enganosamente fácil de ler.

Infelizmente, nem o uso de uma classe nem a sintaxe `async` / `await` nos ajudaram a resolver esse problema. Esta versão sofre exatamente as mesmas condições de concorrência, pelas mesmas razões.

### O Problema {#the-problem}

Os componentes do React possuem seu próprio "ciclo de vida". Eles podem receber props ou atualizar o state a qualquer momento. No entanto, cada requisição assíncrona *também* possui seu próprio "ciclo de vida". Começa quando iniciamos e termina quando obtemos uma resposta. A dificuldade que estamos enfrentando é "sincronizar" vários processos no tempo que se afetam. Isso é difícil de pensar.

### Resolvendo Condições de Concorrência com Suspense {#solving-race-conditions-with-suspense}

Vamos reescrever este exemplo novamente, mas usando apenas Suspense:

```js
const initialResource = fetchProfileData(0);

function App() {
  const [resource, setResource] = useState(initialResource);
  return (
    <>
      <button onClick={() => {
        const nextUserId = getNextId(resource.userId);
        setResource(fetchProfileData(nextUserId));
      }}>
        Next
      </button>
      <ProfilePage resource={resource} />
    </>
  );
}

function ProfilePage({ resource }) {
  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails resource={resource} />
      <Suspense fallback={<h1>Loading posts...</h1>}>
        <ProfileTimeline resource={resource} />
      </Suspense>
    </Suspense>
  );
}

function ProfileDetails({ resource }) {
  const user = resource.user.read();
  return <h1>{user.name}</h1>;
}

function ProfileTimeline({ resource }) {
  const posts = resource.posts.read();
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```

**[Experimente no CodeSandbox](https://codesandbox.io/s/infallible-feather-xjtbu)**

No exemplo anterior utilizando Suspense, tínhamos apenas um `resource`, portanto mantivemos em uma variável de nível superior. Agora que temos vários recursos, o movemos para o state do componente `<App>`:

```js{4}
const initialResource = fetchProfileData(0);

function App() {
  const [resource, setResource] = useState(initialResource);
```

Quando clicamos em "Next", o componente `<App>` inicia uma requisição para o próximo perfil e passa *esse* objeto para o componente `<ProfilePage>`:

```js{4,8}
  <>
    <button onClick={() => {
      const nextUserId = getNextId(resource.userId);
      setResource(fetchProfileData(nextUserId));
    }}>
      Next
    </button>
    <ProfilePage resource={resource} />
  </>
```

Mais uma vez, observe que **não estamos aguardando a resposta para definir o state. É o contrário: definimos o state (e começamos a renderizar) imediatamente após iniciar uma requisição**. Assim que tivermos mais dados, o React "preenche" o conteúdo dentro dos componentes `<Suspense>`.

Esse código é muito legível, mas, ao contrário dos exemplos anteriores, a versão Suspense não sofre condições de concorrência. Você pode estar se perguntando o porquê. A resposta é que, na versão Suspense, não precisamos pensar tanto em *tempo* em nosso código. Nosso código original com condições de concorrência precisava definir o state *no momento certo depois* ou, caso contrário, estaria errado. Mas com o Suspense, definimos o state *imediatamente* -- por isso é mais difícil estragar tudo.

## Tratando Erros {#handling-errors}

Quando escrevemos código com Promises, podemos usar `catch()` para lidar com erros. Como isso funciona com o Suspense, já que não *esperamos* pelas Promises para começar a renderizar?

Com Suspense, o tratamento de erros de busca funciona da mesma maneira que o tratamento de erros de renderização -- você pode renderizar um [error boundary](/docs/error-boundaries.html) em qualquer lugar para "capturar" erros nos componentes abaixo.

Primeiro, definiremos um componente error boundary para usar em nosso projeto:

```js
// Error boundaries atualmente tem que ser classes.
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };
  static getDerivedStateFromError(error) {
    return {
      hasError: true,
      error
    };
  }
  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}
```

E então podemos colocá-lo em qualquer lugar da árvore para detectar erros:

```js{5,9}
function ProfilePage() {
  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails />
      <ErrorBoundary fallback={<h2>Could not fetch posts.</h2>}>
        <Suspense fallback={<h1>Loading posts...</h1>}>
          <ProfileTimeline />
        </Suspense>
      </ErrorBoundary>
    </Suspense>
  );
}
```

**[Experimente no CodeSandbox](https://codesandbox.io/s/adoring-goodall-8wbn7)**

Ele pegaria erros de renderização *e* erros da busca de dados do Suspense. Podemos ter quantos error boundaries quisermos, mas é melhor [ser intencional](https://aweary.dev/fault-tolerance-react/) sobre o posicionamento deles.

## Próximos Passos {#next-steps}

Agora, abordamos o básico do Suspense para Busca de Dados! Importante, agora entendemos melhor *por que* o Suspense funciona dessa maneira e como ele se encaixa no espaço de busca de dados.

O Suspense responde a algumas perguntas, mas também trás suas próprias novas questões:

* Se algum componente "suspende", o aplicativo congela? Como evitar isso?
* E se quisermos mostrar um spinner em um local diferente do que "acima" do componente em uma árvore?
* Se intencionalmente *queremos* mostrar uma UI inconsistente por um pequeno período, podemos fazer isso?
* Em vez de mostrar um spinner, podemos adicionar um efeito visual como "acinzentar" a tela atual?
* Por que nosso [último exemplo Suspense](https://codesandbox.io/s/infallible-feather-xjtbu) mostra um warning no console ao clicar no botão "Next"?

Para responder a essas perguntas, veremos a próxima seção sobre [Padrões de UI Concorrente](/docs/concurrent-mode-patterns.html).
