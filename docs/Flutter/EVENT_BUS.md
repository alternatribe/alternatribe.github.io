# Guia Arquitetural: Event Bus (Domain Events) — Flutter/Dart

> **Propósito deste documento:** Servir como referência arquitetural autossuficiente sobre o uso correto do padrão Event Bus (Domain Events) em projetos Flutter/Dart que adotam arquitetura em camadas (MVVM, MVC, Clean Architecture, Hexagonal, etc.). Adapte os exemplos de estrutura de pastas e injeção de dependência à convenção do seu projeto.

---

## 1. Definição e Escopo

O Event Bus é um **mediador interno** usado exclusivamente para orquestrar **efeitos colaterais entre agregados de domínio diferentes** após uma operação de escrita bem-sucedida. Ele implementa o padrão **Domain Events** do Domain-Driven Design (DDD).

### 1.1. O Que o Event Bus É

- Um canal de comunicação **unidirecional** entre domínios de negócio.
- Restrito à **camada de aplicação/orquestração** (`application/listeners/`), invisível para a UI.
- O mecanismo que garante o **Open/Closed Principle** entre domínios: adicionar um novo efeito colateral significa adicionar um novo `Listener`, sem modificar o código que publica o evento.

### 1.2. O Que o Event Bus NÃO É

- **Não é** um barramento de mensagens entre Widgets e Repositories.
- **Não é** um mecanismo de atualização reativa de telas (para isso, use `ChangeNotifier`, `ValueNotifier`, `Signals` ou `Stream`).
- **Não é** um substituto para Streams do Drift/Isar, `ValueListenable`, `Signal` ou qualquer padrão reativo de UI.
- **Não é** um sistema de pub/sub genérico para qualquer comunicação.
- **Não é** um mecanismo de request-response (como `Bloc` events ou `MediatR`).

---

## 2. O Problema Real Que o Event Bus Resolve

### 2.1. Cenário Concreto

Considere um sistema onde, ao finalizar uma venda, as seguintes ações precisam acontecer:

1. **Baixar estoque** dos produtos vendidos (Domínio: Estoque).
2. **Registrar movimentação financeira** (Domínio: Financeiro).
3. **Imprimir comprovante** (Domínio: Impressão).
4. **Notificar painel de produção** (Domínio: Produção).
5. **Atualizar métricas** de desempenho (Domínio: BI/Relatórios).

Nenhuma dessas ações faz parte da **responsabilidade primária** de "finalizar uma venda". São efeitos colaterais em domínios alheios.

### 2.2. Sem Event Bus — Acoplamento Direto

```dart
class VendaRepository {
  // 5 dependências de outros domínios:
  final EstoqueRepository _estoqueRepo;
  final FinanceiroRepository _financeiroRepo;
  final ImpressaoService _impressaoService;
  final ProducaoService _producaoService;
  final RelatorioRepository _relatorioRepo;

  AsyncResult<Venda> finalizarVenda(Venda venda) async {
    final result = await _vendaService.save(_mapper.toDto(venda));

    // VendaRepository CONHECE todos os domínios afetados:
    await _estoqueRepo.darBaixa(venda.itens);          // ❌ acoplamento
    await _financeiroRepo.registrarReceita(venda);      // ❌ acoplamento
    await _impressaoService.imprimirCupom(venda);       // ❌ acoplamento
    await _producaoService.removerPedido(venda.id);     // ❌ acoplamento
    await _relatorioRepo.atualizarMetricas(venda);      // ❌ acoplamento

    return Success(venda);
  }
}
```

**Problemas:**
- `VendaRepository` acumula **N dependências** de outros domínios.
- Cada novo efeito colateral exige **modificar** `VendaRepository` — viola Open/Closed.
- Se `ImpressaoService` falhar, a venda inteira pode falhar — mesmo que impressão não seja crítica.
- Impossível testar `VendaRepository` isoladamente sem mocar 5 dependências com `mocktail`.
- Viola **Single Responsibility**: o repositório de Venda sabe sobre estoque, finanças, impressão, produção e relatórios.

