# Padronização de Código no Spring Boot

Este documento serve como um guia de padronização de código e arquitetura para o desenvolvimento backend com **Spring Boot (Java 17 / 21+ e Spring Boot 3.x+)**. Ele foi elaborado para garantir a consistência, manutenibilidade, segurança e escalabilidade dos microsserviços e APIs da equipe. Seguir estas diretrizes e adotar as ferramentas descritas aqui é fundamental para a colaboração eficiente e a qualidade do software em produção.

> *Este guia é um documento vivo e pode ser atualizado conforme a equipe evolui e adota novas práticas. Se você tiver alguma dúvida ou sugestão, sinta-se à vontade para discuti-la com a equipe.*

> ***(Texto tachado ainda está em estudo)***

---

## 1. Ferramentas e Configurações

Para garantir um ambiente de desenvolvimento coeso e automatizado, adotamos as seguintes ferramentas e plugins:

- **IDEs Recomendadas**:
  - **IntelliJ IDEA (Ultimate ou Community)**: IDE padrão recomendada para desenvolvimento Java/Spring.
  - **VS Code**: Com o pacote **Extension Pack for Java** e **Spring Boot Extension Pack** (Microsoft & VMware).

- **Plugins e Extensões Essenciais**:
  - **Lombok Plugin**: Suporte a anotações como `@Getter`, `@Setter`, `@RequiredArgsConstructor` e `@Slf4j`.
  - **SonarLint**: Análise estática em tempo real para detectar bugs, falhas de segurança e *code smells* antes do commit.
  - **Checkstyle / Spotless**: Garantia de formatação e estilo consistente de código Java.
  - **GitLens**: Rastreabilidade e histórico de alterações diretamente no editor.

- **Análise Estática e Qualidade**:
  - **Spotless / Checkstyle**: Padronização de formatação automática (Google Java Style ou padrão da equipe) configurada no build (*Maven/Gradle*).
  - **SonarQube / SonarCloud**: Integração contínua para monitoramento de dívida técnica, vulnerabilidades e cobertura de testes.
  - ~~**Husky & Pre-commit Hooks**~~: Automação da validação de compilação e formatação antes do commit local.

---

## 2. Estrutura de Arquivos e Nomenclatura

Convenções claras de nomenclatura facilitam a leitura, navegação e integração entre múltiplos desenvolvedores.

### Nomenclatura

- **Pacotes**: Sempre em letras minúsculas (`lowercase`), sem sublinhados e separados por pontos (`com.empresa.projeto.modulo`).
- **Classes, Interfaces, Records e Enums**: `PascalCase` (*`UsuarioService`*, *`PedidoRepository`*, *`StatusPedidoEnum`*, *`CriarUsuarioRequest`*).
  - **Controllers**: Sufixo `Controller` (*`UsuarioController`*).
  - **Services**: Sufixo `Service` (*`UsuarioService`*, *`ProcessarPagamentoService`*).
  - **Repositories**: Sufixo `Repository` (*`UsuarioRepository`*).
  - **Entities (JPA)**: Nome do domínio em `PascalCase` sem sufixo desnecessário (*`Usuario`*, *`Pedido`*, *`ItemPedido`*).
  - **DTOs (Data Transfer Objects)**: Sufixos que expressem a intenção, preferencialmente usando `Record` (*`UsuarioResponse`*, *`CriarUsuarioRequest`*, *`AtualizarStatusRequest`*).
  - **Mappers**: Sufixo `Mapper` (*`UsuarioMapper`*).
  - **Exceptions**: Sufixo `Exception` (*`RecursoNaoEncontradoException`*, *`RegraDeNegocioException`*).
  - **Configs**: Sufixo `Config` (*`SecurityConfig`*, *`OpenApiConfig`*).
- **Métodos e Variáveis**: `camelCase` (*`calcularTotal()`*, *`buscarPorId()`*, *`dataCriacao`*).
- **Constantes**: `UPPER_SNAKE_CASE` (*`MAX_TENTATIVAS_LOGIN`*, *`CABECALHO_AUTORIZACAO`*).
- **Tabelas e Colunas no Banco de Dados**: `snake_case` (*`tb_usuario`*, *`tb_item_pedido`*, *`data_criacao`*, *`usuario_id`*).
- **Endpoints REST**: `kebab-case`, substantivos no plural e organizados de forma hierárquica:
  - `GET /api/v1/usuarios` (Listagem)
  - `GET /api/v1/usuarios/{id}` (Consulta por ID)
  - `POST /api/v1/usuarios` (Criação)
  - `PUT /api/v1/usuarios/{id}` (Atualização total)
  - `PATCH /api/v1/usuarios/{id}/status` (Atualização parcial)
  - `DELETE /api/v1/usuarios/{id}` (Exclusão)

