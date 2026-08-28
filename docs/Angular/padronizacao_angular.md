# Padronização de Código no Angular

Este documento serve como um guia de padronização de código para o desenvolvimento frontend com **Angular**. Ele foi elaborado para garantir a consistência, a manutenibilidade e a escalabilidade dos projetos, com foco na padronização, legibilidade e qualidade do código. Seguir estas diretrizes e adotar as ferramentas e práticas descritas aqui é fundamental para a colaboração eficiente da equipe.

> *Este guia é um documento vivo e pode ser atualizado conforme a equipe evolui e adota novas práticas. Se você tiver alguma dúvida ou sugestão, sinta-se à vontade para discuti-la com a equipe.*

> ***(Texto tachado ainda está em estudo)***

## 1. Ferramentas e Configurações

Para garantir um ambiente de desenvolvimento coeso e automatizado, usaremos as seguintes ferramentas:

- **Editor de Código**: Recomendado o **VS Code** como editor de desenvolvimento. **É obrigatório **ter as seguintes extensões instaladas:

  - **ESLint (Microsoft)**: Essencial para a padronização e qualidade do nosso código.
  - **Prettier - Code formatter (prettier.io)**: Garante a formatação automática e consistente do código.

- **Extensões Indicadas**: As seguintes extensões ajudam no desenvolvimento, mas não são obrigatórias.

  - **Angular Extension Pack (Loiane Groner)**: Um pacote completo com várias extensões úteis para Angular. Caso não queira instalar o pacote completo, recomendamos pelo menos o Angular Language Service (angular.dev) e o EditorConfig for VS Code (EditorConfig).  
Atenção: Se você instalar este pacote, certifique-se de desinstalar a extensão *`TypeScript Hero`*, pois ela não é mais suportada e pode causar conflitos.
  - **Angular Snippets 2025 – v19 (JMGomes)**: Oferece *snippets* de código para agilizar o desenvolvimento, focando nas novas funcionalidades do Angular 19, como os **signals** e a nova sintaxe para o controle de fluxo (*`@if`, `@for`*).
  - **Error Lens (Alexander)**: Destaca erros e avisos diretamente na linha de código, melhorando a visibilidade.
  - **GitLens – Git supercharged (GitKraken)**: Adiciona funcionalidades avançadas para trabalhar com Git, como a visualização de quem alterou cada linha de código.
  - **HTML CSS Support (ecmel)**: Oferece preenchimento automático e validação para HTML e CSS.

- **Análise Estática de Código**:

  - **ESLint**: Analisa estaticamente o código para identificar e corrigir erros, más práticas e padrões inconsistentes, indo além da simples formatação. A configuração é definida no arquivo *eslint.config.mjs*, que deve ser respeitado ao máximo.

  > *Inicialmente, adotaremos um padrão mais simples para garantir a compatibilidade com projetos legados. No entanto, o objetivo é evoluir para os padrões de mercado em futuras atualizações.*

  - ~~**Husky**~~: Automatiza a execução de *`lint`* e *`prettier`* antes de cada *`commit`*, prevenindo a inclusão de código fora do padrão no repositório.

  - ~~**Lint-staged**~~: Executa o *`lint`* e *`prettier`* apenas nos arquivos que estão sendo preparados para o commit (*`staged`*), tornando o processo mais rápido e eficiente.

## 2. Estrutura de Arquivos e Nomenclatura

Estes padrões garantem que o código seja fácil de ler, entender e manter.

> *Com o lançamento do Angular 20, a equipe do framework promoveu uma mudança significativa nas convenções de nomenclatura. Agora o Angular CLI, por padrão, não adiciona mais os sufixos de tipo aos nomes de arquivos e classes. Resolvemos manter o padrão anterior, ou seja, manteremos os sufixos de tipo aos nomes de arquivos.*

### Nomenclatura

- **Arquivos e Pastas**: Utilize `kebab-case` (*`exemplo-de-arquivo.component.ts`*).

- **Classes, Interfaces e Enums**: Utilize `PascalCase` (*`MinhaClasse`, `MinhaInterface`, `MeuEnum`*).

