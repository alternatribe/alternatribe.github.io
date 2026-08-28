# SIGNALS

**Signals** representam a nova abordagem do framework para a reatividade, oferecendo uma forma mais simples e performática de gerenciar o estado reativo local em nossos componentes. Deve-se adotar Signals como a principal ferramenta para lidar com dados que mudam ao longo do tempo. **OBRIGATÓRIO A PARTIR DO ANGULAR v21**

---

## 1. Conceitos Fundamentais dos Signals

Signals são valores que podem ser lidos e escritos, e que notificam o Angular de forma otimizada quando o seu valor muda. Diferente das propriedades de classe normais, que exigem verificações manuais ou do ciclo de vida, Signals garantem que apenas os elementos da tela que dependem desse valor sejam atualizados.

Os principais pilares dos Signals são:

- **signal()**: Um valor reativo que pode ser lido chamando-o como função (`meuSignal()`) e escrito através de `.set()` (para substituir o valor diretamente) ou `.update()` (para atualizar baseado no valor anterior).

- **computed()**: Uma função que retorna um valor derivado de outros Signals. Ela se **re-executa automaticamente** apenas quando um de seus Signals de **dependência é alterado**. É somente leitura, memoizada e avaliada sob demanda (*lazy*).

- **effect()**: Uma função para executar efeitos colaterais (como logs no console, sincronização com APIs externas, `localStorage` ou manipulações manuais do DOM) em resposta a mudanças em um ou mais Signals.

- **linkedSignal()**: Introduzido no Angular 19, é um sinal gravável cujo valor padrão é recalculado automaticamente a partir de um sinal de origem (como um `input`), mas que ainda permite alterações manuais locais (via `.set()` ou `.update()`), resetando-se quando a fonte original for alterada.

### Exemplo: Estado e Reatividade Derivada

Este componente demonstra os pilares em ação. Campos de input atualizam *Signals*, que por sua vez acionam funções *`computed`* e *`effect`* para reagir às mudanças:

```typescript
import { Component, signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-user-profile',
  template: `
    <input 
      type="text" 
      [value]="nome()" 
      (input)="atualizarNome($event)" 
      placeholder="Nome" 
    />
    <input 
      type="text" 
      [value]="sobrenome()" 
      (input)="atualizarSobrenome($event)" 
      placeholder="Sobrenome" 
    />
    
    <p>{{ saudacao() }}</p>
    <p>Nome completo: {{ nomeCompleto() }}</p>
  `,
})
export class UserProfileComponent {
  nome = signal('João');
  sobrenome = signal('Silva');

  // Sinal computado que rastreia 'nome' e 'sobrenome'
  nomeCompleto = computed(() => `${this.nome()} ${this.sobrenome()}`);

  // Sinal computado que rastreia APENAS o 'nome'
  saudacao = computed(() => {
    const nomeAtual = this.nome();
    return nomeAtual.length > 0 ? `Olá, ${nomeAtual}!` : 'Digite um nome!';
  });

  constructor() {
    // Este effect rastreia APENAS o signal 'nome'
    effect(() => {
      console.log('>>> Nome alterado:', this.nome());
    });

    // Este outro effect rastreia APENAS o signal 'sobrenome'
    effect(() => {
      console.log('>>> Sobrenome alterado:', this.sobrenome());
    });

    // Este effect rastreia ambos os signals (via nomeCompleto())
    effect(() => {
      console.log('>>> Nome completo alterado:', this.nomeCompleto());
    });
  }

  atualizarNome(event: Event) {
    const input = event.target as HTMLInputElement;
    this.nome.set(input.value);
  }

  atualizarSobrenome(event: Event) {
    const input = event.target as HTMLInputElement;
    this.sobrenome.set(input.value);
  }
}
```

### Opções Avançadas do `signal()` e `effect()`

- **Função de Comparação Customizada (`equal`)**:
  Por padrão, o Signal utiliza igualdade estrita (`===`). É possível customizar a verificação para objetos ou coleções:
  ```typescript
  const usuario = signal({ id: 1, nome: 'Ana' }, {
    equal: (a, b) => a.id === b.id && a.nome === b.nome
  });
  ```