### 2.3. Com Event Bus — Desacoplamento

```dart
// ── PUBLICADOR (não conhece nenhum consumidor) ────────────────
class VendaRepository {
  final VendaService _vendaService;
  final VendaMapper _mapper;
  final EventBus _eventBus;

  AsyncResult<Venda> finalizarVenda(Venda venda) async {
    final result = await _vendaService.save(_mapper.toDto(venda));

    // Publica o FATO e vai embora. Não sabe quem reage.
    _eventBus.publish(VendaFinalizadaEvent(
      vendaId: venda.id,
      itensVendidos: venda.itens,  // snapshot completo
      valorTotal: venda.total,
      operadorId: venda.operadorId,
      dataFinalizacao: DateTime.now(),
    ));

    return Success(venda);
  }
}

// ── CONSUMIDORES (cada um isolado em seu Listener) ────────────
class EstoqueListener {
  final ProdutoRepository _produtoRepo;

  EstoqueListener(this._produtoRepo);

  void onVendaFinalizada(VendaFinalizadaEvent event) {
    _produtoRepo.darBaixaDeEstoquePorVenda(event.itensVendidos);
  }
}

class FinanceiroListener {
  final FinanceiroRepository _financeiroRepo;

  FinanceiroListener(this._financeiroRepo);

  void onVendaFinalizada(VendaFinalizadaEvent event) {
    _financeiroRepo.registrarReceitaDeVenda(
      event.vendaId,
      event.valorTotal,
    );
  }
}

class ImpressaoListener {
  final ImpressaoService _impressaoService;

  ImpressaoListener(this._impressaoService);

  void onVendaFinalizada(VendaFinalizadaEvent event) {
    _impressaoService.imprimirCupomDeVenda(event.vendaId);
  }
}
```

**Resultado:**
- `VendaRepository` tem **zero dependências** de outros domínios.
- Novo efeito colateral = novo `Listener`. Nenhum código existente muda.
- Falha na impressão não afeta a venda nem o estoque.
- Cada Listener é testável isoladamente com `mocktail`.
- Cada domínio evolui de forma independente.

---

## 3. Por Que Streams / Signals / ValueNotifier NÃO Substituem o Event Bus

### 3.1. A Diferença Semântica Fundamental

| Conceito | Semântica | Pergunta que responde | Natureza |
|----------|-----------|----------------------|----------|
| **Stream / Signal / ValueNotifier** | Estado atual dos dados | *"Como estão os dados agora?"* | Contínuo — deduplica, último valor vence |
| **Evento de Domínio** | Fato de negócio que ocorreu | *"O que acabou de acontecer?"* | Discreto — cada ocorrência é única e deve ser processada |

### 3.2. Prova por Contradição

Tentando usar `Stream` do Drift para efeitos colaterais entre domínios:

```dart
// ❌ TENTATIVA: ProdutoRepository "observa" vendas via Stream reativo
class ProdutoRepository {
  ProdutoRepository(VendaRepository vendaRepo) {
    vendaRepo.watchVendas().listen((List<Venda> vendas) {
      // PROBLEMA 1 — Qual venda é nova?
      // O .watch() do Drift emite a lista INTEIRA de vendas.
      // Preciso manter cache do estado anterior e fazer diff.

      // PROBLEMA 2 — Perda de eventos
      // Se duas vendas forem finalizadas em rápida sequência,
      // o Drift pode coalescer as notificações e emitir apenas
      // UMA atualização com ambas já incluídas.
      // Uma das vendas não é "vista" como nova pelo diff.

      // PROBLEMA 3 — Reprocessamento no restart
      // Se o app reiniciar, o .watch() emite todas as vendas existentes.
      // Vou dar baixa no estoque de vendas históricas?
      // Preciso de um marcador "já processado" → reinventei um event log.

      // PROBLEMA 4 — Acoplamento
      // ProdutoRepository agora DEPENDE de VendaRepository.
      // Se FinanceiroRepository também precisar reagir,
      // também vai depender de VendaRepository. Todos dependem de todos.

      // PROBLEMA 5 — Sem intenção
      // A stream diz O QUE EXISTE, não O QUE ACONTECEU.
      // Venda cancelada e venda nova aparecem igual na stream.
      // Não tenho a INTENÇÃO: foi criação? atualização? cancelamento?
    });
  }
}
```