---

### Estrutura de Pastas e Pacotes

Recomendamos a organização por **Features/Domínios** ou **Camadas Modulares**, mantendo alta coesão e baixo acoplamento:

```text
src/
├── main/
│   ├── java/
│   │   └── com/empresa/app/
│   │       ├── Application.java               # Classe principal de inicialização (@SpringBootApplication)
│   │       │
│   │       ├── core/                          # Configurações globais, segurança e infraestrutura comum
│   │       │   ├── config/                    # Configurações do Spring (Security, OpenAPI, Jackson, Cors)
│   │       │   │   ├── SecurityConfig.java
│   │       │   │   └── OpenApiConfig.java
│   │       │   ├── exception/                 # Tratamento global de erros e exceções customizadas
│   │       │   │   ├── GlobalExceptionHandler.java
│   │       │   │   ├── ProblemDetailResponse.java
│   │       │   │   └── RecursoNaoEncontradoException.java
│   │       │   ├── filter/                    # Filtros de requisição (JWT, Logging, CorrelationId)
│   │       │   │   └── JwtAuthFilter.java
│   │       │   └── util/                      # Classes utilitárias puras
│   │       │       └── DateUtils.java
│   │       │
│   │       └── domain/                        # Módulos de negócio (Features / Bounded Contexts)
│   │           ├── usuario/
│   │           │   ├── controller/            # Entrada REST da feature
│   │           │   │   └── UsuarioController.java
│   │           │   ├── dto/                   # Records para Request e Response
│   │           │   │   ├── CriarUsuarioRequest.java
│   │           │   │   └── UsuarioResponse.java
│   │           │   ├── entity/                # Entidades JPA de banco de dados
│   │           │   │   └── Usuario.java
│   │           │   ├── mapper/                # Mapeamento DTO <-> Entity (MapStruct)
│   │           │   │   └── UsuarioMapper.java
│   │           │   ├── repository/            # Interfaces Spring Data JPA
│   │           │   │   └── UsuarioRepository.java
│   │           │   └── service/               # Regras de negócio e casos de uso
│   │           │       └── UsuarioService.java
│   │           │
│   │           └── pedido/
│   │               ├── controller/
│   │               ├── dto/
│   │               ├── entity/
│   │               ├── repository/
│   │               └── service/
│   │
│   └── resources/
│       ├── db/
│       │   └── migration/                     # Scripts de versionamento do banco (Flyway)
│       │       ├── V1__criar_tabela_usuarios.sql
│       │       └── V2__criar_tabela_pedidos.sql
│       ├── application.yml                    # Configurações base da aplicação
│       ├── application-dev.yml                # Perfil de desenvolvimento
│       └── application-prod.yml               # Perfil de produção
│
└── test/
    ├── java/
    │   └── com/empresa/app/
    │       ├── core/
    │       └── domain/
    │           └── usuario/
    │               ├── controller/UsuarioControllerTest.java  # Testes de Controller (@WebMvcTest)
    │               ├── repository/UsuarioRepositoryTest.java  # Testes de Repositório (@DataJpaTest)
    │               └── service/UsuarioServiceTest.java        # Testes Unitários com Mockito
    └── resources/
        └── application-test.yml               # Configurações para ambiente de testes
```

---

## 3. Boas Práticas no Spring Boot

### 3.1. Java Moderno e Spring Boot 3+
- **Use Java 17 ou 21 LTS**: Aproveite recursos nativos como **Java Records** (perfeitos para DTOs imutáveis), *Pattern Matching*, *Text Blocks* e *Virtual Threads* (Java 21 / Spring Boot 3.2+ com `spring.threads.virtual.enabled=true`).
- **Namespace `jakarta.*`**: Em conformidade com o Spring Boot 3+, use sempre os pacotes `jakarta.persistence.*`, `jakarta.validation.*` e `jakarta.servlet.*` (nunca o legado `javax.*`).

