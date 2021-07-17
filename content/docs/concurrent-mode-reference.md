---
id: concurrent-mode-reference
title: Referência da API do Modo Concorrente (Experimental)
permalink: docs/concurrent-mode-reference.html
prev: concurrent-mode-adoption.html
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
>Esta página tratava de recursos experimentais que ainda não estão disponíveis em uma versão estável. Ele era voltado para os primeiros usuários e pessoas curiosas.
>
>Muitas das informações nesta página estão desatualizadas e existem apenas para fins de arquivamento. **Consulte a [postagem de anúncio do React 18 Alpha](/blog/2021/06/08/the-plan-for-react-18.html
) para as informações atualizadas.**
>
>Antes do React 18 ser lançado, substituiremos esta página por uma documentação estável.

</div>

Esta página é uma referência de API para o [Modo Concurrent](/docs/concurrent-mode-intro.html) do React. Se você está procurando uma introdução guiada, confira [Padrões de UI Concorrente](/docs/concurrent-mode-patterns.html).

**Nota: Esta é uma Prévia da Comunidade e não a versão estável final. Provavelmente haverá mudanças futuras nessas APIs. Use por sua conta e risco!**

- [Ativando o Modo Concorrente](#concurrent-mode)
    - [`createRoot`](#createroot)
- [Suspense](#suspense)
    - [`Suspense`](#suspensecomponent)
    - [`SuspenseList`](#suspenselist)
    - [`useTransition`](#usetransition)
    - [`useDeferredValue`](#usedeferredvalue)

## Ativando o Modo Concorrente {#concurrent-mode}

### `createRoot` {#createroot}

```js
ReactDOM.createRoot(rootNode).render(<App />);
```

Substitui o `ReactDOM.render(<App />, rootNode)` e ativa o Modo Concorrente.

Para mais informações sobre o Modo Concorrente, consulte a [documentação do Modo Concorrente.](/docs/concurrent-mode-intro.html)

## Suspense API {#suspense}

### `Suspense` {#suspensecomponent}

```js
<Suspense fallback={<h1>Carregando...</h1>}>
  <ProfilePhoto />
  <ProfileDetails />
</Suspense>
```

`Suspense` permite que seus componentes "esperem" por algo antes que eles possam renderizar, mostrando um fallback enquanto aguardam.

Neste exemplo, `ProfileDetails` está aguardando uma chamada de API assíncrona para buscar alguns dados. Enquanto aguardamos o `ProfileDetails` e o `ProfilePhoto`, mostraremos o `Carregando...` como fallback. É importante observar que até que todos os filhos dentro de `<Suspense>` sejam carregados, continuaremos a mostrar o fallback.

`Suspense` recebe duas props:
* **fallback** recebe um indicador de carregamento. O fallback é mostrado até que todos os filhos do componente `Suspense` tenham concluído a renderização.
* **unstable_avoidThisFallback** recebe um boolean. Isso diz ao React se deve "pular" revelando esse limite durante o carregamento inicial. Essa API provavelmente será removida em uma versão futura.

### `<SuspenseList>` {#suspenselist}

```js
<SuspenseList revealOrder="forwards">
  <Suspense fallback={'Carregando...'}>
    <ProfilePicture id={1} />
  </Suspense>
  <Suspense fallback={'Carregando...'}>
    <ProfilePicture id={2} />
  </Suspense>
  <Suspense fallback={'Carregando...'}>
    <ProfilePicture id={3} />
  </Suspense>
  ...
</SuspenseList>
```

`SuspenseList` ajuda a coordenar muitos componentes que podem ser suspensos, orquestrando a ordem em que esses componentes são revelados ao usuário.

Quando vários componentes precisam buscar dados, esses dados podem chegar em uma ordem imprevisível. No entanto, se você agrupar esses itens em um `SuspenseList`, React não mostrará um item na lista até que os itens anteriores sejam exibidos (esse comportamento é ajustável).

`SuspenseList` recebe duas props:
* **revealOrder (forwards, backwards, together)** define a ordem em que os filhos de `SuspenseList` devem ser reveladas.
  * `together` revela *todos* eles quando estiverem prontos em vez de um por um.
* **tail (collapsed, hidden)** determina como os itens não-carregados em um `SuspenseList` são mostrados. 
    * Por padrão, `SuspenseList` mostrará todos os fallbacks na lista.
    * `collapsed` mostra apenas o próximo fallback na lista.
    * `hidden` não mostra nenhum item não-carregado.

Observe que `SuspenseList` funciona apenas nos componentes `Suspense` e `SuspenseList` mais próximos abaixo dele. Ele não procura por componentes mais profundos que um nível. No entanto, é possível aninhar múltiplos componentes `SuspenseList` um no outro para construir grades.

### `useTransition` {#usetransition}

```js
const SUSPENSE_CONFIG = { timeoutMs: 2000 };

const [startTransition, isPending] = useTransition(SUSPENSE_CONFIG);
```

`useTransition` permite que os componentes evitem estados de carregamento indesejáveis, aguardando o carregamento do conteúdo antes da **transição para a próxima tela**. Ele também permite que os componentes adiem atualizações de busca de dados mais lentas até as renderizações subsequentes, para que atualizações mais cruciais possam ser renderizadas imediatamente.

O hook `useTransition` retorna dois valores em um array.
* `startTransition` é uma função que recebe um callback. Podemos usá-lo para dizer ao React qual estado queremos adiar.
* `isPending` é um boolean. É a maneira do React de nos informar se estamos esperando a transição terminar.

**Se alguma atualização de estado fizer com que um componente seja suspenso, essa atualização de estado deverá ser agrupada em uma transição.**

```js
const SUSPENSE_CONFIG = { timeoutMs: 2000 };

function App() {
  const [resource, setResource] = useState(initialResource);
  const [startTransition, isPending] = useTransition(SUSPENSE_CONFIG);
  return (
    <>
      <button
        disabled={isPending}
        onClick={() => {
          startTransition(() => {
            const nextUserId = getNextId(resource.userId);
            setResource(fetchProfileData(nextUserId));
          });
        }}
      >
        Próximo
      </button>
      {isPending ? " Carregando..." : null}
      <Suspense fallback={<Spinner />}>
        <ProfilePage resource={resource} />
      </Suspense>
    </>
  );
}
```

Nesse código, agrupamos nossa busca de dados com `startTransition`. Isso nos permite começar a buscar os dados do perfil imediatamente, enquanto adia a renderização da próxima página de perfil e seu `Spinner` por 2 segundos (o tempo apresentado em `timeoutMs`).

O boolean `isPending` informa ao React que nosso componente está em transição, para que possamos informar ao usuário mostrando algum texto de carregando na página de perfil anterior.

**Para uma análise detalhada das transições, você pode ler [Padrões de UI Concorrente](/docs/concurrent-mode-patterns.html#transitions).**

#### Configuração do useTransition {#usetransition-config}

```js
const SUSPENSE_CONFIG = { timeoutMs: 2000 };
```

`useTransition` aceita uma **configuração opcional do Suspense** com um `timeoutMs`. Esse timeout (em milissegundos) informa ao React quanto tempo esperar antes de mostrar o próximo estado (a nova página de perfil no exemplo acima).

**Nota: Recomendamos que você compartilhe a Configuração do Suspense entre diferentes módulos.**


### `useDeferredValue` {#usedeferredvalue}

```js
const deferredValue = useDeferredValue(value, { timeoutMs: 2000 });
```

Retorna uma versão adiada (defer) do valor que pode "atrasar" por no máximo `timeoutMs`.

Isso é comumente usado para manter a interface responsiva quando você tem algo que é renderizado imediatamente com base na entrada do usuário e algo que precisa aguardar uma busca de dados.

Um bom exemplo disso é um input de texto.

```js
function App() {
  const [text, setText] = useState("Olá");
  const deferredText = useDeferredValue(text, { timeoutMs: 2000 }); 

  return (
    <div className="App">
      {/* Continua passando o atual texto para o input */}
      <input value={text} onChange={handleChange} />
      ...
      {/* Mas a lista pode "atrasar" quando necessário */}
      <MySlowList text={deferredText} />
    </div>
  );
 }
```

Isso nos permite começar a mostrar o novo texto para o "input" imediatamente, o que permite que a página se sinta responsiva. Enquanto isso, `MySlowList` fica "atrasado" por até 2 segundos, de acordo com o `timeoutMs` antes da atualizar, permitindo que seja renderizado com o texto atual em segundo plano.

**Para uma análise detalhada do adiamento de valores, você pode ler [Padrões de UI Concorrente](/docs/concurrent-mode-patterns.html#deferring-a-value).**

#### Configuração do useDeferredValue {#usedeferredvalue-config}

```js
const SUSPENSE_CONFIG = { timeoutMs: 2000 };
```

`useDeferredValue` aceita uma **configuração opcional do Suspense** com um `timeoutMs`. Esse tempo limite (em milissegundos) informa ao React quanto tempo o valor adiado pode atrasar.

O React sempre tentará usar um atraso menor quando a rede e o dispositivo permitirem.