### 3.3. Analogia Operacional

- **Event Bus** = **Comanda de restaurante**: cada pedido é uma comanda individual. Mesmo que cheguem 10 pedidos em 1 minuto, todos são processados na ordem correta.
- **Stream/Signal** = **Quadro branco**: você escreve o pedido atual. Se chegar um novo antes do cozinheiro ler, o anterior é apagado.

### 3.4. Resumo Visual: Cada Ferramenta Para Seu Problema

```
┌──────────────────────────────────────────────────────────────┐
│                    FLUXO DE LEITURA (UI)                     │
│                                                              │
│  Widget ← ChangeNotifier ← Controller ← Repository          │
│                                              ↑               │
│                                     Drift .watch() / Stream  │
│                                                              │
│  Ferramenta: Stream / Signal / ChangeNotifier / ValueNotifier│
│  Propósito: A UI reflete o ESTADO ATUAL dos dados            │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                FLUXO DE ESCRITA (Backend/Domínio)            │
│                                                              │
│  Widget → Controller → Repository → Service (Drift/API)     │
│                                         ↓                    │
│                                     EventBus                 │
│                                         ↓                    │
│                                Listener (application/)       │
│                                         ↓                    │
│                          Outro Repository / UseCase          │
│                                                              │
│  Ferramenta: Event Bus (Domain Events)                       │
│  Propósito: Domínios reagem a FATOS DE NEGÓCIO              │
└──────────────────────────────────────────────────────────────┘
```

> **Regra de Ouro:** A UI (Widgets, Pages, Controllers) NUNCA escuta o Event Bus. O Event Bus NUNCA é usado para leitura.

---

## 4. Regras de Uso

### 4.1. Quem PUBLICA Eventos

**Exclusivamente os Repositories**, após a persistência bem-sucedida de uma operação de escrita.

```dart
// ✅ CORRETO: Publicar APÓS persistência bem-sucedida
AsyncResult<Venda> finalizarVenda(Venda venda) async {
  final result = await _vendaService.save(_mapper.toDto(venda));
  if (result.isSuccess()) {
    _eventBus.publish(VendaFinalizadaEvent(...)); // só publica se salvou
  }
  return result;
}

// ❌ ERRADO: Publicar ANTES ou independente da persistência
AsyncResult<Venda> finalizarVenda(Venda venda) async {
  _eventBus.publish(VendaFinalizadaEvent(...)); // e se o save falhar?
  final result = await _vendaService.save(_mapper.toDto(venda));
  return result;
}
```

### 4.2. Quem CONSOME Eventos

**Exclusivamente classes `Listener`** na camada `application/listeners/`.

- Listeners são **singletons** registrados no `GetIt` (ou outro container de DI) durante a inicialização da aplicação.
- Não há subscribe/unsubscribe dinâmico — portanto, **não há risco de vazamento de memória**.
- Um `Listener` pode invocar um `Repository` para ações simples (mono-domínio) ou um `UseCase` para ações complexas (multi-domínio).

```dart
// Registro no GetIt (injection_container.dart ou main.dart)
getIt.registerSingleton<EstoqueListener>(
  EstoqueListener(getIt<ProdutoRepository>())
    ..subscribeTo(getIt<EventBus>()),
);

getIt.registerSingleton<FinanceiroListener>(
  FinanceiroListener(getIt<FinanceiroRepository>())
    ..subscribeTo(getIt<EventBus>()),
);
```

### 4.3. Matriz de Permissões