- **Variáveis e Funções**: Utilize `camelCase` (*`minhaVariavel`, `minhaFuncao`*).

  - **Variáveis e Funções Privadas**: deve iniciar com underscore (*\_minhaVariavelPrivada*). Isso serve como uma convenção visual para indicar que o membro não deve ser acessado externamente.

  - **Variáveis Observables**: deve terminar com $ (*listaA$*). Essa é uma convenção amplamente utilizada no ecossistema RxJS para identificar imediatamente que a variável é um *Observable*.

- **Componentes**: O seletor do componente deve seguir o padrão *`app-nome-componente`* (`<app-nome-componente></app-nome-componente>`).

  - **Componentes compartilhados externamente**: SOMENTE componentes que possam ser compartilhados com outros projetos, pensando em uma futura biblioteca nossa, devem seguir o padrão *`ds-nome-do-componente`* (`<ds-nome-componente></ds-nome-componente>`).

### Estrutura de Pastas

A organização do código é fundamental para a escalabilidade do projeto.

```text
src/
├── app/
│   ├── core/ # Funcionalidades essenciais e configurações globais específicas do projeto
│   │   ├── auth/ # Lógica de autenticação
│   │   │   ├── auth.service.ts
│   │   │   └── auth.guard.ts
│   │   ├── interceptors/ # Interceptores HTTP
│   │   │   └── auth.interceptor.ts
│   │   ├── services/ # Serviços globais e utilitários essenciais compartilhados entre várias features
│   │   │   └── logger.service.ts
│   │   ├── styles/ # Pasta para arquivos de estilo global
│   │   │   └── global-styles.scss 
│   │   └── guards/ # Guards de navegação
│   │       └── role.guard.ts
│   │
│   ├── shared/ # Código reutilizável entre várias features, utilitários e funções auxiliares
│   │   ├── components/ # Componentes reutilizáveis
│   │   │   ├── button/ # Componente de botão como standalone
│   │   │   │   └── button.component.ts
│   │   │   └── spinner/ # Componente de spinner como standalone
│   │   │       └── spinner.component.ts
│   │   ├── directives/ # Diretivas reutilizáveis
│   │   │   └── tooltip.directive.ts
│   │   ├── pipes/ # Pipes reutilizáveis
│   │   │   └── date.pipe.ts
│   │   ├── models/ # Modelos e interfaces compartilhadas
│   │   │   └── user.model.ts
│   │   ├── utils/ # Funções auxiliares genéricas
│   │   │   └── format-date.ts
│   │   ├── helpers/ # Funções auxiliares e helpers
│   │   │   ├── form-helpers.ts
│   │   │   └── email.helpers.ts # Funções auxiliares relacionadas a email (formatar, normalizar, etc.)
│   │   └── validators/ # Validações genéricas
│   │       └── email.validator.ts # Função de validação de email
│   │
│   ├── features/ # Funcionalidades específicas do aplicativo
│   │   ├── auth/
│   │   │   ├── login/
│   │   │   │   ├── login.component.ts # Standalone component
│   │   │   │   └── login.component.html
│   │   │   ├── logout/
│   │   │   └── auth.service.ts
│   │   ├── user/ # Feature de usuários
│   │   │   ├── user-list/ # Listagem de usuários
│   │   │   │   ├── user-list.component.ts # Standalone component
│   │   │   │   └── user-list.component.spec.ts
│   │   │   ├── user-detail/ # Detalhes do usuário
│   │   │   │   ├── user-detail.component.ts # Standalone component
│   │   │   │   └── user-detail.component.spec.ts
│   │   │   └── user.service.ts # Serviço para usuários
│   │   ├── product/ # Feature de produtos
│   │   │   ├── product-list/ # Listagem de produtos
│   │   │   │   ├── product-list.component.ts # Standalone component
│   │   │   │   └── product-list.component.spec.ts
│   │   │   ├── product-detail/ # Detalhes do produto
│   │   │   │   ├── product-detail.component.ts # Standalone component
│   │   │   │   └── product-detail.component.spec.ts
│   │   │   └── product.service.ts
│   │   └── order/ # Feature de pedidos
│   │       └── order-list/ # Listagem de pedidos
│   │
│   ├── app.config.ts # Configuração principal da aplicação standalone (provideRouter, provideHttpClient, etc.)
│   ├── app.routes.ts # Arquivo de rotas principal da aplicação
│   ├── app.component.ts # Componente raiz da aplicação standalone
│   └── app.component.html # Template do componente raiz
│
├── assets/ # Imagens, ícones e outros ativos estáticos
├── environments/ # Configurações de ambiente (desenvolvimento, produção, etc.)
├── styles.scss # Arquivo global de estilos (SCSS)
└── index.html # Página inicial do aplicativo
```

