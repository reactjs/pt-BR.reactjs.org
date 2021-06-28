---
id: concurrent-mode-intro
title: Introdução ao Modo Concorrente (Experimental)
permalink: docs/concurrent-mode-intro.html
next: concurrent-mode-suspense.html
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
>Esta página descreve os **recursos experimentais que ainda [não estão disponíveis](/docs/concurrent-mode-adoption.html) em um release estável**. Não confie nas versões experimentais do React em aplicativos de produção. Esses recursos podem mudar significativamente e sem aviso antes de se tornarem parte do React.
>
>Esta documentação é destinada a adotante precoces e pessoas curiosas. **Se você é novo no React, não se preocupe com esses recursos** -- não precisa aprendê-los agora.
=======
>This page was about experimental features that aren't yet available in a stable release. It was aimed at early adopters and people who are curious.
>
>Much of the information on this page is now outdated and exists only for archival purposes. **Please refer to the [React 18 Alpha announcement post](/blog/2021/06/08/the-plan-for-react-18.html
) for the up-to-date information.**
>
>Before React 18 is released, we will replace this page with stable documentation.
>>>>>>> 25f756d8e3800afb032cb31ce3626d6134e31e38

</div>

Esta página fornece uma visão geral teórica do Modo Concorrente. **Para uma introdução mais prática, consulte as próximas seções:**

* [Suspense Para Busca de Dados](/docs/concurrent-mode-suspense.html) descreve um novo mecanismo para buscar dados nos componentes do React.
* [Padrões de UI Concorrente](/docs/concurrent-mode-patterns.html) mostra alguns padrões de UI possibilitados pelo Modo Concorrente e pelo Suspense.
* [Adotando o Modo Concorrente](/docs/concurrent-mode-adoption.html) explica como você pode experimentar o Modo Concorrente em seu projeto.
* [Referência da API do Modo Concorrente](/docs/concurrent-mode-reference.html) documenta as novas APIs disponíveis em builds experimentais.

## O Que é o Modo Concorrente? {#what-is-concurrent-mode}

O Modo Concorrente é um conjunto de novos recursos que ajudam os aplicativos React a permanecerem responsivos e a se ajustarem normalmente aos recursos do dispositivo do usuário e à velocidade da rede.

Esses recursos ainda são experimentais e estão sujeitos a alterações. Eles ainda não fazem parte de um release estável do React, mas você pode experimentá-los em uma versão experimental.

## Bloqueio vs Renderização Interrompida {#blocking-vs-interruptible-rendering}

**Para explicar o Modo Concorrente, usaremos o controle de versão como uma metáfora.** Se você trabalha em equipe, provavelmente usa um sistema de controle de versão como o Git e trabalha em _branches_. Quando uma _branch_ estiver pronta, você poderá dar merge do seu trabalho na master(_branch_ principal), para que outras pessoas possam puxa-las (_pull request_).

Antes da existência do controle de versão, o fluxo de trabalho de desenvolvimento era muito diferente. Não havia conceito de _branches_. Se você quiser editar alguns arquivos, precisará dizer a todos para não tocarem nesses arquivos até que você termine seu trabalho. Você não podia nem começar a trabalhar neles simultaneamente com essa pessoa - você estava literalmente *bloqueado* por ela.

Isso ilustra como as bibliotecas de UI, incluindo React, normalmente funcionam hoje. Depois que eles começam a renderizar uma atualização, incluindo a criação de novos nós do DOM e a execução do código nos componentes, eles não podem interromper esse trabalho. Vamos chamar essa abordagem de "bloqueio de renderização".

Em Modo Concorrente, a renderização e não bloqueante. É interruptível. Isso melhora a experiência do usuário. Também desbloqueia novos recursos que antes não eram possíveis. Antes de examinarmos exemplos concretos nos [próximos](/docs/concurrent-mode-suspense.html) [capítulos](/docs/concurrent-mode-patterns.html), faremos uma visão geral de alto nível dos novos recursos.

### Renderização interrompida {#interruptible-rendering}

Considere uma lista de produtos filtrável. Você já digitou um filtro de lista e sentiu que ele gagueja a cada pressionamento de tecla? Alguns dos trabalhos para atualizar a lista de produtos podem ser inevitáveis, como criar novos nós no DOM ou o navegador executando o layout. No entanto, *quando* e *como* realizamos esse trabalho desempenha um grande papel.

Uma maneira comum de contornar a gagueira é "rejeitar" a entrada. Ao rebater, atualizamos apenas a lista *após* o usuário parar de digitar. No entanto, pode ser frustrante que a interface do usuário não seja atualizada enquanto estamos digitando. Como alternativa, poderíamos "acelerar" a entrada e atualizar a lista com uma certa frequência máxima. Mas então, em dispositivos de baixa potência, ainda acabaríamos com travadas. A depuração e a otimização criam uma experiência de usuário abaixo do ideal.

A razão para os travamentos são simples: uma vez iniciada a renderização, ela não pode ser interrompida. Portanto, o navegador não pode atualizar a entrada de texto logo após pressionar a tecla. Independentemente da aparência de uma biblioteca de interface do usuário (como React), se ela usa a renderização de bloqueante, uma certa quantidade de trabalho em seus componentes sempre causarão travamentos. E, muitas vezes, não há solução fácil.

**O Modo Concorrente corrige essa limitação fundamental, tornando a renderização interrompível.** Isso significa que quando o usuário pressiona outra tecla, o React não precisa impedir o navegador de atualizar a entrada de texto. Em vez disso, ele pode deixar o navegador pintar uma atualização na entrada e continuar renderizando a lista atualizada *na memória*. Quando a renderização é concluída, o React atualiza o DOM e as alterações são refletidas na tela.