| Camada / Classe | Publicar? | Consumir? | Motivo |
|---|---|---|---|
| **Widget / Page** | ❌ | ❌ | UI usa `ChangeNotifier`/`Signal`/`Stream`, não eventos |
| **Controller / ViewModel** | ❌ | ❌ | Consome dados via Repository, não via eventos |
| **Service / DAO (Drift)** | ❌ | ❌ | Camada técnica pura, sem lógica de negócio |
| **Entity** | ❌ | ❌ | Domínio puro, sem dependências de infra |
| **Repository** | ✅ | ❌ | Publica após escrita bem-sucedida |
| **Listener** | ❌ | ✅ | Único consumidor autorizado |
| **UseCase** | ❌ | ❌ | É invocado pelo Listener, não pelo EventBus diretamente |

### 4.4. Anatomia de um Evento de Domínio

Todo evento deve ser um **snapshot imutável** do fato de negócio, utilizando `Equatable` para comparação e tipos seguros (`BigDecimal` para valores monetários). O consumidor **nunca deve precisar consultar o banco** para complementar os dados do evento.

```dart
// ✅ CORRETO: Evento autossuficiente com dados completos
class VendaFinalizadaEvent extends DomainEvent with EquatableMixin {
  final String vendaId;
  final List<ItemVendaSnapshot> itensVendidos;  // snapshot, não referência
  final BigDecimal valorTotal;
  final String operadorId;
  final DateTime dataFinalizacao;

  const VendaFinalizadaEvent({
    required this.vendaId,
    required this.itensVendidos,
    required this.valorTotal,
    required this.operadorId,
    required this.dataFinalizacao,
  });

  @override
  List<Object> get props => [vendaId, dataFinalizacao];
}

// ❌ ERRADO: Evento que exige consulta adicional
class VendaFinalizadaEvent extends DomainEvent {
  final String vendaId; // só o ID? O listener vai buscar no banco?
  // Isso recria o acoplamento entre domínios que o EventBus deveria eliminar.
}
```

**Motivo:** Se o evento carrega apenas o ID, o `EstoqueListener` precisaria injetar o `VendaRepository` para buscar os itens — criando dependência direta entre domínios, anulando o propósito do Event Bus.

### 4.5. Convenções de Nomenclatura

- **Eventos:** Substantivo + verbo no particípio passado. O nome descreve um **fato consumado**, não uma intenção.
  - ✅ `VendaFinalizadaEvent`, `PedidoCanceladoEvent`, `EstoqueAjustadoEvent`
  - ❌ `FinalizarVendaEvent` (isso é um comando, não um evento)
  - ❌ `VendaEvent` (genérico demais — qual fato?)

- **Listeners:** Nome do domínio que **reage** + sufixo `Listener`.
  - ✅ `EstoqueListener`, `FinanceiroListener`, `NotificacaoListener`
  - ❌ `VendaListener` (ambíguo — o domínio que publicou ou que reagiu?)

- **Arquivos:**
  - Eventos: `lib/domain/events/<nome_evento>.dart`
  - Listeners: `lib/application/listeners/<nome_listener>.dart`
  - Event Bus core: `lib/core/events/event_bus.dart`
  - Classe base: `lib/core/events/domain_event.dart`

---

## 5. Quando NÃO Usar o Event Bus

### 5.1. Não Use Para Atualizar a UI

```dart
// ❌ NUNCA FAÇA ISSO
class ProdutoPage extends StatefulWidget {
  @override
  _ProdutoPageState createState() => _ProdutoPageState();
}

class _ProdutoPageState extends State<ProdutoPage> {
  @override
  void initState() {
    super.initState();
    getIt<EventBus>().on<EstoqueAtualizadoEvent>().listen((event) {
      setState(() => _quantidade = event.novaQuantidade);
      // Causa: vazamento de memória (listen sem cancel no dispose),
      // updates fora de ordem, acoplamento da UI com eventos internos.
    });
  }
}

// ✅ FAÇA ISSO: Use Stream reativo do Drift ou ChangeNotifier
class ProdutoController extends ChangeNotifier {
  Stream<Produto> watchProduto(String id) {
    return _produtoRepository.watchById(id);
    // Drift .watch() emite automaticamente quando os dados mudam no banco.
    // O banco é a fonte da verdade, não o evento.
  }
}
```

### 5.2. Não Use Para Comunicação Dentro do Mesmo Domínio