### 3.2. Injeção de Dependências por Construtor
- **Nunca use `@Autowired` em campos de classe** (*Field Injection*). A injeção por construtor garante imutabilidade (`final`), facilita testes unitários puros sem carregar o contexto Spring e evita dependências circulares ocultas.
- Use a anotação `@RequiredArgsConstructor` do Lombok para reduzir o código boilerplate.

```java
@Service
@RequiredArgsConstructor
public class UsuarioService {

    private final UsuarioRepository usuarioRepository;
    private final UsuarioMapper usuarioMapper;
    private final NotificacaoService notificacaoService;

    // Construtor com campos 'final' gerado automaticamente pelo Lombok
}
```

---

### 3.3. DTOs Imutáveis com `record` e Validação Declarativa
- **Nunca exponha entidades JPA diretamente nos Controllers**: Entidades contêm detalhes do banco de dados, podem expor campos sensíveis (como hashes de senhas) e causar problemas de serialização recursiva (*LazyInitializationException*).
- Use **Java Records** para DTOs de entrada (*Request*) e saída (*Response*).
- Valide dados de entrada com as anotações do **Jakarta Validation** (`@NotNull`, `@NotBlank`, `@Size`, `@Email`, `@Min`, etc.) combinadas com `@Valid` nos parâmetros do Controller.

```java
// DTO de Entrada
public record CriarUsuarioRequest(
    @NotBlank(message = "O nome é obrigatório")
    @Size(min = 3, max = 100, message = "O nome deve ter entre 3 e 100 caracteres")
    String nome,

    @NotBlank(message = "O e-mail é obrigatório")
    @Email(message = "Formato de e-mail inválido")
    String email,

    @NotBlank(message = "A senha é obrigatória")
    @Size(min = 8, message = "A senha deve ter no mínimo 8 caracteres")
    String senha
) {}

// DTO de Saída
public record UsuarioResponse(
    Long id,
    String nome,
    String email,
    LocalDateTime dataCriacao
) {}
```

---

### 3.4. Controladores REST Padronizados
- Os controllers devem ser leves, limitando-se a receber requisições, delegar para a camada de serviço e formatar a resposta HTTP adequada com `ResponseEntity`:

```java
@RestController
@RequestMapping("/api/v1/usuarios")
@RequiredArgsConstructor
public class UsuarioController {

    private final UsuarioService usuarioService;

    @PostMapping
    public ResponseEntity<UsuarioResponse> criar(
        @RequestBody @Valid CriarUsuarioRequest request,
        UriComponentsBuilder uriBuilder
    ) {
        UsuarioResponse novoUsuario = usuarioService.criar(request);
        URI uri = uriBuilder.path("/api/v1/usuarios/{id}").buildAndExpand(novoUsuario.id()).toUri();
        return ResponseEntity.created(uri).body(novoUsuario);
    }

    @GetMapping("/{id}")
    public ResponseEntity<UsuarioResponse> buscarPorId(@PathVariable Long id) {
        return ResponseEntity.ok(usuarioService.buscarPorId(id));
    }

    @GetMapping
    public ResponseEntity<Page<UsuarioResponse>> listar(
        @PageableDefault(size = 20, sort = "nome", direction = Sort.Direction.ASC) Pageable pageable
    ) {
        return ResponseEntity.ok(usuarioService.listar(pageable));
    }
}
```

---

