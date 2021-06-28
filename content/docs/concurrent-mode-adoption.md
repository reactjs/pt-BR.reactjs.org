---
id: concurrent-mode-adoption
title: Adotando o Modo Concorrente (Experimental)
permalink: docs/concurrent-mode-adoption.html
prev: concurrent-mode-patterns.html
next: concurrent-mode-reference.html
---

<style>
.scary > blockquote {
  background-color: rgba(237, 51, 21, 0.2);
  border-left-color: #ed3315;
}
</style>

<div class="scary">

>Atenção:
>
<<<<<<< HEAD
>Esta página descreve **recursos experimentais que ainda não estão disponíveis numa versão estável**. Não confie nas versões experimentais do React em aplicativos de produção. Esses recursos podem mudar significativamente e sem aviso antes de se tornarem parte do React.
>
>Esta documentação é destinada a adotantes precoces e pessoas curiosas. **Se você é iniciante no React, não se preocupe com esses recursos** -- não precisa aprendê-los agora.

>Cuidado:
>
>Todas as referências abaixo para "modo de bloqueio" e `createBlockingRoot` estão desatualizadas e devem ser ignoradas.
=======
>This page was about experimental features that aren't yet available in a stable release. It was aimed at early adopters and people who are curious.
>
>Much of the information on this page is now outdated and exists only for archival purposes. **Please refer to the [React 18 Alpha announcement post](/blog/2021/06/08/the-plan-for-react-18.html
) for the up-to-date information.**
>
>Before React 18 is released, we will replace this page with stable documentation.
>>>>>>> 25f756d8e3800afb032cb31ce3626d6134e31e38

</div>

