# Angular Renaissance

Com o Angular v14 começou uma reestruturação no angular em direção ao Angular Moderno com o lançamento dos Standalone Components (que são componentes que não dependem de módulos) em um movimento para remover futuramente as dependências da biblioteca zone.js (que é a biblioteca responsável pelo gerenciamento do ciclo de vida do angular / detecção de mudanças). No Angular 16 foi introduzido o Signals para tornar a programação reativa sem depender dessa biblioteca (zone.js). E logo após foi cunhado o termo Angular Renaissance, ou seja, o renascimento do Angular, também conhecido como Angular Moderno.

## Principais Novidades

### Angular v14  
- Standalone Components - componentes que podem ser usados sem precisar de um módulo, simplificando a estrutura e tornando-os mais modular.  
- Typed Forms - tipagem para formulários reativos, garantindo que os dados dos formulários sejam validados e manipulados de maneira mais precisa e semântica, usando tipos de dados do TypeScript.  
- Inject() - injetar dependência sem precisar do construtor.

### Angular v15  
- Autonomous Routes - as rotas autônomas não precisam de um módulo central para definir as rotas, elas podem ser definidas dentro dos componentes standalones.  
- Functional Router Guards - uma forma de simplificar a configuração e implementação de guardas de roteamento, implementando como funções em vez de classes.  
- Directive Composition - uma maneira poderosa de combinar múltiplas diretivas em uma só, criando componentes e funcionalidades mais modulares e reutilizáveis.  
- ngSrc - diretiva de imagem que melhora a performance.  
- melhorias no Standalone Components.  
- recomendação de criar Guards, Resolvers e Interceptors em forma de função.

### Angular v16  
- Signals - é um novo mecanismo reativo para gerenciar o estado de forma direta, sendo uma maneira mais fácil e eficiente de gerenciar e rastrear mudanças de estado, com maior controle sobre quando e como as alterações no estado disparam atualizações na interface do usuário, alternativa ao gerenciamento do ciclo de vida tradicional (detecção de mudanças). Pode ser convertido para Observables e vice-versa.  
- melhorias no Content Projection - permite que o conteúdo (geralmente passado como HTML ou outro conteúdo de template) seja inserido dentro de um componente de maneira dinâmica e reutilizável, sem que o componente precise saber sobre o conteúdo real, ou seja, o conteúdo pode ser definido externamente ao componente, permitindo que você separe a estrutura e a lógica do componente da apresentação de conteúdo.  
- @Input(\{ required: true, alias: 'nomeCampoNaTela' \}) torna o input required, verificando antes da execução do campo, e alias torna a mensagem de erro mais legível exibindo o *alias* na mensagem.  
- Parâmetros de entrada nas rotas - pode ser configurado em Routes ou via @Input no componente  
   ex: https://meu.app.com/user/1?search=busca  
   @Input() userId: string; // 1  
   @Input() search: string; // busca  
- Suporte oficial a teste unitários com Jest, em substituição ao Karma.

### Angular v17  
- Control flow - Nova syntax dos templates, substituindo as diretivas \*ngIf, \*ngFor e \*ngSwitch.  
- Deferrable views @defer - pedaço do template como lazy load, ou componente, diretiva, pipe ou css, que só carrega na tela se satisfazer a condição, por exemplo quando terminar de carregar uma imagem ou em ponto visível da tela.  
- ~~*novidades no SSR e Hidration* - Processamento no lado do servidor, sendo mais utilizado para páginas estáticas por melhorar a indexação por motores de busca ou o conteúdo é personalizado no lado do servidor.~~  
- melhorias no Signals  
- NgModules serão mantidos por enquanto, mas a equipe do Angular recomenda a migração gradual para Standalone Components.

### Angular v18  
- Observables em eventos do AbstractControl  
- ng-content com conteúdo padrão caso ele não seja passado.  
- RedirectCommand() - novo comando para redirecionamento de rotas com opções adicionais que não existiam antes.  
- redirecTo() - flexibilidade para retornar função ou UrlTree, em vez de string.