### 3.5. Tratamento Global de Exceções e RFC 7807 (`ProblemDetail`)
- Centralize o tratamento de erros em uma classe com `@RestControllerAdvice`.
- Utilize o padrão **RFC 7807 / RFC 9457 (`ProblemDetail`)**, nativo a partir do Spring 6 e Spring Boot 3:

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(RecursoNaoEncontradoException.class)
    public ProblemDetail handleRecursoNaoEncontrado(RecursoNaoEncontradoException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        problem.setTitle("Recurso Não Encontrado");
        problem.setProperty("timestamp", Instant.now());
        return problem;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail handleValidacao(MethodArgumentNotValidException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, "Erro de validação nos campos informados.");
        problem.setTitle("Dados Inválidos");

        Map<String, String> erros = ex.getBindingResult().getFieldErrors().stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                fieldError -> fieldError.getDefaultMessage() != null ? fieldError.getDefaultMessage() : "Inválido",
                (antigo, novo) -> antigo
            ));

        problem.setProperty("erros", erros);
        problem.setProperty("timestamp", Instant.now());
        return problem;
    }

    @ExceptionHandler(Exception.class)
    public ProblemDetail handleErroGenerico(Exception ex) {
        log.error("Erro inesperado no servidor: ", ex);
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.INTERNAL_SERVER_ERROR, 
            "Ocorreu um erro interno. Contate o suporte se o problema persistir."
        );
        problem.setTitle("Erro Interno do Servidor");
        problem.setProperty("timestamp", Instant.now());
        return problem;
    }
}
```

---

### 3.6. Transacionalidade e Performance no Banco de Dados
- **Uso correto do `@Transactional`**:
  - Em métodos que apenas leem dados, utilize sempre `@Transactional(readOnly = true)`. Isso otimiza o Hibernate (desativando o *dirty checking* e permitindo otimizações no driver JDBC).
  - Em métodos que realizam escrita/alteração de dados, utilize `@Transactional`.
- **Evite o problema do N+1**: Em relacionamentos JPA (`@OneToMany`, `@ManyToOne`), utilize consultas com `JOIN FETCH`, `@EntityGraph` ou DTO Projections para carregar os dados necessários em uma única query.
- **Versionamento de Banco de Dados Obrigatório**: Utilize **Flyway** ou **Liquibase**. O parâmetro `spring.jpa.hibernate.ddl-auto` deve ser configurado como `validate` ou `none` em ambientes de produção.

---

### 3.7. Logs Estruturados com SLF4J
- Utilize a anotação `@Slf4j` do Lombok.
- **Evite concatenação de strings em logs**: use os placeholders `{}` para ganho de performance.
- Respeite os níveis de log:
  - `ERROR`: Falhas críticas que impedem o fluxo normal ou exigem atenção imediata.
  - `WARN`: Situações anômalas, mas que a aplicação conseguiu contornar (ex: retry, fallback).
  - `INFO`: Marcos importantes de execução de negócio (ex: pedido criado, pagamento processado).
  - `DEBUG`: Detalhes técnicos úteis para depuração em ambiente de desenvolvimento.

```java
// Correto:
log.info("Processando pagamento para o pedido ID: {} com valor: {}", pedidoId, valor);

// Evite:
log.info("Processando pagamento para o pedido ID: " + pedidoId + " com valor: " + valor);
```

---

### 3.8. Testes Automatizados
- **Pirâmide de Testes**:
  - **Testes Unitários**: Rápidos, cobrindo regras de negócio puras nos Services usando **JUnit 5** e **Mockito** (sem levantar o contexto Spring).
  - **Testes de Integração Web**: Utilizando `@WebMvcTest` para validar endpoints, status codes, validações e serialização JSON.
  - **Testes de Integração com Banco**: Utilizando `@DataJpaTest` ou `@SpringBootTest` com **Testcontainers** para testar queries e repositórios em um banco de dados real idêntico ao de produção (ex: PostgreSQL/MySQL).

```java
@ExtendWith(MockitoExtension.class)
class UsuarioServiceTest {

    @Mock
    private UsuarioRepository usuarioRepository;

    @Mock
    private UsuarioMapper usuarioMapper;

    @InjectMocks
    private UsuarioService usuarioService;

    @Test
    @DisplayName("Deve lançar exceção ao buscar usuário por ID inexistente")
    void deveLancarExcecaoAoBuscarIdInexistente() {
        Long idInexistente = 99L;
        when(usuarioRepository.findById(idInexistente)).thenReturn(Optional.empty());

        assertThrows(RecursoNaoEncontradoException.class, () -> usuarioService.buscarPorId(idInexistente));
        verify(usuarioRepository, times(1)).findById(idInexistente);
    }
}
```

---

## 4. Configurações nos Projetos

Estes arquivos padronizados devem estar presentes nos repositórios da equipe.

### .gitignore (Padronizado para Java, Spring Boot e IDEs)

```gitignore
# -----------------------------------------------------------------------------
# Compilação e Build (Maven / Gradle)
# -----------------------------------------------------------------------------
target/
build/
out/
*.class
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# -----------------------------------------------------------------------------
# IDEs e Editores
# -----------------------------------------------------------------------------
# IntelliJ IDEA
.idea/
*.iml
*.ipr
*.iws
out/

# Eclipse
.classpath
.project
.settings/
bin/
tmp/
.metadata
*.launch
.apt_generated