Se a ação afeta apenas o mesmo agregado, use uma chamada direta de método.

```dart
// ❌ EXAGERO: Evento dentro do próprio domínio
await produtoRepository.save(produto);
eventBus.publish(ProdutoSalvoEvent(produto));
// ProdutoListener → produtoRepository.recalcularPrecoMedio(produto)
// O listener chama o mesmo repository que publicou! Circular e desnecessário.

// ✅ CORRETO: Lógica interna fica no próprio Repository
class ProdutoRepository {
  AsyncResult<Produto> save(Produto produto) async {
    final produtoComPreco = _recalcularPrecoMedio(produto); // lógica interna
    final dto = _mapper.toDto(produtoComPreco);
    return _produtoService.save(dto);
  }
}
```

### 5.3. Não Use Para Consultas (Queries)

```dart
// ❌ NUNCA: Pedir dados via evento
eventBus.publish(SolicitarDadosEstoqueEvent(produtoId));
// Quem responde? Como recebo o retorno? Não é request-response.

// ✅ CORRETO: Consulta direta via Repository
final estoque = await produtoRepository.getEstoquePorId(produtoId);
```

### 5.4. Não Use Para Eventos de Infraestrutura

Eventos como "conexão caiu", "token expirou" ou "erro fatal" são eventos de **infraestrutura**, não de domínio. Eles devem usar um mecanismo de estado global (`Signal`, `ValueNotifier` global), mas não devem poluir o Event Bus de domínio.

```dart
// ❌ ERRADO: Misturar infraestrutura no Event Bus de domínio
eventBus.publish(ConexaoPerdidaEvent());

// ✅ CORRETO: Usar estado global para infraestrutura
// Exemplo com Signal:
final conectividadeSignal = Signal<bool>(true);
// Exemplo com ValueNotifier:
final conectividadeNotifier = ValueNotifier<bool>(true);
```

---

## 6. Tratamento de Falhas nos Listeners

> ⚠️ **O documento de arquitetura do projeto deve definir explicitamente qual estratégia adotar.**

### Estratégia A: Fire-and-Forget (Tolerante a Falhas)

O evento é publicado e o Repository **não aguarda** (`await`) os Listeners terminarem. Se um Listener falhar, o erro é logado, mas a operação principal não é afetada.

```dart
// O publish não é awaited
_eventBus.publish(VendaFinalizadaEvent(...));
return Success(venda); // sucesso independente dos listeners
```

- **Quando usar:** Quando os efeitos colaterais são **compensáveis** ou **não-críticos** (impressão, notificação push, métricas de BI).
- **Risco:** Estado temporariamente inconsistente entre domínios. Exige mecanismo de reconciliação ou retry.

### Estratégia B: Transacional (Consistência Forte)

O evento é publicado e o Repository **aguarda** todos os Listeners concluírem, idealmente dentro de uma `Transaction` do Drift.

```dart
AsyncResult<Venda> finalizarVenda(Venda venda) async {
  return _database.transaction(() async {
    await _vendaService.save(dto);
    final resultados = await _eventBus.publishAndWait(
      VendaFinalizadaEvent(...),
    );
    if (resultados.any((r) => r.isError())) {
      throw RollbackException('Falha em listener crítico');
    }
    return Success(venda);
  });
}
```

- **Quando usar:** Quando os efeitos colaterais são **críticos para a integridade do negócio** (baixa de estoque em sistema financeiro/fiscal).
- **Risco:** Performance reduzida. O Listener mais lento define o tempo total. Complexidade de rollback.

### Estratégia C: Híbrida (Recomendada)

Classificar cada Listener como **crítico** ou **não-crítico**:
- **Críticos** (estoque, financeiro): Executados de forma síncrona dentro da mesma transação do Drift.
- **Não-críticos** (impressão, notificação): Executados de forma assíncrona com `unawaited()`, com retry e logging.