### Angular v19  
- Standalone por padrão - components, pipes e directives serão standalone caso não seja informado para não ser.  
- ~~*resource API* (experimental ainda)~~  
- Variáveis locais em templates: @let greeting = 'Olá ' + name.value;  
- melhorias no Signals;  
- Anúncio que o Karma será descontinuado em 2025.

### Angular v20
- Integração de WebAssembly (Wasm) - A versão 20 foca na integração de WebAssembly (Wasm) para melhorar o desempenho de processamentos complexos. A ideia é permitir que partes críticas da aplicação sejam executadas em Wasm para alcançar velocidades quase nativas, especialmente em cálculos intensivos ou processamento de imagens.
- Melhorias na performance com Signals e Change Detection - O desenvolvimento de componentes totalmente baseados em signals continua a evoluir, com o objetivo de otimizar a detecção de mudanças e eliminar a necessidade do NgZone. Isso tornará as aplicações mais rápidas, eficientes e fáceis de depurar.
- Resource API Estável - A Resource API, introduzida como experimental na versão 19, foi estabilizada, tornando-se a abordagem recomendada para a busca de dados.
- Ferramentas de Migração e Codemods - A equipe do Angular disponibilizou mais ferramentas automatizadas (codemods) para facilitar a migração de aplicações existentes para a nova sintaxe de Control flow e componentes baseados em Signals, tornando o processo de atualização menos manual e mais seguro. (ferramentas usadas na atualização da versão 19 para a 20).
- Suporte a Novas Versões do TypeScript e ECMAScript - O Angular 20 traz suporte aprimorado para as últimas versões do TypeScript e das especificações do ECMAScript, garantindo que os desenvolvedores possam utilizar as novidades da linguagem.

### Angular v21 - Arquitetura "Signals-First" e AI-Tooling
- Zoneless por Padrão: O `zone.js` passa a ser excluído por padrão em novos projetos (`ng new`), consolidando a detecção de mudanças puramente baseada em **Signals** para maior performance e pacotes menores.
- Signal Forms (Developer Preview): Introdução de uma API totalmente nova e reativa para formulários baseada em *Signals*, eliminando as subscrições manuais via RxJS (`valueChanges`) e o padrão `takeUntil`.
- Angular Aria (Developer Preview): Lançamento de um pacote headless focado em acessibilidade, trazendo padrões de UI e componentes nativos otimizados para leitores de tela.
- Vitest como Test Runner Padrão: O Vitest assume o lugar do Karma em definitivo para novas aplicações, garantindo testes muito mais rápidos e execução nativa em ambientes *zoneless*.
- Angular MCP Server: Integração de ferramentas de Inteligência Artificial nativas via **Model Context Protocol**, permitindo refatorações, migrações e geração de código direto pelo CLI através de prompts em linguagem natural. 
- HttpClient Global: O `HttpClient` passa a ser injetado nativamente no nível raiz, dispensando configurações manuais repetitivas no início do projeto. 

### Angular v22
- Estabilização do Signal Forms & Angular Aria: As APIs de formulários baseadas em **Signals** e os componentes acessíveis headless tornam-se **100% estáveis e recomendados para produção**.
- OnPush por Padrão: A estratégia de detecção de mudanças `OnPush` passa a ser o padrão do ecossistema do framework, forçando uma renderização mais econômica e performática.
- Asynchronous Reactivity APIs Estáveis: Consolidação das diretivas de busca assíncrona como `resource()` e `httpResource()` para o gerenciamento nativo de requisições via *Signals*.
- Novo Decorador @Service e injectAsync: Introdução do decorador `@Service` focado nas novas estruturas modernas e da função `injectAsync()` para carregamento sob demanda (lazy loading) de injeções de dependência assíncronas.
- Melhorias em Templates: Suporte a comentários em formato HTML, operadores de espalhamento (*spread syntax*), checagem exaustiva dentro do bloco `@switch` e o uso de **arrow functions** direto nas expressões do template.
- Fim do Suporte ao Webpack: O framework descontinua oficialmente o suporte ao Webpack no build padrão, movendo os holofotes completamente para o motor de compilação ultra-rápido baseado em Go (tsgo) e esbuild. 