# Visual Studio Code
.vscode/
!.vscode/settings.json
!.vscode/tasks.json
!.vscode/launch.json
!.vscode/extensions.json
.history/

# -----------------------------------------------------------------------------
# Logs e Arquivos Temporários
# -----------------------------------------------------------------------------
*.log
logs/
pids/
*.pid
*.seed
*.pid.lock
hs_err_pid*.log

# -----------------------------------------------------------------------------
# Configurações Locais e Variáveis de Ambiente
# -----------------------------------------------------------------------------
.env
.env.local
local.properties
/DevProfile

# -----------------------------------------------------------------------------
# Sistema Operacional
# -----------------------------------------------------------------------------
.DS_Store
Thumbs.db
```

---

### application.yml (Estrutura de Perfis)

```yaml
spring:
  application:
    name: ms-usuario
  profiles:
    active: dev
  threads:
    virtual:
      enabled: true # Habilita Virtual Threads no Java 21+

server:
  port: 8080
  shutdown: graceful

# Monitoramento com Spring Boot Actuator
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
  endpoint:
    health:
      show-details: when_authorized

---
# Perfil de Desenvolvimento (application-dev.yml)
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:postgresql://localhost:5432/db_usuarios
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:postgres}
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: true
    properties:
      hibernate:
        format_sql: true
  flyway:
    enabled: true

---
# Perfil de Produção (application-prod.yml)
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
  flyway:
    enabled: true
```

---

### .editorconfig (Formatação Base para Java)

```ini
# Editor configuration, see https://editorconfig.org
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 4
insert_final_newline = true
trim_trailing_whitespace = true
end_of_line = lf

[*.{yml,yaml,json,xml,md}]
indent_size = 2

[*.md]
max_line_length = off
```

---

## 5. Configurações para VS Code (Projetos Java / Spring Boot)

Para desenvolvedores que utilizam o VS Code, disponibilizamos os arquivos padronizados para a pasta `.vscode`:

### .vscode/extensions.json

```json
{
  "recommendations": [
    "vscjava.vscode-java-pack",
    "vmware.vscode-spring-boot",
    "vscjava.vscode-spring-initializr",
    "vscjava.vscode-spring-boot-dashboard",
    "vscjava.vscode-java-dependency",
    "Sonarlint.sonarlint",
    "eamodio.gitlens",
    "streetsidesoftware.code-spell-checker",
    "streetsidesoftware.code-spell-checker-portuguese-brazilian"
  ]
}
```

### .vscode/settings.json

```json
{
  "java.configuration.updateBuildConfiguration": "automatic",
  "java.compile.nullAnalysis.mode": "automatic",
  "java.format.settings.url": "https://raw.githubusercontent.com/google/styleguide/gh-pages/eclipse-java-google-style.xml",
  "java.format.settings.profile": "GoogleStyle",
  "editor.formatOnSave": true,
  "editor.tabSize": 4,
  "editor.insertSpaces": true,
  "[xml]": {
    "editor.tabSize": 2
  },
  "[yaml]": {
    "editor.tabSize": 2
  },
  "[json]": {
    "editor.tabSize": 2
  },
  "files.exclude": {
    "**/.git": true,
    "**/.DS_Store": true,
    "**/target": true,
    "**/build": true,
    "**/.metadata": true
  }
}
```

---

## 6. Checklist de Qualidade do Código

Antes de abrir um *Pull Request*, certifique-se de validar os seguintes itens:

1. [ ] Não há anotações `@Autowired` em atributos de classe (injeção por construtor aplicada).
2. [ ] Nenhuma Entidade JPA está sendo exposta ou recebida diretamente em endpoints REST (uso de DTOs / Records).
3. [ ] Todos os DTOs de entrada possuem validações com `jakarta.validation` e os endpoints usam `@Valid`.
4. [ ] Métodos de consulta possuem `@Transactional(readOnly = true)`.
5. [ ] Tratamento de exceções centralizado no `@RestControllerAdvice` retornando `ProblemDetail`.
6. [ ] Não há logs com `System.out.println()` (uso exclusivo de `@Slf4j` com placeholders `{}`).
7. [ ] Testes unitários cobrem as regras de negócio críticas da camada de serviço.
8. [ ] Scripts de migração de banco de dados (Flyway) estão devidamente versionados e idempotentes.