### Configurando o Padrão de Nomenclatura no Angular 20

Para configurar o Angular CLI e manter o padrão de nomenclatura com prefixo (*.component.ts, .service.ts*) no Angular 20, adicione a seção `schematics` no arquivo de configuração **angular.json**.

```json
{
  "projects": { ... },
  "schematics": {
    "@schematics/angular:component": { "type": "component" },
    "@schematics/angular:directive": { "type": "directive" },
    "@schematics/angular:service": { "type": "service" },
    "@schematics/angular:guard": { "typeSeparator": "." },
    "@schematics/angular:interceptor": { "typeSeparator": "." },
    "@schematics/angular:module": { "typeSeparator": "." },
    "@schematics/angular:pipe": { "typeSeparator": "." },
    "@schematics/angular:resolver": { "typeSeparator": "." }
  }
}
```

## 3. Boas Práticas

- **Typescript**: Sempre utilize tipagem estrita para variáveis, parâmetros de funções e retornos. Isso aumenta a segurança e a previsibilidade do código.
- **Comentários**: Comente apenas quando necessário para explicar a "intenção" por trás do código, não o que ele faz. O código deve ser autoexplicativo.
- **Standalone**: Todos os novos componentes, diretivas e pipes devem ser standalone (a partir do Angular 19, `standalone: true` é o padrão implícito do framework). A utilização de *NgModules* deve ser evitada e só é permitida em casos de integração com bibliotecas ou partes legadas do sistema.
- **Padrões de Acessibilidade (ARIA)**: Use, sempre que possível, os atributos `aria-*` e padrões de acessibilidade nos elementos interativos.
- **Lazy Loading**: é uma técnica de otimização no desenvolvimento web onde o Angular carrega componentes ou recursos apenas quando eles são necessários. O *Lazy Loading* é implementado nas rotas usando `loadComponent` ou `loadChildren`:
```typescript
{
  path: 'products',
  // O Angular só carrega este componente quando o usuário navegar para 'products'
  loadComponent: () => import('./products/products.component').then(m => m.ProductsComponent)
}
```
- **Inject**: A função **inject()** é uma API moderna do Angular que permite injetar dependências de forma mais flexível e organizada. Prefira *inject()* em vez da injeção de parâmetros do construtor, pois ela pode ser usada em qualquer lugar onde o contexto de injeção esteja disponível, não se limitando apenas ao construtor.
- **Protected**: Use o modificador **protected** em propriedades e métodos de uma classe de componente que são utilizados exclusivamente pelo seu template, tornando o encapsulamento mais seguro. Exemplo:

```typescript
@Component({
  selector: 'app-user-card',
  template: `
    <h2>{{ getFullName() }}</h2>
    <p>ID: {{ userId }}</p>
  `
})
export class UserCardComponent {
  // Propriedade acessível apenas pelo template e classes derivadas
  protected userId: string = '12345';

  // Método acessível apenas pelo template e classes derivadas
  protected getFullName(): string {
    return 'João da Silva';
  }
}
```
- **Testes**: Utilize **Vitest** (padrão oficial moderno do ecossistema Angular) ou **Jest** para a execução de testes unitários rápidos e seguros.
- **Readonly e Imutabilidade com Signals**: Utilize as novas funções baseadas em Signal (`input()`, `input.required()`, `viewChild()`), que já são somente leitura por definição. Para propriedades de componentes legados decoradas com `@Input()`, `@ViewChild()` ou `@ContentChild()`, continue utilizando o modificador `readonly` para evitar reatribuições acidentais.
- **Angular Signals**: Utilize **Signals** para um gerenciamento de estado mais reativo e eficiente. Eles são a ferramenta padrão para o estado local e comunicação reativa de componentes, substituindo *Observables* em cenários síncronos e de interface através das novas APIs `signal()`, `computed()`, `input()`, `output()`, `model()`, `linkedSignal()` e `resource()`.
- **RxJS**: Continue utilizando *Observables* para lidar com fluxos de dados assíncronos contínuos (WebSockets, SSE) ou orquestrações complexas com operadores (`debounceTime`, `switchMap`, `retry`). Converta-os para Signals nos componentes utilizando `toSignal()` para consumo direto no template.
- **Single Responsibility Principle**: Cada serviço deve ter uma única responsabilidade. Por exemplo, um *UserService* deve ser responsável por gerenciar dados de usuários (CRUD), e não por fazer requisições HTTP para outras entidades (como login ou verificar sessão). **Evite** `God Class / God Service`, classes ou serviços "deuses" que acumulam funções desconexas, como *UserService* gerenciar senha, enviar e-mail ou gerar PDF. 
- **ProvidedIn**: Nos serviços, a propriedade **providedIn: 'root'** registra o serviço como um **singleton** (uma única instância global), ideal para serviços que compartilham os mesmos dados por toda a aplicação. Já a propriedade **providedIn: 'any'** cria uma nova instância para cada módulo ou contexto que o injeta, sendo perfeita para cenários onde é preciso isolar o estado. Por exemplo, para ter a informação do usuário logado em qualquer lugar da aplicação usamos **providedIn: 'root'**, já para gerenciar as informações de um formulário ou filtros específicos, usamos *providedIn: 'any'*.
- **Lógica no Componente**: Mantenha a lógica de negócio **separada** dos *templates*. O **template** deve ser usado apenas para a apresentação dos dados, enquanto o **componente** deve ser o responsável por manipular os dados e interagir com serviços.
  - No template:
    - Em vez de: `<div> {{ user.firstName + ' ' + user.lastName }} </div>`
    - Use: `<div> {{ fullName() }} </div>` *(sinal computado no componente)* ou a diretiva local `@let fullName = user.firstName + ' ' + user.lastName;`
  - Nos formulários, utilize **Reactive Forms** ou a nova API **Signal Forms**.
- **Componentes de Apresentação e Contêineres (Dumb/Smart)**: Separe a responsabilidade de apresentação da lógica de negócios. Componentes de apresentação (*dumb*) apenas recebem dados via `input()` e emitem eventos com `output()`, enquanto componentes contêineres (*smart*) interagem com serviços e gerenciam o fluxo de dados.
- **OnPush Change Detection e Zoneless**: A estratégia **OnPush** e a arquitetura **Zoneless** otimizam o desempenho do Angular ao atualizar de forma cirúrgica apenas os nós do template associados a Signals que sofreram alteração, reduzindo drasticamente os ciclos de verificação desnecessários.
- **Padrões de Nomenclatura para Manipuladores e Eventos**:
  - **Manipuladores de Eventos**: Nomeie as funções pelo que elas fazem, não pelo evento de disparo. Essa prática melhora a legibilidade do código, deixando clara a intenção da ação.
    - Em vez de: `<button (click)="onClick()">Salvar</button>`
    - Use: `<button (click)="salvarDados()">Salvar</button>`
  - **Exceção para lógica complexa**: Em casos onde a lógica de tratamento de eventos é particularmente ampla (por exemplo, ao lidar com todas as teclas pressionadas), é aceitável usar um nome descritivo como *whenKeydown*.
  - **Eventos de Componentes (`output()` / `@Output`)**: Evite o prefixo `on` para evitar confusão com eventos nativos do DOM. Prefira verbos de ação ou o particípio passado:
    - Em vez de: `onClick = output<void>();`
    - Use: `salvar = output<void>();` ou `salvo = output<void>();`

## 4. Configurações nos Projetos

Esses arquivos de configuração devem ser incluídos em todos os projetos e, preferencialmente, não devem ser modificados para garantir a padronização.

### eslint.config.mjs – Define as regras do analisador de código ESLint.
```javascript
// @ts-check
import eslintJs from '@eslint/js';
import { config, configs as _tsConfigs } from 'typescript-eslint';
import { configs as _angularConfigs, processInlineTemplates } from 'angular-eslint';
import prettierConfig from 'eslint-config-prettier';
import prettierPlugin from 'eslint-plugin-prettier';

const { configs } = eslintJs;

export default config(
  {
    files: ['**/*.ts'],
    ignores: ['**/*.spec.ts'],
    extends: [
      configs.recommended,
      ..._tsConfigs.recommended,
      ..._tsConfigs.stylistic,
      ..._angularConfigs.tsRecommended,
      prettierConfig,
    ],
    plugins: {
      prettier: prettierPlugin,
    },
    processor: processInlineTemplates,
    rules: {
      'prettier/prettier': 'error',
      'no-console': [
        'warn',
        {
          allow: ['warn', 'error'],
        },
      ],
      'no-underscore-dangle': 'off',
      '@angular-eslint/directive-selector': [
        'error',
        {
          type: 'attribute',
          prefix: ['app', 'ds'],
          style: 'camelCase',
        },
      ],
      '@angular-eslint/component-selector': [
        'error',
        {
          type: 'element',
          prefix: ['app', 'ds'],
          style: 'kebab-case',
        },
      ],
    },
  },
  {
    files: ['**/*.html'],
    extends: [
      ..._angularConfigs.templateRecommended,
      ..._angularConfigs.templateAccessibility,
    ],
    rules: {},
  },
);
```