- [Instalação](#installation)
  - [A Quem os Recursos Experimentais se Destinam?](#who-is-this-experimental-release-for)
  - [Habilitando o Modo Concorrente](#enabling-concurrent-mode)
- [O Que Esperar](#what-to-expect)
  - [Etapa de Migração: Modo Bloqueante](#migration-step-blocking-mode)
  - [Por Que Tantos Modos?](#why-so-many-modes)
  - [Comparativo de Recursos](#feature-comparison)

## Instalação {#installation}

O Modo Concorrente está disponível apenas em [versões experimentais](/blog/2019/10/22/react-release-channels.html#experimental-channel) do React. Para instalá-las, execute:

```
npm install react@experimental react-dom@experimental
```

**Não existem garantias de versionamento semântico para versões experimentais.**  
APIs podem ser adicionadas, alteradas ou removidas em qualquer versão `@experimental`.

**Versões experimentais constantemente terão alterações com problemas.**

Você pode testar essas versões nos seus projetos pessoais ou em uma branch, mas nós não recomendamos utilizar elas em produção. No Facebook, nós utilizamos elas em produção, mas isso é porque nós estamos prontos para corrigir os problemas assim que ocorrem. Você deve estar atento a isso.

### A Quem os Recursos Experimentais se Destinam? {#who-is-this-experimental-release-for}

Esta versão é destinada a adotantes precoces, autores de livros e pessoas curiosas.

Nós estamos usando esse código em produção (e funciona para nós), mas ainda possui alguns problemas, recursos faltando e falta de documentação. Nós gostaríamos de saber mais caso algo quebre no Modo Concorrente assim nós podemos ajustar na versão estável que será oficialmente disponibilizada futuramente.

### Habilitando o Modo Concorrente {#enabling-concurrent-mode}

Normalmente, quando nós adicionamos recursos ao React, você pode começar a usá-lo imediatamente. Fragments, Context e até mesmo Hooks são exemplos desses recursos. Você pode usá-los no novo código sem fazer alterações no código existente.

O Modo Concorrente é diferente. Ele introduz alterações semânticas na forma do React funcionar. Do contrário, os [novos recursos](/docs/concurrent-mode-patterns.html) habilitados por ele *não teriam se tornado possíveis*. Este é o motivo de eles estarem agrupadas em um novo "modo" ao invés de serem liberados um a um isoladamente.

Você não pode utilizar o Modo Concorrente apenas em uma sub árvore do seu aplicativo. Para utilizá-lo, você deve incluir no ponto onde atualmente você chama `ReactDOM.render()`.

**Isso irá habilitar o modo concorrente para toda a árvore `<App />`:**

```js
import ReactDOM from 'react-dom';

// Se anteriormente você tinha:
//
// ReactDOM.render(<App />, document.getElementById('root'));
//
// Você pode incluir o Modo Concorrente escrevendo:

ReactDOM.unstable_createRoot(
  document.getElementById('root')
).render(<App />);
```

>Nota:
>
>As APIs do Modo Concorrente como `createRoot`apenas existem em versões experimentais do React.

No Modo Concorrente, os métodos do ciclo de vida [anteriormente marcados](/blog/2018/03/27/update-on-async-rendering.html) como "inseguros" são *realmente* inseguros, e podem gerar ainda mais problemas do que na versão atual do React. Nós não recomendamos testar o Modo Concorrente até que sua aplicação seja compatível com o [Strict Mode](/docs/strict-mode.html).

## O Que Esperar {#what-to-expect}

Se você tem uma aplicação existente muito grande, ou se a sua aplicação depende de muitas bibliotecas de terceiros, por favor não comece a utilizar o Modo Concorrente imediatamente. **Por exemplo, no Facebook nós estamos usando o Modo Concorrente para o nosso novo website, mas não planejamos habilitá-lo na versão antiga.** Isto porque o nosso website antigo ainda utiliza métodos de ciclo de vida inseguros em código de produção, e alguns padrões que não funcionam bem com o Modo Concorrente.

De acordo com nossa experiência, utilizar os padrões idiomáticos do React no código e não utilizar soluções externas para gerenciamento de estado são a maneira mais fácil de iniciar a utilização do Modo Concorrente. Nós vamos descrever problemas comuns que nós experimentamos e as soluções para cada um deles nas próximas semanas.

### Etapa de Migração: Modo Bloqueante {#migration-step-blocking-mode}

Para códigos antigos, o Modo Concorrente pode ser um passo muito grande. Este é o motivo de nós também termos disponibilizado o novo "Modo Bloqueante" nas versões experimentais do React. Você pode testá-lo substituindo `createRoot` por `createBlockingRoot`. Isso apenas confere uma *pequena parte* dos recursos presentes no Modo Concorrente, mas é mais próximo da forma como o React funciona hoje e pode servir como um passo na migração.

Para recaptular:

* **Modo Legado:** `ReactDOM.render(<App />, rootNode)`. Está é a forma que os aplicativos React usam hoje. Não existem planos para remover o Modo Legado num futuro próximo — mas não será possível dar suporte para esses novos recursos.
* **Modo Bloqueante:** `ReactDOM.createBlockingRoot(rootNode).render(<App />)`. Atualmente é experimental. A intenção é que seja um primeiro passo na migração para aplicativos que desejam utilizar uma parte dos recursos do Modo Concorrente.
* **Modo Concorrente:** `ReactDOM.createRoot(rootNode).render(<App />)`. Atualmente é experimental. No futuro, após se tornar estável, nos planejamos torná-lo o modo padrão do React. Esse modo habilita *todos* os novos recursos.

### Por Que Tantos Modos? {#why-so-many-modes}

Nós acreditamos que é melhor oferecer uma [estratégia de migração gradativa](/docs/faq-versioning.html#commitment-to-stability) do que fazer grandes mudanças — ou então deixar o React estagnado na irrelevância.

Na prática, nós esperamos que a maioria dos aplicativos que hoje estão usando o Modo Legado possam migrar pelo menos para o Modo Bloqueante (se não para o Modo Concorrente). Esta fragmentação pode ser incômoda para bibliotecas que almejam dar suporte para todos os modos em curto prazo. No entanto, gradativamente migrando o ecossistema irá *resolver* problemas que afetam a maioria das bibliotecas no ecossistema do React, bem como  [comportamentos confusos do Suspense quando lê o layout](https://github.com/facebook/react/issues/14536) e [a falta de garantias consistentes de gerenciamento em lote](https://github.com/facebook/react/issues/15080). Existe um número grande de problemas que não podem ser resolvidos no Modo Legado sem mudanças na semântica, e que não ocorrem nos modos Bloqueante e Concorrente.

Você pode pensar no Modo Bloqueante como uma versão suavemente degradada do Modo Concorrente. **Como resultado, a longo prazo nós poderemos convergir e parar de pensar em modos diferentes.** Mas não de imediato, Modos são uma importante estratégia de migração. Eles permitem que todos decidam quando a migração vale a pena, e atualizar em seu próprio ritmo.

### Comparativo de Recursos {#feature-comparison}

<style>
  #feature-table table { border-collapse: collapse; }
  #feature-table th { padding-right: 30px; }
  #feature-table tr { border-bottom: 1px solid #eee; }
</style>

<div id="feature-table">

|   |Modo Legado  |Modo Bloqueante  |Modo Concorrente  |
|---  |---  |---  |---  |
|[String Refs](/docs/refs-and-the-dom.html#legacy-api-string-refs)  |✅  |🚫**  |🚫**  |
|[Legacy Context](/docs/legacy-context.html) |✅  |🚫**  |🚫**  |
|[findDOMNode](/docs/strict-mode.html#warning-about-deprecated-finddomnode-usage)  |✅  |🚫**  |🚫**  |
|[Suspense](/docs/concurrent-mode-suspense.html#what-is-suspense-exactly) |✅  |✅  |✅  |
|[SuspenseList](/docs/concurrent-mode-patterns.html#suspenselist) |🚫  |✅  |✅  |
|Suspense SSR + Hydration |🚫  |✅  |✅  |
|Progressive Hydration  |🚫  |✅  |✅  |
|Selective Hydration  |🚫  |🚫  |✅  |
|Cooperative Multitasking |🚫  |🚫  |✅  |
|Automatic batching of multiple setStates     |🚫* |✅  |✅  |
|[Priority-based Rendering](/docs/concurrent-mode-patterns.html#splitting-high-and-low-priority-state) |🚫  |🚫  |✅  |
|[Interruptible Prerendering](/docs/concurrent-mode-intro.html#interruptible-rendering) |🚫  |🚫  |✅  |
|[useTransition](/docs/concurrent-mode-patterns.html#transitions)  |🚫  |🚫  |✅  |
|[useDeferredValue](/docs/concurrent-mode-patterns.html#deferring-a-value) |🚫  |🚫  |✅  |
|[Suspense Reveal "Train"](/docs/concurrent-mode-patterns.html#suspense-reveal-train)  |🚫  |🚫  |✅  |

</div>

\*: O modo Legado possui eventos gerenciados pelo React em lote mas é limitado a uma tarefa do navegador. Eventos que não são do React podem utilizá-lo através do `unstable_batchedUpdates`. No Modo Bloqueante e Concorrente, todos os `setState`s são feitos em lote por padrão.

\*\*: Avisa em desenvolvimento.