Conceitualmente, você pode pensar nisso como React preparando todas as atualizações "em uma _branch_". Assim como você pode abandonar o trabalho em _branches_ ou alternar entre elas, o React no modo concorrente pode interromper uma atualização contínua para fazer algo mais importante e depois voltar ao que estava fazendo anteriormente. Essa técnica também pode lembrá-lo do [buffer duplo](https://wiki.osdev.org/Double_Buffering) nos videogames.

As técnicas do modo concorrente reduzem a necessidade de renúncia e limitação na interface do usuário. Como a renderização é interrompível, o React não precisa *atrasar* artificialmente o trabalho para evitar interrupções. Ele pode começar a renderizar imediatamente, mas interrompa esse trabalho quando necessário para manter o aplicativo responsivo.

### Sequências de Carregamento Intencionais {#intentional-loading-sequences}

Dissemos antes que o Modo Concorrente é como React trabalhando "em uma _branch_". _Branches_ são úteis não apenas para correções de curto prazo, mas também para recursos de longa duração. Às vezes, você pode trabalhar em um recurso, mas pode levar semanas até que ele esteja em um "estado suficientemente bom" para fazer o _merge_ a master. Esse lado da nossa metáfora de controle de versão também se aplica à renderização.

Imagine que estamos navegando entre duas telas em um aplicativo. Às vezes, talvez não tenhamos código e dados suficientes carregados para mostrar um estado de carregamento "suficientemente bom" para o usuário na nova tela. A transição para uma tela vazia ou um grande *spinner* em toda a tela pode ser uma experiência chocante. No entanto, também é comum que o código e os dados necessários não demorem muito para serem buscados. **Não seria melhor se o React pudesse permanecer na tela antiga por mais algum tempo e "pular" o "estado de carregamento incorreto" antes de mostrar a nova tela?**

Embora isso seja possível hoje, pode ser difícil orquestrar. No Modo Concorrente, esse recurso é interno. O React começa a preparar a nova tela em memória primeiro - ou, como nossa metáfora diz, "em uma _branch_ diferente". Portanto, o React pode esperar antes de atualizar o DOM para que mais conteúdo possa ser carregado. No Modo Concorrente, podemos dizer ao React para continuar mostrando a tela antiga, totalmente interativa, com um indicador de carregamento embutido. E quando a nova tela estiver pronta, o React pode nos levar a ela.

### Concorrência {#concurrency}

Vamos recapitular os dois exemplos acima e ver como o Modo Concorrente os unifica. **No Modo Concorrente, o React pode trabalhar em várias atualizações de estado *concorrente*** - assim como as _branches_, diferentes membros da equipe trabalham independentemente:

* Para atualizações associadas à CPU (como criar nós DOM e executar código de componente), a concorrência significa que uma atualização mais urgente pode "interromper" a renderização que já foi iniciada.
* Para atualizações associadas à IO (como a busca de código ou dados da rede), a concorrência significa que o React pode iniciar a renderização na memória antes mesmo da chegada de todos os dados, e pule a exibição de estados de carregamento vazios discordantes.

Importante, a maneira como você *usa* o React é a mesma. Conceitos como componentes, _props_, e _state_ funcionam fundamentalmente da mesma maneira. Quando você deseja atualizar a tela, você define o estado.

O React usa uma heurística para decidir o quão "urgente" é uma atualização e permite que você possa obter a experiência do usuário desejada para cada interação.

## Colocando Pesquisa em Produção {#putting-research-into-production}

Há um tema comum nos recursos do Modo Concorrente. **Sua missão é ajudar a integrar as descobertas da pesquisa sobre Interação Homem-Computador em interfaces de usuários reais.**

Por exemplo, pesquisas mostram que exibir muitos estados de carregamento intermediários ao fazer a transição entre telas faz com que uma transição pareça mais *lenta*. É por isso que o Modo Concorrente mostra novos estados de carregamento em um "agendamento" fixo para evitar atualizações dissonantes e muito frequentes.

Da mesma forma, sabemos da pesquisa que interações como passar o mouse e inserir texto precisam ser tratadas dentro de um período muito curto de tempo, enquanto cliques e transições de página podem esperar um pouco mais sem ficarem parados. As diferentes "prioridades" que o Modo Concorrente usa internamente correspondem aproximadamente às categorias de interação na pesquisa de percepção humana.

Às vezes, equipes com forte foco na experiência do usuário resolvem problemas semelhantes com soluções pontuais. No entanto, essas soluções raramente sobrevivem por um longo tempo, pois são difíceis de manter. Com o Modo Concorrente, nosso objetivo é incorporar as descobertas da pesquisa de interface do usuário na própria abstração e fornecer maneiras idiomáticas de usá-las. Como uma biblioteca de interface do usuário, o React está bem posicionado para fazer isso.

## Próximos Passos {#next-steps}

Agora você sabe o que é o Modo Concorrente!

Nas próximas páginas, você aprenderá mais detalhes sobre tópicos específicos:

* [Suspense Para Busca de Dados](/docs/concurrent-mode-suspense.html) descreve um novo mecanismo para buscar dados nos componentes do React.
* [Padrões de UI Concorrente](/docs/concurrent-mode-patterns.html) mostra alguns padrões de UI possibilitados pelo Modo Concorrente e pelo Suspense.
* [Adotando o Modo Concorrente](/docs/concurrent-mode-adoption.html) explica como você pode experimentar o Modo Concorrente em seu projeto.
* [Referência da API do Modo Concorrente](/docs/concurrent-mode-reference.html) documenta as novas APIs disponíveis em builds experimentais.