### .gitignore – Define quais arquivos o Git não deve gerenciar.
```gitignore
# -----------------------------------------------------------------------------
# Common Development Files & System Exclusions
# -----------------------------------------------------------------------------
# IDE (IntelliJ IDEA)
.idea/
*.iml
*.ipr
*.iws
out/
# IDE (Visual Studio Code)
.vscode/
!.vscode/settings.json   # Keep specific settings to share
!.vscode/tasks.json      # Keep specific tasks to share
!.vscode/launch.json     # Keep launch configurations to share
!.vscode/extensions.json # Keep recommended extensions to share
!.vscode/*.code-snippets # Keep shared code snippets
.vscode-test/            # VS Code extension testing temporary files
.history/                # Local History for VS Code
*.vsix                   # Built Visual Studio Code Extensions
*.todo                   # VS Code TODO files
# Other IDEs/Editors
.c9/                 # AWS/Cloud9 IDE
*.sublime-workspace  # Sublime Text workspace file
# Cache Files (Generic)
.cache/            # Generic cache directory
.eslintcache       # ESLint cache
.sass-cache/       # Sass compiler cache
# Estaleiro
estaleiro/deploy/  # Ignore 'deploy' do estaleiro
# Test & Coverage Files (Common)
/coverage/         # Test coverage reports (e.g., Karma, Jest, Vitest)
testem.log         # Testem log file
# Runtime & Temporary Data (Common)
pids/
*.pid
*.seed
*.pid.lock
# System & OS Files
.DS_Store          # macOS
Thumbs.db          # Windows
# Version Control Specific Files (Git)
.git/              # Git directory (usually not explicitly ignored)
.gitignore         # The .gitignore file itself
.gitattributes
# Environment Variables & Sensitive Files (Common)
.env
.env.development.local
.env.test.local
.env.production.local
.env.local
local.properties    # Often contains local developer-specific properties
/DevProfile         # Specific development profile or sensitive configurations
# -----------------------------------------------------------------------------
# Angular & Node.js Specific Files
# -----------------------------------------------------------------------------
# Build Artifacts
/dist/             # Angular build output directory
.angular/cache/    # Angular build cache
/out-tsc           # TypeScript compiler output
*.tsbuildinfo      # TypeScript build info file
*.js.map           # JavaScript source maps
# Dependencies & Package Manager Files
/node_modules/     # Node.js dependencies
npm-debug.log*
yarn-debug.log*
yarn-error.log*
lerna-debug.log*
.pnpm-debug.log*
.npm/              # Optional npm cache directory
.yarn/             # Yarn cache/specifics (if not using v2/Berry)
.yarn-integrity    # Yarn integrity file
package-lock.json
yarn.lock
# Yarn v2 (Berry) specific files
.yarn/cache/
.yarn/unplugged/
.yarn/build-state.yml
.yarn/install-state.gz
.pnp.*
# Test Specifics
e2e/*.js           # Compiled E2E test files
e2e/*.map          # Source maps for E2E test files
cypress.env.json   # Cypress environment variables (often sensitive)
# Logs & Diagnostic Reports
logs/              # Node.js specific logs (if not covered by common)
*.log
libpeerconnection.log/ # WebRTC specific log
# Node.js Diagnostic reports (https://nodejs.org/api/report.html)
report.[0-9]*.[0-9]*.[0-9]*.[0-9]*.json
# Output of 'npm pack'
*.tgz                     # NPM package archives
# Serverless Webpack directories
.webpack/
# Optional REPL history
.node_repl_history
```

