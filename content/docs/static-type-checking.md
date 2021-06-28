---
id: static-type-checking
title: Verificando Tipos Estáticos
permalink: docs/static-type-checking.html
---

Verificadores de tipos estáticos, como [Flow](https://flow.org/) e [TypeScript](https://www.typescriptlang.org/), identificam certos tipos de problemas mesmo antes do seu código ser executado. Eles também melhoram o fluxo de trabalho do desenvolvedor adicionando features como preenchimento automático. Por isso, recomendamos usar Flow ou TypeScript ao invés de `PropTypes` para bases de código maiores.

## Flow {#flow}

[Flow](https://flow.org/) é um verificador de tipos estáticos para o seu código JavaScript. É desenvolvido no Facebook e frequentemente usado com o React. Ele permite que você faça anotações às variáveis, funções e componentes do React com um tipo especial de sintaxe e capture erros cedo. Você pode ler a [introdução ao Flow](https://flow.org/en/docs/getting-started/) para aprender o básico.

Para usar o Flow, você precisa:

* Adicionar o Flow como dependência ao seu projeto.
* Garantir que a sintaxe do Flow seja removida do código compilado.
* Adicionar anotações de tipo e executar o Flow para checá-las.

Explicaremos abaixo esses passos com detalhes.

### Adicionando Flow a um Projeto {#adding-flow-to-a-project}

Primeiro, use o terminal e navegue até o diretório do seu projeto. Você precisará executar o seguinte comando:

Se você usa [Yarn](https://yarnpkg.com/), execute:

```bash
yarn add --dev flow-bin
```

Se você usa [npm](https://www.npmjs.com/), execute:

```bash
npm install --save-dev flow-bin
```

Este comando instala a versão mais recente do Flow no seu projeto.

Agora, adicione `flow` à seção `"scripts"` do seu `package.json` para conseguir usar isto no terminal:

```js{4}
{
  // ...
  "scripts": {
    "flow": "flow",
    // ...
  },
  // ...
}
```

Por fim, execute um dos comandos a seguir:

Se você usa [Yarn](https://yarnpkg.com/), execute:

```bash
yarn run flow init
```

Se você usa [npm](https://www.npmjs.com/), execute:

```bash
npm run flow init
```

Este comando criará um arquivo de configuração do Flow que você precisará fazer commit.

### Separando a Sintaxe do Flow do Código Compilado {#stripping-flow-syntax-from-the-compiled-code}

O Flow estende a linguagem JavaScript com uma sintaxe especial para anotações de tipo. Entretanto, os navegadores não estão cientes desta sintaxe. Assim, precisamos ter certeza que a sintaxe do Flow não permaneça no código JavaScript compilado que é enviado ao navegador.

A forma exata de fazer isso depende das ferramentas que você usa para compilar o JavaScript.

#### Create React App {#create-react-app}

Se o seu projeto foi configurado com [Create React App](https://github.com/facebookincubator/create-react-app), parabéns! As anotações do Flow já estão sendo retiradas por padrão, então você não precisa fazer mais nada nesta etapa.

#### Babel {#babel}

>Nota:
>
>Estas instruções *não* são para usuários do Create React App. Apesar do Create React App usar Babel por baixo dos panos, ele já está configurado para entender o Flow. Siga estes passos somente se você *não* usa o Create React App.

Se você configurou o Babel manualmente no seu projeto, precisará instalar um preset especial para Flow.

Se você usa [Yarn](https://yarnpkg.com/), execute:

```bash
yarn add --dev @babel/preset-flow
```

Se você usa [npm](https://www.npmjs.com/), execute:

```bash
npm install --save-dev @babel/preset-flow
```

Então adicione o preset `flow` à sua [configuração do Babel](https://babeljs.io/docs/usage/babelrc/). Por exemplo, se você configura o Babel através do arquivo `.babelrc`, pode ficar parecido com isto:

```js{3}
{
  "presets": [
    "@babel/preset-flow",
    "react"
  ]
}
```

Isto permitirá que você use a sintaxe do Flow no seu código.

>Nota:
>
>O Flow não requer o preset `react`, mas eles são frequentemente usados juntos. O Flow por si só já vem pronto para entender a sintaxe JSX.

#### Outras Configurações de Build {#other-build-setups}

Se você não usa Create React App nem Babel, você pode usar [flow-remove-types](https://github.com/flowtype/flow-remove-types) para remover as anotações de tipos.

### Executando o Flow {#running-flow}

Se você seguiu os passos acima, deve ser capaz de executar o Flow pela primeira vez.

Se você usa [Yarn](https://yarnpkg.com/), execute:

```bash
yarn flow
```

Se você usa [npm](https://www.npmjs.com/), execute:

```bash
npm run flow
```

Você deverá ver uma mensagem como esta:

```
No errors!
✨  Done in 0.17s.
```

### Adicionando Anotações de Tipo do Flow {#adding-flow-type-annotations}

Por padrão, o Flow checa apenas os arquivos que incluem esta anotação:

```js
// @flow
```

Normalmente, é posicionado no topo de um arquivo. Tente adicioná-la em alguns arquivos do seu projeto e execute `yarn flow` ou `npm run flow` para ver se o Flow já achou algum problema.

Também há [uma opção](https://flow.org/en/docs/config/options/#toc-all-boolean) para forçar o Flow a checar *todos* os arquivos independente se há a anotação ou não. Isto pode ser meio turbulento para projetos já existentes, mas é sensato para um novo projeto se você quer deixá-lo totalmente tipado com o Flow.

Agora está tudo certo! Recomendamos dar uma lida nos seguintes recursos para aprender mais sobre o Flow (em inglês):

* [Flow Documentation: Type Annotations](https://flow.org/en/docs/types/)
* [Flow Documentation: Editors](https://flow.org/en/docs/editors/)
* [Flow Documentation: React](https://flow.org/en/docs/react/)
* [Linting in Flow](https://medium.com/flow-type/linting-in-flow-7709d7a7e969)

## TypeScript {#typescript}

O [TypeScript](https://www.typescriptlang.org/) é uma linguagem de programação desenvolvida pela Microsoft. É um superset tipado do JavaScript e inclui seu próprio compilador. Sendo uma linguagem tipada, o TypeScript consegue detectar erros e bugs em tempo de compilação, muito antes do seu aplicativo iniciar. Você pode aprender mais sobre o uso do TypeScript com React [aqui](https://github.com/Microsoft/TypeScript-React-Starter#typescript-react-starter).

Para usar o TypeScript você precisa:
* Adicionar o TypeScript como uma dependência ao seu projeto
* Configurar as opções de compilação do TypeScript
* Usar as extensões de arquivos corretas
* Adicionar definições para bibliotecas que você usa

Vamos passar por cada uma em detalhes.

### Usando TypeScript com Create React App {#using-typescript-with-create-react-app}

O Create React App já vem com suporte para o TypeScript.

Para criar um **novo projeto** com suporte ao TypeScript, execute:

```bash
npx create-react-app my-app --template typescript
```

Vocë também pode adicioná-lo a um **projeto Create React App existente**, [como está documentado aqui](https://facebook.github.io/create-react-app/docs/adding-typescript).

>Nota:
>
>Se você usa Create React App, você pode **pular o resto desta página**. Ela descreve a configuração manual no qual não se aplicam a usuários do Create React App.


### Adicionando TypeScript a um Projeto {#adding-typescript-to-a-project}
Tudo começa executando um comando no seu terminal.

Se você usa [Yarn](https://yarnpkg.com/), execute:

```bash
yarn add --dev typescript
```

Se você usa [npm](https://www.npmjs.com/), execute:

```bash
npm install --save-dev typescript
```

Parabéns! Você instalou a versão mais recente do TypeScript no seu projeto. Instalar o TypeScript nos dá acesso ao comando `tsc`. Antes da configuração, vamos adicionar `tsc` à seção "scripts" no nosso `package.json`:

```js{4}
{
  // ...
  "scripts": {
    "build": "tsc",
    // ...
  },
  // ...
}
```

### Configurando o Compilador do TypeScript {#configuring-the-typescript-compiler}
O compilador não é de ajuda alguma até que o dizemos o que deve fazer. No TypeScript, essas regras são definidas em um arquivo especial chamado `tsconfig.json`. Para gerar esse arquivo execute:

Se você usa [Yarn](https://yarnpkg.com/), execute:

```bash
yarn run tsc --init
```

Se você usa [npm](https://www.npmjs.com/), execute:

```bash
npx tsc --init
```

Olhando o então gerado `tsconfig.json`, você pode ver que há muitas opções que você pode usar para configurar o compilador. Para uma descrição detalhada de todas as opções, veja [aqui](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html).

Das muitas opções, daremos uma olhada em `rootDir` e `outDir`. Na sua forma verdadeira, o compilador receberá arquivos typescript e gerará arquivos javascript. Entretanto, não queremos que nossos arquivos fontes e a saída gerada fiquem confusos.

Vamos cobrir isso em dois passos:
* Primeiramente, vamos organizar a estrutura do nosso projeto desta forma. Iremos colocar todo o nosso código-fonte na pasta `src`.

```
├── package.json
├── src
│   └── index.ts
└── tsconfig.json
```

* Depois, diremos ao compilador onde nosso código-fonte está e para onde a saída deverá ir.

```js{6,7}
// tsconfig.json

{
  "compilerOptions": {
    // ...
    "rootDir": "src",
    "outDir": "build"
    // ...
  },
}
```

Ótimo! Agora quando executarmos o nosso script de build, o compilador vai colocar o JavaScript gerado na pasta `build`. O [TypeScript React Starter](https://github.com/Microsoft/TypeScript-React-Starter/blob/master/tsconfig.json) oferece um `tsconfig.json` com um bom conjunto de regras para você começar.

Geralmente, você não quer manter o JavaScript gerado no seu repositório, então tenha certeza de que adicionou a pasta do build no seu `.gitignore`.

### Extensões de Arquivos {#file-extensions}
No React, você provavelmente escreverá seus componentes em um arquivo `.js`. No TypeScript temos 2 extensões de arquivo:

`.ts` é a extensão de arquivo padrão, enquanto `.tsx` é uma extensão especial usada em arquivos que contém `JSX`.

### Executando o TypeScript {#running-typescript}

Se você seguiu as instruções acima, você deverá ser capaz de executar o TypeScript pela primeira vez.

```bash
yarn build
```

Se você usa npm, execute:

```bash
npm run build
```

Se você não vê um output, significa que completou com sucesso.


### Definições de Tipo {#type-definitions}
Para ser capaz de mostrar erros e dicas de outros pacotes, o compilador depende dos arquivos de declaração. Um arquivo de declaração oferece todo a informação de tipos sobre uma biblioteca. Isso nos permite usar bibliotecas javascript, como as que estão no npm, no nosso projeto.

Existem duas formas principais de conseguir declarações para uma biblioteca:

__Bundled__ - A biblioteca empacota o seu próprio arquivo de declaração. Isto é ótimo para nós, pois tudo o que precisaremos fazer é instalar a biblioteca e estaremos prontos para usá-la. Para verificar se uma biblioteca tem tipos empacotados, procure por um arquivo `index.d.ts` no projeto. Algumas bibliotecas terá o arquivo especificado em seus `package.json`, no campo `typings` ou `types`.

__[DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped)__ - DefinitelyTyped é um repositório enorme de declarações para bibliotecas que não empacotam um arquivo de declaração. As declarações são feitas pelo público e gerenciadas pela Microsoft e contribuidores de código aberto. O React, por exemplo, não empacota seu próprio arquivo de declaração. Em vez disso, nós podemos pegá-lo do DefinitelyTyped. Para isso, entre este comando no seu terminal:

```bash
# yarn
yarn add --dev @types/react

# npm
npm i --save-dev @types/react
```

__Declarações Locais__
Algumas vezes o pacote que você quer usar não empacota declarações nem está disponível no DefinitelyTyped. Neste caso, podemos ter um arquivo de declaração local. Para fazer isto, crie um arquivo `declarations.d.ts` na raiz da pasta do seu código fonte. Uma declaração simples ficaria assim:

```typescript
declare module 'querystring' {
  export function stringify(val: object): string
  export function parse(val: string): object
}
```

Vocé agora pode começar a programar! Nós recomendamos verificar os seguintes recursos para aprender mais sobre TypeScript (em inglês):

<<<<<<< HEAD
* [TypeScript Documentation: Basic Types](https://www.typescriptlang.org/docs/handbook/basic-types.html)
=======
* [TypeScript Documentation: Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
>>>>>>> 25f756d8e3800afb032cb31ce3626d6134e31e38
* [TypeScript Documentation: Migrating from JavaScript](https://www.typescriptlang.org/docs/handbook/migrating-from-javascript.html)
* [TypeScript Documentation: React and Webpack](https://www.typescriptlang.org/docs/handbook/react-&-webpack.html)

## Reason {#reason}

[Reason](https://reasonml.github.io/) não é uma linguagem nova; é uma nova sintaxe e cadeia de ferramentas disponibilizada pela linguagem testada em batalha [OCaml](https://ocaml.org/). A Reason dá a OCaml uma sintaxe familiar voltada para programadores JavaScript e atende ao fluxo de trabalho do NPM/Yarn que o pessoal já conhece.

A Reason é desenvolvida no Facebook e é usada em alguns dos seus produtos como o Messenger. Ainda é de alguma forma experimental, mas ela tem [ligações dedicadas com o React](https://reasonml.github.io/reason-react/) mantida pelo Facebook e pela [comunidade vibrante](https://reasonml.github.io/docs/en/community.html).

## Kotlin {#kotlin}

[Kotlin](https://kotlinlang.org/) é uma linguagem estaticamente tipada desenvolvida pelo JetBrains. As plataformas alvo dela incluem o JVM, Android, LLVM e [JavaScript](https://kotlinlang.org/docs/reference/js-overview.html).

JetBrains desenvolve e mantém algumas ferramentas especificamente para a comunidade do React: [ligações com o React](https://github.com/JetBrains/kotlin-wrappers) assim como [Create React Kotlin App](https://github.com/JetBrains/create-react-kotlin-app). A última auxilia você a começar a construir aplicações React com Kotlin sem precisar configurar um build.

## Outras Linguagens {#other-languages}

Perceba que há outras linguagens estaticamente tipadas que compilam para JavaScript e assim são compatíveis com React. Por exemplo, [F#/Fable](https://fable.io/) com [elmish-react](https://elmish.github.io/react). Verifique os respectivos sites para mais informações e sinta-se livre para adicionar a esta página mais linguagens estaticamente tipadas que trabalham com React!