- **Limpeza com `onCleanup` no `effect()`**:
  Permite cancelar timers ou tarefas pendentes antes da próxima execução do efeito:
  ```typescript
  effect((onCleanup) => {
    const termo = this.termoBusca();
    const timer = setTimeout(() => console.log('Buscando:', termo), 500);

    onCleanup(() => clearTimeout(timer));
  });
  ```

---

## 2. Comunicação entre Componentes com Signals

Nas versões mais recentes do Angular, as diretivas e decoradores tradicionais (`@Input`, `@Output`, `@ViewChild`) foram substituídos por funções reativas baseadas em Signals:

- **input() / input.required()**: Substitui o `@Input()`. Retorna um Signal somente leitura com o valor recebido do componente pai. Suporta funções `transform` para tipagem ou conversão.
- **output()**: Substitui o `@Output()` e `EventEmitter` para emissão padronizada de eventos.
- **model()**: Cria uma propriedade bidirecional (*two-way binding*), permitindo o uso direto da sintaxe `[(propriedade)]` sem a necessidade de declarar `@Input` e `@Output` separados.
- **viewChild() / contentChild()**: Substitui `@ViewChild` e `@ContentChild`, retornando uma referência reativa para elementos do template ou componentes filhos.

### Exemplo: Componente Filho com Inputs, Outputs e Two-Way Binding

```typescript
import { Component, input, output, model } from '@angular/core';

@Component({
  selector: 'app-contador-item',
  template: `
    <div>
      <h4>{{ titulo() }}</h4>
      <p>Quantidade: {{ quantidade() }}</p>
      
      <button (click)="incrementar()">+1</button>
      <button (click)="remover.emit(titulo())">Remover Item</button>
    </div>
  `,
})
export class ContadorItemComponent {
  // Input obrigatório recebido do pai
  titulo = input.required<string>();

  // Two-way binding reativo (lido e alterado pelo filho e pelo pai via [(quantidade)])
  quantidade = model<number>(1);

  // Emissão de evento tipado para o componente pai
  remover = output<string>();

  incrementar() {
    this.quantidade.update(q => q + 1);
  }
}
```

**Consumo no componente pai:**
```html
<app-contador-item 
  [titulo]="'Caderno Universitário'" 
  [(quantidade)]="quantidadeItens" 
  (remover)="aoRemoverItem($event)" 
/>
```

---

## 3. Requisições Assíncronas com a Resource API (Angular 20+)

A partir do **Angular 20** (estabilizada após a introdução experimental na v19), a **Resource API** (`resource()` e `rxResource()`) tornou-se a abordagem padrão recomendada para gerenciar operações assíncronas e requisições HTTP integradas nativamente a Signals, substituindo o boilerplate de `toObservable()` + `switchMap` + `toSignal()`.

Uma *Resource* expõe sinais automáticos para controle de estado:
- `value()`: O dado retornado pela requisição.
- `isLoading()`: Sinal booleano indicando se a requisição está em andamento.
- `error()`: Sinal contendo o erro caso a requisição falhe.
- `status()`: O estado atual da requisição (`'idle'`, `'loading'`, `'resolved'`, `'error'`).
- `reload()`: Método para forçar o recarregamento dos dados.

### Exemplo: Busca de Usuários com `rxResource()`

A requisição é reexecutada automaticamente sempre que o Signal `termoBusca()` for alterado:

```typescript
import { Component, signal, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { rxResource } from '@angular/core/rxjs-interop';

interface UsuarioGitHub {
  id: number;
  login: string;
  avatar_url: string;
}

@Component({
  selector: 'app-search-users',
  template: `
    <input 
      type="text" 
      [value]="termoBusca()" 
      (input)="atualizarTermo($event)" 
      placeholder="Buscar usuários no GitHub..."
    />

    @if (usuarios.isLoading()) {
      <p>Carregando resultados...</p>
    } @else if (usuarios.error()) {
      <p class="erro">Ocorreu um erro ao buscar os usuários.</p>
    } @else {
      <ul>
        @for (item of usuarios.value()?.items; track item.id) {
          <li>
            <img [src]="item.avatar_url" width="28" height="28" />
            {{ item.login }}
          </li>
        } @empty {
          <li>Nenhum resultado encontrado.</li>
        }
      </ul>
    }
  `,
})
export class SearchUsersComponent {
  private http = inject(HttpClient);
  
  termoBusca = signal('angular');

  // Recarrega automaticamente sempre que termoBusca() mudar
  usuarios = rxResource({
    request: () => ({ query: this.termoBusca() }),
    loader: ({ request }) => 
      this.http.get<{ items: UsuarioGitHub[] }>(
        `https://api.github.com/search/users?q=${encodeURIComponent(request.query)}`
      ),
  });

  atualizarTermo(event: Event) {
    const input = event.target as HTMLInputElement;
    this.termoBusca.set(input.value);
  }
}
```

---

## 4. Integração com o Ecossistema Angular e RxJS

Signals não substituem o RxJS, mas trabalham em conjunto. Signals são ideais para o estado local e reatividade de interface, enquanto RxJS continua sendo a escolha para fluxos de dados assíncronos contínuos e orquestrações complexas.

As funções `toObservable()` e `toSignal()` do pacote `@angular/core/rxjs-interop` fazem a ponte entre os dois mundos.

### Exemplo: Chamada HTTP Reativa com RxJS

Quando se deseja aplicar operadores avançados de fluxo como `debounceTime` ou `distinctUntilChanged`, a conversão entre Signals e Observables é direta:

```typescript
import { Component, signal, inject } from '@angular/core';
import { toObservable, toSignal } from '@angular/core/rxjs-interop';
import { HttpClient } from '@angular/common/http';
import { debounceTime, distinctUntilChanged, switchMap } from 'rxjs';

interface Usuario {
  id: number;
  name: string;
}

@Component({
  selector: 'app-search-users-stream',
  template: `
    <input 
      type="text" 
      [value]="termoBusca()" 
      (input)="atualizarTermo($event)" 
      placeholder="Buscar usuários..."
    />

    <ul>
      @for (item of resultados(); track item.id) {
        <li>{{ item.name }}</li>
      } @empty {
        <li>Nenhum resultado encontrado.</li>
      }
    </ul>
  `,
})
export class SearchUsersStreamComponent {
  private http = inject(HttpClient);
  
  termoBusca = signal('');
  
  // Converte o sinal em stream, aplica operadores e reconverte em sinal
  resultados = toSignal(
    toObservable(this.termoBusca).pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(termo => 
        this.http.get<Usuario[]>(`https://api.github.com/users?q=${termo}`)
      )
    ),
    { initialValue: [] }
  );

  atualizarTermo(event: Event) {
    const input = event.target as HTMLInputElement;
    this.termoBusca.set(input.value);
  }
}
```

### Signals vs. Observables: Quando Usar Cada Um

Essa é uma das dúvidas mais comuns. *Signals* e *Observables* não são concorrentes, mas sim ferramentas complementares. A escolha entre eles depende da natureza do dado que você está manipulando:

- **Signals são a melhor escolha para...**
  - **Estado Local e Síncrono**: Use para gerenciar o estado dentro de um componente (ex: `isLoading`, `isToggled`, valor de um campo de input). Eles representam um único valor atual, pronto para uso imediato.
  - **Reatividade de UI**: São perfeitos para ligar dados diretamente ao template e garantir que a interface do usuário seja reativa de forma granular, sem a necessidade do `async` pipe.
  - **Comunicação entre Componentes**: Use `input()`, `output()` e `model()` para fluxo de dados limpo e tipado.

- **Observables são a melhor escolha para...**
  - **Fluxos de Dados Assíncronos Contínuos**: Use para eventos que ocorrem ao longo do tempo (ex: `WebSockets`, Server-Sent Events, fluxos de mensagens, eventos de clique contínuos).
  - **Orquestração e Transformação Complexa**: Use quando precisar de operadores como `switchMap`, `debounceTime`, `retry`, `catchError` ou `combineLatest` para tratar múltiplos fluxos de dados concorrentes.

---

## 5. Formulários com Signals: Signal Forms e Reactive Forms

O Angular evoluiu a arquitetura de formulários para eliminar a necessidade de subscrições manuais (`valueChanges.subscribe()`) e garantir sincronização direta com a reatividade por Signals.

### 5.1. Signal Forms (Angular v21+)

A partir do **Angular 21**, a arquitetura **Signals-First** disponibiliza os **Signal Forms**, onde formulários e campos são nativamente Signals:
- Valores de campos são lidos diretamente como Signals (`campo.value()`).
- O status de validação (`valid()`, `dirty()`, `touched()`, `errors()`) é computado reativamente como Signal.
- Dispensa o uso de `takeUntilDestroyed()` ou subscrições com `valueChanges` para reações em tempo real na interface.

### 5.2. Integração com `ReactiveFormsModule`

Para aplicações que utilizam formulários reativos tradicionais (`FormGroup`, `FormControl`), a forma recomendada de sincronizar o estado de validação com Signals é através do `toSignal()` conectado a `statusChanges` ou `valueChanges`:

> **Atenção**: Propriedades como `this.signupForm.valid` não são Signals. Escrever `computed(() => this.signupForm.valid)` **NÃO** reagirá em tempo real às alterações do usuário. É obrigatório converter os eventos com `toSignal()`.

```typescript
import { Component, computed, inject } from '@angular/core';
import { FormBuilder, Validators, ReactiveFormsModule } from '@angular/forms';
import { toSignal } from '@angular/core/rxjs-interop';
import { startWith } from 'rxjs';