### .prettierrc – Define as regras de formatação para o Prettier.
```json
{
  "printWidth": 120,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "quoteProps": "as-needed",
  "trailingComma": "all",
  "bracketSpacing": true,
  "objectWrap": "preserve",
  "bracketSameLine": true,
  "arrowParens": "always",
  "proseWrap": "preserve",
  "htmlWhitespaceSensitivity": "ignore",
  "endOfLine": "lf",
  "embeddedLanguageFormatting": "off",
  "singleAttributePerLine": true,
  "plugins": ["prettier-plugin-multiline-arrays"],
  "multilineArraysWrapThreshold": 2,
  "multilineArraysLinePattern": "threshold",
  "overrides": [
    {
      "files": "*.ts",
      "options": {
        "semi": true,
        "singleQuote": true
      }
    },
    {
      "files": "*.html",
      "options": {
        "semi": false,
        "singleQuote": false,
        "htmlWhitespaceSensitivity": "ignore",
        "singleAttributePerLine": true
      }
    },
    {
      "files": "*.scss",
      "options": {
        "semi": false,
        "trailingComma": "none",
        "singleQuote": false
      }
    },
    {
      "files": "*.js",
      "options": {
        "semi": false,
        "singleQuote": true,
        "trailingComma": "all",
        "printWidth": 80
      }
    },
    {
      "files": "*.json",
      "options": {
        "semi": false,
        "singleQuote": false,
        "trailingComma": "none"
      }
    }
  ]
}
```

### .prettierignore – Lista de arquivos e diretórios que o Prettier não deve formatar. Deve ser similar ao *.gitignore* em relação a dependências e builds.
```
.prettierrc
.npmrc
.editorconfig
.gitignore
# -----------------------------------------------------------------------------
# Common Development Files & System Exclusions
# -----------------------------------------------------------------------------
# IDE (IntelliJ IDEA)
.idea/
*.iml
*.ipr
*.iws
out/
# IDE (Visual Studio Code)
.vscode/
!.vscode/settings.json   # Keep specific settings to share
!.vscode/tasks.json      # Keep specific tasks to share
!.vscode/launch.json     # Keep launch configurations to share
!.vscode/extensions.json # Keep recommended extensions to share
!.vscode/*.code-snippets # Keep shared code snippets
.vscode-test/            # VS Code extension testing temporary files
.history/                # Local History for VS Code
*.vsix                   # Built Visual Studio Code Extensions
*.todo                   # VS Code TODO files
# Other IDEs/Editors
.c9/                 # AWS/Cloud9 IDE
*.sublime-workspace  # Sublime Text workspace file
# Cache Files (Generic)
.cache/            # Generic cache directory
.eslintcache       # ESLint cache
.sass-cache/       # Sass compiler cache
# Estaleiro
estaleiro/deploy/  # Ignore 'deploy' do estaleiro
# Test & Coverage Files (Common)
/coverage/         # Test coverage reports (e.g., Karma, Jest, Vitest)
testem.log         # Testem log file
# Runtime & Temporary Data (Common)
pids/
*.pid
*.seed
*.pid.lock
# System & OS Files
.DS_Store          # macOS
Thumbs.db          # Windows
# Version Control Specific Files (Git)
.git/              # Git directory (usually not explicitly ignored)
.gitignore         # The .gitignore file itself
.gitattributes
# Environment Variables & Sensitive Files (Common)
.env
.env.development.local
.env.test.local
.env.production.local
.env.local
local.properties    # Often contains local developer-specific properties
/DevProfile         # Specific development profile or sensitive configurations
# -----------------------------------------------------------------------------
# Angular & Node.js Specific Files
# -----------------------------------------------------------------------------
# Build Artifacts
/dist/             # Angular build output directory
/out-tsc           # TypeScript compiler output
*.tsbuildinfo      # TypeScript build info file
*.js.map           # JavaScript source maps
# Dependencies & Package Manager Files
/node_modules/     # Node.js dependencies
npm-debug.log*
yarn-debug.log*
yarn-error.log*
lerna-debug.log*
.pnpm-debug.log*
.npm/              # Optional npm cache directory
.yarn/             # Yarn cache/specifics (if not using v2/Berry)
.yarn-integrity    # Yarn integrity file
package-lock.json
yarn.lock
# Yarn v2 (Berry) specific files
.yarn/cache/
.yarn/unplugged/
.yarn/build-state.yml
.yarn/install-state.gz
.pnp.*
# Test Specifics
e2e/*.js           # Compiled E2E test files
e2e/*.map          # Source maps for E2E test files
cypress.env.json   # Cypress environment variables (often sensitive)
# Logs & Diagnostic Reports
logs/              # Node.js specific logs (if not covered by common)
*.log
libpeerconnection.log/ # WebRTC specific log
# Node.js Diagnostic reports (https://nodejs.org/api/report.html)
report.[0-9]*.[0-9]*.[0-9]*.[0-9]*.json
# Output of 'npm pack'
*.tgz                     # NPM package archives
# Serverless Webpack directories
.webpack/
# Optional REPL history
.node_repl_history
```

