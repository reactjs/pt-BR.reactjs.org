---
id: release-channels
title: Canais de Release
permalink: docs/release-channels.html
layout: docs
category: installation
prev: cdn-links.html
next: hello-world.html
---

React conta com uma próspera comunidade de código aberto para registrar relatórios de bugs, abrir pull requests e [enviar RFCs](https://github.com/reactjs/rfcs). Para incentivar feedback, às vezes compartilhamos builds especiais do React que incluem recursos não lançados.

> Este documento será mais relevante para desenvolvedores que trabalham em frameworks, bibliotecas, ou ferramentas de desenvolvedor. Os desenvolvedores que usam React principalmente para criar aplicações voltadas para o usuário final não precisam se preocupar com nossos canais de prerelease.

Cada um dos canais de release do React foram projetados para um caso de uso distinto:

- [**Latest**](#latest-channel) é para estável, releases semver do React. É o que você recebe ao instalar o React a partir do npm. Este é o canal que você já está usando hoje. **Use isso para todos as aplicações React voltadas para o usuário final.**
- [**Next**](#next-channel) acompanha a branch master do repositório de código fonte do React. Pense nisso como candidatos a release para o próximo minor semver release. Use isso para teste de integração entre React e projetos de terceiros.
- [**Experimental**](#experimental-channel) inclui APIs experimentais e recursos que não estão disponíveis nas releases estáveis. Eles também acompanham a branch master, mas com flags de recursos adicionais ativadas. Use isso para experimentar os próximos recursos antes de serem lançados.

<<<<<<< HEAD
Todos os releases são publicados no npm, mas apenas os Latest usam [versionamento semântico](/docs/faq-versioning.html). Os prereleases (aqueles nos canais Next e Experimental) têm versões geradas a partir de um hash de seu conteúdo, por exemplo `0.0.0-1022ee0ec` para Next e `0.0.0-experimental-1022ee0ec` para Experimental.
=======
All releases are published to npm, but only Latest uses [semantic versioning](/docs/faq-versioning.html). Prereleases (those in the Next and Experimental channels) have versions generated from a hash of their contents and the commit date, e.g. `0.0.0-68053d940-20210623` for Next and `0.0.0-experimental-68053d940-20210623` for Experimental.
>>>>>>> 25f756d8e3800afb032cb31ce3626d6134e31e38

**O único canal de release oficialmente suportado para aplicações voltadas para o usuário final é o Latest**. Next e Experimental releases são fornecidos apenas para fins de teste e não fornecemos garantias de que o comportamento não seja alterado entre as versões. Eles não seguem o protocolo semver que usamos para releases no Latest.

Por publicar prereleases no mesmo registro que usamos para releases estáveis, podemos tirar proveito das muitas ferramentas que suportam o fluxo de trabalho no npm, como [unpkg](https://unpkg.com) e [CodeSandbox](https://codesandbox.io).

### Canal Latest {#latest-channel}

Latest é o canal usado para releases estáveis do React. Corresponde à tag `latest` no npm. É o canal recomendado para todos os apps React que são enviados para usuários reais.

**Se você não tem certeza de qual canal deve usar, é o Latest.** Se você é um desenvolvedor React, é isso que você já está usando.

Você pode esperar que as atualizações do Latest sejam extremamente estáveis. As versões seguem o esquema de versão semântica. Saiba mais sobre nosso compromisso com a estabilidade e a migração incremental em nossa [política de versão](/docs/faq-versioning.html).

### Canal Next {#next-channel}

O canal Next é um canal de prerelease que acompanha a branch master do repositório React. Usamos as prereleases no canal Next como release candidates para o canal Latest. Você pode pensar em Next como um superconjunto de Latest que é atualizado com mais frequência.

O grau de alteração entre o release mais recente do Next e a versão mais recente do Latest é aproximadamente o mesmo que você encontraria entre dois releases de minor semver. No entanto, **o canal Next não está de acordo com o controle de versão semântico.** Você deve esperar breaking changes ocasionais durante sucessivos releases no canal Next.

**Não use prereleases em aplicações voltadas para o usuário final.**

<<<<<<< HEAD
Releases no Next são publicadas com a tag `next` no npm. As versões são geradas a partir de um hash do conteúdo do build, por exemplo `0.0.0-1022ee0ec`.
=======
Releases in Next are published with the `next` tag on npm. Versions are generated from a hash of the build's contents and the commit date, e.g. `0.0.0-68053d940-20210623`.
>>>>>>> 25f756d8e3800afb032cb31ce3626d6134e31e38

#### Usando o Canal Next para Testes de Integração {#using-the-next-channel-for-integration-testing}

O canal Next foi projetado para dar suporte aos testes de integração entre React e outros projetos.

Todas as alterações no React passam por extensos testes internos antes de serem lançadas para o público. No entanto, há uma infinidade de ambientes e configurações usados em todo o ecossistema React, e não é possível testar cada um deles.

Se você é o autor de um projeto de terceiros como um framework, biblioteca, ferramenta de desenvolvedor, ou qualquer tipo de infraestrutura semelhante, você pode nos ajudar a manter o React estável para seus usuários e para toda a comunidade React executando periodicamente sua suíte de testes nas alterações mais recentes. Se você estiver interessado, siga estas etapas:

- Configure um cron job usando sua plataforma de integração contínua preferida. Cron jobs são suportados ambos pelo [CircleCI](https://circleci.com/docs/2.0/triggers/#scheduled-builds) e pelo [Travis CI](https://docs.travis-ci.com/user/cron-jobs/).
- No cron job, atualize seus pacotes React para o release mais recente do React no canal Next, usando a tag `next` no npm. Usando o npm cli:

  ```
  npm update react@next react-dom@next
  ```

  Ou yarn:

  ```
  yarn upgrade react@next react-dom@next
  ```
- Execute sua suíte de testes nos pacotes atualizados.
- Se tudo passar, ótimo! Você pode esperar que seu projeto funcione com o próximo minor release do React.
- Se algo quebrar inesperadamente, por favor avise-nos [criando uma issue](https://github.com/facebook/react/issues).

Um projeto que usa esse fluxo de trabalho é Next.js. (Sem trocadilhos! Sério!) Você pode consultar a sua [configuração do CircleCI](https://github.com/zeit/next.js/blob/c0a1c0f93966fe33edd93fb53e5fafb0dcd80a9e/.circleci/config.yml) como exemplo.

### Canal Experimental {#experimental-channel}

Como o Next, o canal Experimental é um canal de prerelease que acompanha a branch master do repositório React. Ao contrário do Next, os releases Experimental incluem recursos e APIs adicionais que não estão prontas para release maior.

Geralmente, uma atualização para Next é acompanhada por uma atualização correspondente para Experimental. Eles são baseados na mesma revisão de origem, mas são criados usando um conjunto diferente de flags de recursos.

Os releases Experimental podem ser significativamente diferentes dos releases Next e Latest. **Não use releases Experimental em aplicações voltadas para o usuário final.** Você deve esperar breaking changes frequentes entre releases no canal Experimental.

<<<<<<< HEAD
Releases no Experimental são publicados com a tag `experimental` no npm. As versões são geradas a partir de um hash do conteúdo do build, por exemplo `0.0.0-experimental-1022ee0ec`.
=======
Releases in Experimental are published with the `experimental` tag on npm. Versions are generated from a hash of the build's contents and the commit date, e.g. `0.0.0-experimental-68053d940-20210623`.
>>>>>>> 25f756d8e3800afb032cb31ce3626d6134e31e38

#### O Que Entra em um Release Experimental? {#what-goes-into-an-experimental-release}

Recursos experimentais são aqueles que não estão prontos para serem liberados ao público em geral e podem sofrer alterações drásticas antes de serem finalizados. Alguns experimentos podem nunca ser finalizados - a razão pela qual temos experimentos é testar a viabilidade das alterações propostas.

Por exemplo, se o canal Experimental existisse quando anunciamos Hooks, teríamos lançado Hooks no canal Experimental semanas antes de estarem disponíveis no Latest.

Você pode achar valioso executar testes de integração no Experimental. Isso é com você. No entanto, saiba que o Experimental é ainda menos estável que o Next. **Não garantimos estabilidade entre releases Experimental.**

#### Como Posso Aprender Mais Sobre os Recursos Experimentais? {#how-can-i-learn-more-about-experimental-features}

Recursos Experimentais podem ou não ser documentados. Geralmente, os experimentos não são documentados até que estejam perto de serem lançados no Next ou no Latest.

Se um recurso não estiver documentado, ele poderá ser acompanhado por um [RFC](https://github.com/reactjs/rfcs).

Publicaremos no [blog React](/blog) quando estivermos prontos para anunciar novos experimentos, mas isso não significa que divulgaremos cada experimento.

Você sempre pode consultar o [histórico](https://github.com/facebook/react/commits/master) do repositório público do GitHub para obter uma lista abrangente de alterações.