@Component({
  selector: 'app-signup',
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="signupForm" (ngSubmit)="onSubmit()">
      <label>Email:</label>
      <input type="email" formControlName="email" />
      
      <label>Senha:</label>
      <input type="password" formControlName="password" />
      
      <button type="submit" [disabled]="!isFormValid()">
        Cadastrar
      </button>
    </form>
  `,
})
export class SignupComponent {
  private fb = inject(FormBuilder);
  
  // Criação do formulário com ReactiveForms
  signupForm = this.fb.group({
    email: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(6)]],
  });
  
  // Converte as alterações de status do formulário em um Signal reativo
  private formStatus = toSignal(
    this.signupForm.statusChanges.pipe(
      startWith(this.signupForm.status)
    ),
    { initialValue: this.signupForm.status }
  );

  // Sinal computado que reage automaticamente ao Signal formStatus
  isFormValid = computed(() => this.formStatus() === 'VALID');

  onSubmit() {
    if (this.isFormValid()) {
      console.log('Formulário válido. Dados:', this.signupForm.value);
    }
  }
}
```

---

## 6. Arquitetura Signals-First e Aplicações Zoneless (Angular 21+)

A partir do **Angular v21**, a detecção de mudanças puramente baseada em Signals consolida o ecossistema **Zoneless** (sem dependência da biblioteca `zone.js`):

- **Performance Superior e Granular**: O framework não intercepta todos os eventos assíncronos do navegador (como `setTimeout` ou `addEventListener`). A atualização ocorre cirurgicamente nos nós do DOM associados aos Signals alterados.
- **Menor Tamanho de Pacote**: Elimina o carregamento e overhead do `zone.js`.
- **Rastreabilidade e Previsibilidade**: O fluxo de dados torna-se totalmente explícito e transparente, facilitando depuração e testes unitários.

---

## 7. Como Migrar para Signals

- **Comece pelo Estado Local**: Identifique propriedades que guardam o estado interno do componente (ex: `isLoading: boolean = false;`) e as substitua por Signals (`isLoading = signal(false);`).

- **Utilize `computed()` para Valores Derivados**: Substitua getters de classe e lógica de `ngOnChanges` por funções `computed()` ou `linkedSignal()`.

- **Adote `input()`, `output()` e `model()`**: Migre os decoradores legados `@Input()`, `@Output()` e `@ViewChild()` para as novas funções de sinal correspondentes.

- **Utilize a Resource API para Dados Assíncronos**: Prefira `rxResource()` ou `resource()` para requisições HTTP e carregamento de dados em vez de pipelines manuais.

- **Atualize os Templates para Control Flow**: Substitua diretivas estruturais legadas (`*ngIf`, `*ngFor`) pela nova sintaxe `@if` e `@for (...; track ...)`.

- **Integre RxJS com Sabedoria**: Use `toObservable()` para transformar Signals em streams quando precisar de operadores temporais/debounce e `toSignal()` para consumir Observables diretamente no template sem o pipe `async`.

A transição para Signals é um passo natural para um código mais limpo, eficiente e fácil de manter. Comece substituindo o estado local de componentes com `signal()`, use `computed()` para valores derivados, e integre o RxJS para fluxos de dados mais complexos ou assíncronos. A combinação de Signals para o estado, a Resource API para assincronismo e a nova arquitetura reativa é a forma recomendada de construir aplicações modernas e performáticas com Angular.