### .editorconfig – Define um padrão de formatação básico para editores de código que não possuem integração com o Prettier, evitando inconsistências.
```
# Editor configuration, see https://editorconfig.org
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 2
insert_final_newline = true
trim_trailing_whitespace = true
max_line_length = 120
end_of_line = lf

[*.{js,ts,json,html,css,scss}]
quote_type = single
ij_typescript_use_double_quotes = false
[*.{json,editorconfig}]
trim_trailing_whitespace = false
[*.html]
trim_trailing_whitespace = false
[*.md]
max_line_length = off
trim_trailing_whitespace = false
insert_final_newline = false
 
```

### package.json – Scripts e configurações no arquivo do projeto:

```json
{
  "type": "module",
  "scripts": {
    "lint": "ng lint",
    "format": "prettier --write \"src/**/*.{ts,html,css,scss}\"",
    "prepare": "husky install"
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{ts,js,scss,html,json}": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```

> **Verificação de Dependências**: Certifique-se de que `prettier`, `prettier-plugin-multiline-arrays`, `eslint`, `eslint-config-prettier`, `eslint-plugin-prettier`, `angular-eslint` e `typescript-eslint` (além de `husky` e `lint-staged` caso habilitados) estejam presentes no **devDependencies**.

## 5. Configurações do VS Code

Para uma experiência de desenvolvimento consistente, é recomendado o uso do **VS Code** por todos os desenvolvedores. Para seguir essa padronização **é obrigatório** ter as seguintes extensões instaladas:

- **ESLint (Microsoft)**: Essencial para a padronização e qualidade do nosso código.
- **Prettier - Code formatter (prettier.io)**: Garante a formatação automática e consistente do código.

E devem ser versionadas as configurações e extensões recomendadas para o VS Code em cada projeto.

### .vscode/extensions.json - Sugere extensões essenciais para os desenvolvedores do projeto
```json
{
  "recommendations": [
    "esbenp.prettier-vscode", // O formatador Prettier
    "dbaeumer.vscode-eslint", // O linter ESLint
    "eamodio.gitlens", // Melhora a integração com o Git
    "streetsidesoftware.code-spell-checker", // Verificador ortográfico para código e comentários
    "streetsidesoftware.code-spell-checker-portuguese-brazilian" // Dicionário PT-BR para o anterior
  ],
  "unwantedRecommendations": [
    // --- CONFLITO DE FORMATAÇÃO ---
    // Usamos Prettier, então o Beautify não deve ser usado aqui para evitar formatação dupla ou conflitante.
    "HookyQR.beautify"
  ]
}
```

### .vscode/settings.json - Define configurações do editor específicas para o projeto.

Fica na pasta *.vscode* dentro de cada projeto. EVITE ALTERAR ESSE ARQUIVO (se alterar, não commitar).

```json
{
  // --- CONFIGURAÇÕES DO WORKSPACE DO PROJETO ---
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "prettier.requireConfig": true,
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit",
    "source.organizeImports": "explicit"
  },
  // --- FORMATAÇÃO ESPECÍFICA POR LINGUAGEM ---
 // Garante que o Prettier seja usado para todas as linguagens que ele suporta,
 // mesmo que um usuário tenha outro formatador padrão configurado para JSON ou CSS, por exemplo.
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[jsonc]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[markdown]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[css]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[scss]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  // --- OTIMIZAÇÃO DE DESEMPENHO E FOCO ---
 // Exclui pastas e arquivos desnecessários da árvore de arquivos do VS Code.
 // Isso melhora a performance e mantém a interface limpa.
  "files.exclude": {
    "**/.git": true,
    "**/.svn": true,
    "**/.hg": true,
    "**/CVS": true,
    "**/.DS_Store": true,
    "**/Thumbs.db": true,
    "node_modules/": true,
    "dist/": true,
    "build/": true,
    ".vscode/": false
  },
 // Exclui as pastas da busca (Ctrl+Shift+F), tornando-a muito mais rápida e relevante.
  "search.exclude": {
    "**/node_modules": true,
    "**/bower_components": true,
    "**/*.code-search": true,
    "dist/": true,
    "build/": true,
    "package-lock.json": true
  }
}
```