```dart
AsyncResult<Venda> finalizarVenda(Venda venda) async {
  return _database.transaction(() async {
    await _vendaService.save(dto);

    // Listeners críticos — dentro da transação
    await _eventBus.publishCritical(VendaFinalizadaEvent(...));

    return Success(venda);
  });
  // Listeners não-críticos — fora da transação, fire-and-forget
  _eventBus.publishAsync(VendaFinalizadaEvent(...));
}
```

---

## 7. Comparativo: Event Bus vs. Alternativas no Ecossistema Flutter

### 7.1. Event Bus vs. Stream (Drift `.watch()`) / Signal / ValueNotifier

| Critério | Event Bus | Stream / Signal / ValueNotifier |
|----------|-----------|-------------------------------|
| **Propósito** | Reagir a fatos de negócio entre domínios | Observar estado atual dos dados para a UI |
| **Quem usa** | `Listener` (camada `application/`) | `Controller`/`Widget` (camada `ui/`) |
| **Semântica** | *"Isso aconteceu"* (passado, discreto) | *"Esses são os dados agora"* (presente, contínuo) |
| **Duplicatas** | Cada evento é único e processado | Deduplica — emite apenas último estado |
| **Carrega intenção?** | ✅ Sim (criou, cancelou, finalizou) | ❌ Não (apenas dados brutos) |
| **Risco de memory leak** | Nenhum (listeners são singletons no `GetIt`) | Existe se `StreamSubscription` não for cancelada no `dispose()` |

### 7.2. Event Bus vs. Bloc / Cubit

| Critério | Event Bus (Domain Events) | Bloc (Events → States) |
|----------|--------------------------|------------------------|
| **Escopo** | Comunicação entre domínios backend | Gerenciamento de estado de uma feature/página |
| **Direção** | Um-para-muitos (broadcast) | Um-para-um (event → state) |
| **Retorno** | Nenhum (fire-and-forget) | Produz um novo State para a UI |
| **Acoplamento** | Publicador não conhece consumidores | Bloc conhece seus Events e States |
| **Uso ideal** | Efeitos colaterais em cascata entre domínios | Controle de estado complexo de uma página |

### 7.3. Event Bus vs. Chamada Direta de Método

| Critério | Event Bus | Chamada Direta |
|----------|-----------|----------------|
| **Acoplamento** | Zero (publicador não sabe quem consome) | Total (chamador depende do chamado) |
| **Escalabilidade** | Adicionar efeito = adicionar `Listener` | Adicionar efeito = modificar chamador |
| **Rastreabilidade** | Requer logging/tooling | Stack trace direto no debugger |
| **Quando usar** | Efeitos colaterais **entre** domínios | Lógica **dentro** do mesmo domínio |

### 7.4. Event Bus vs. Event Sourcing

| Critério | Event Bus | Event Sourcing |
|----------|-----------|----------------|
| **Persistência** | Eventos são efêmeros (não salvos no banco) | Eventos são a fonte da verdade (persistidos) |
| **Reconstrução de estado** | Impossível (estado está no SQLite/Drift) | Possível (replay de eventos) |
| **Complexidade** | Baixa | Alta |
| **Quando usar** | Comunicação interna entre domínios | Auditoria completa, CQRS avançado, sistemas financeiros regulados |

---

## 8. Implementação Mínima de Referência

### 8.1. Classe Base `DomainEvent`

```dart
// lib/core/events/domain_event.dart
abstract class DomainEvent {
  final DateTime timestamp;

  DomainEvent() : timestamp = DateTime.now();
}
```

### 8.2. Event Bus

```dart
// lib/core/events/event_bus.dart
import 'dart:async';

class EventBus {
  final _controller = StreamController<DomainEvent>.broadcast();

  void publish(DomainEvent event) {
    _controller.add(event);
  }

  Stream<T> on<T extends DomainEvent>() {
    return _controller.stream.whereType<T>();
  }

  void dispose() {
    _controller.close();
  }
}
```

### 8.3. Listener Base