### settings.json do usuário (global)

O `settings.json` *do usuário* deve ser colocado nas configurações do VS Code do desenvolvedor. Ele é usado quando o workspace/projeto não possui uma configuração específica.

```json
{
  // --- CONFIGURAÇÕES GERAIS DO EDITOR E FORMATADOR PADRÃO ---
 // Define o Prettier como a PRIMEIRA OPÇÃO de formatador para todos os arquivos.
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true,
  "editor.wordWrap": "on",
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  // --- AÇÕES DE CÓDIGO AO SALVAR ---
 // Executa o ESLint (se o projeto estiver configurado para isso).
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  // --- REGRAS DE FALLBACK PARA O PRETTIER (alinhadas ao .prettierrc) ---
  "prettier.singleQuote": true,
  "prettier.semi": true,
  "prettier.useTabs": false,
  "prettier.tabWidth": 2,
  "prettier.trailingComma": "all",
  "prettier.bracketSpacing": true,
  "prettier.htmlWhitespaceSensitivity": "ignore",
  "prettier.arrowParens": "always",
  "prettier.endOfLine": "lf",
  "prettier.printWidth": 120,
  "prettier.proseWrap": "preserve",
  // --- REGRAS DE FALLBACK PARA OS FORMATADORES NATIVOS DO VS CODE ---
 // Usadas QUANDO a extensão Prettier não está instalada ou falha.
 // Note que NÃO definimos mais o "editor.defaultFormatter" aqui dentro.
 // Para HTML
  "html.format.indentInnerHtml": true,
  "html.format.wrapLineLength": 120,
  "html.format.wrapAttributes": "auto",
 // Para JavaScript
  "javascript.format.semicolons": "insert",
  "javascript.format.insertSpaceAfterCommaDelimiter": true,
  "javascript.format.insertSpaceBeforeAndAfterBinaryOperators": true,
  "javascript.format.insertSpaceAfterOpeningAndBeforeClosingNonemptyBraces": true,
  "javascript.format.placeOpenBraceOnNewLineForFunctions": false,
  "javascript.format.placeOpenBraceOnNewLineForControlBlocks": false,
  "javascript.format.insertSpaceBeforeFunctionParenthesis": false,
  "javascript.format.insertSpaceAfterKeywordsInControlFlowStatements": true,
 // Para TypeScript
  "typescript.format.semicolons": "insert",
  "typescript.format.insertSpaceAfterCommaDelimiter": true,
  "typescript.format.insertSpaceBeforeAndAfterBinaryOperators": true,
  "typescript.format.insertSpaceAfterOpeningAndBeforeClosingNonemptyBraces": true,
  "typescript.format.placeOpenBraceOnNewLineForFunctions": false,
  "typescript.format.placeOpenBraceOnNewLineForControlBlocks": false,
  "typescript.format.insertSpaceBeforeFunctionParenthesis": false,
  "typescript.format.insertSpaceAfterKeywordsInControlFlowStatements": true,
  // --- CONFIGURAÇÕES ADICIONAIS ---
  "javascript.updateImportsOnFileMove.enabled": "always",
  "typescript.updateImportsOnFileMove.enabled": "always"
}
```

### Ignorando Alterações Locais no settings.json

O arquivo *.vscode/settings.json* é versionado para padronizar o ambiente. Se um desenvolvedor precisar de uma configuração local específica que não deve ser compartilhada, ele deve usar preferencialmente a configuração do usuário, ou pode instruir o Git a ignorar as alterações locais neste arquivo com o comando:

```bash
git update-index --assume-unchanged .vscode/settings.json
```

> Para voltar a rastrear alterações, use:

```bash
git update-index --no-assume-unchanged .vscode/settings.json
```