```dart
// lib/application/listeners/base_listener.dart
import 'dart:async';

abstract class BaseListener {
  final List<StreamSubscription> _subscriptions = [];

  void subscribeTo(EventBus eventBus);

  void addSubscription(StreamSubscription subscription) {
    _subscriptions.add(subscription);
  }

  void dispose() {
    for (final sub in _subscriptions) {
      sub.cancel();
    }
  }
}
```

### 8.4. Exemplo Completo de Listener

```dart
// lib/application/listeners/estoque_listener.dart
class EstoqueListener extends BaseListener {
  final ProdutoRepository _produtoRepository;

  EstoqueListener(this._produtoRepository);

  @override
  void subscribeTo(EventBus eventBus) {
    addSubscription(
      eventBus.on<VendaFinalizadaEvent>().listen(_onVendaFinalizada),
    );
    addSubscription(
      eventBus.on<VendaCanceladaEvent>().listen(_onVendaCancelada),
    );
  }

  void _onVendaFinalizada(VendaFinalizadaEvent event) {
    _produtoRepository.darBaixaDeEstoquePorVenda(event.itensVendidos);
  }

  void _onVendaCancelada(VendaCanceladaEvent event) {
    _produtoRepository.estornarEstoquePorVenda(event.itensVendidos);
  }
}
```

### 8.5. Registro no Container de DI

```dart
// lib/core/config/injection_container.dart
void setupListeners() {
  // Event Bus (singleton)
  getIt.registerSingleton<EventBus>(EventBus());

  // Listeners (singletons, registrados com subscribe automático)
  getIt.registerSingleton<EstoqueListener>(
    EstoqueListener(getIt<ProdutoRepository>())
      ..subscribeTo(getIt<EventBus>()),
  );

  getIt.registerSingleton<FinanceiroListener>(
    FinanceiroListener(getIt<FinanceiroRepository>())
      ..subscribeTo(getIt<EventBus>()),
  );
}
```

---

## 9. Checklist de Implementação

Ao implementar o Event Bus no projeto, verifique:

- [ ] Eventos são publicados **apenas por Repositories**, após persistência bem-sucedida.
- [ ] Listeners estão na camada `application/listeners/`, não na UI nem no domínio puro.
- [ ] Listeners são **singletons** registrados no `GetIt` no boot da aplicação.
- [ ] **Nenhum Widget, Page ou Controller** escuta o Event Bus diretamente.
- [ ] Eventos carregam **snapshots completos** — consumidores não consultam o banco.
- [ ] Eventos são **imutáveis** — nenhum campo pode ser alterado após criação.
- [ ] Nomes de eventos são **fatos no passado** (`PedidoCriadoEvent`, não `CriarPedidoEvent`).
- [ ] Estratégia de falha está **definida no documento de arquitetura** (fire-and-forget, transacional ou híbrida).
- [ ] Event Bus **não é usado para queries** — para isso, use consulta direta ao Repository.
- [ ] Event Bus **não é usado para comunicação intra-domínio** — para isso, use chamada de método.
- [ ] Event Bus **não é usado para eventos de infraestrutura** (conectividade, auth) — para isso, use Signal/ValueNotifier global.
- [ ] Valores monetários nos eventos usam `BigDecimal`, não `double`.
- [ ] IDs nos eventos são `String` (UUID), não `int`.

---

## 10. Resumo Executivo

1. O Event Bus é usado **exclusivamente** para efeitos colaterais entre domínios no **fluxo de escrita**.
2. **Repositories publicam** eventos após persistência. **Listeners consomem** eventos na camada `application/`.
3. A **UI nunca** interage com o Event Bus. Ela usa `ChangeNotifier`, `Signal`, `ValueNotifier` ou `Stream` (Drift `.watch()`).
4. Eventos são **snapshots imutáveis e autossuficientes** — o consumidor nunca precisa consultar o banco.
5. O Event Bus **não substitui** e **não é substituído** por Streams, Signals ou Bloc. São ferramentas **complementares** para problemas **diferentes**.
6. As críticas comuns ao Event Bus (memory leak, imprevisibilidade, falta de rastreabilidade) se aplicam ao **uso incorreto na camada de UI** e não ao uso correto como **mediador de domínio na camada de aplicação**.
