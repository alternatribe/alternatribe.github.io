---
title: Guia Completo de Markdown para MkDocs
description: Tutorial prático de sintaxe Markdown e uso de extensões e plugins configurados no Material for MkDocs.
tags:
  - markdown
  - tutorial
  - mkdocs
  - documentacao
---

# Guia e Tutorial de Markdown no MkDocs

Este guia é um tutorial prático e completo sobre como escrever documentos em **Markdown** aproveitando ao máximo todos os plugins e extensões habilitados na Central de Tutoriais (Material for MkDocs).

---

## 1. Sintaxe Básica do Markdown

Antes de explorar as extensões avançadas, aqui está o resumo dos elementos essenciais do Markdown padrão.

### 1.1 Títulos e Cabeçalhos

Use `#` para definir a hierarquia de títulos (recomenda-se apenas um `#` por página para o título principal):

```markdown
# Título Principal (H1)
## Seção (H2)
### Subseção (H3)
#### Tópico (H4)
##### Subtópico (H5)
###### Detalhe (H6)
```

### 1.2 Formatação de Texto

```markdown
- **Texto em negrito** ou __negrito__
- *Texto em itálico* ou _itálico_
- ***Texto em negrito e itálico***
- ~~Texto tachado / riscado~~
- `código em linha simples`
```

**Resultado:**
- **Texto em negrito**
- *Texto em itálico*
- ***Texto em negrito e itálico***
- ~~Texto tachado / riscado~~
- `código em linha simples`

### 1.3 Listas

```markdown
<!-- Lista não ordenada -->
- Item A
- Item B
    - Subitem B.1
    - Subitem B.2
- Item C

<!-- Lista ordenada -->
1. Primeiro passo
2. Segundo passo
3. Terceiro passo
```

### 1.4 Links e Citações

```markdown
[Visitar Repositório](https://github.com/alternatribe)

> Esta é uma citação em bloco (blockquote).
> Pode conter múltiplas linhas.
```

---

## 2. Metadados e Tags (`tags`)

Graças ao plugin **`tags`**, você pode categorizar páginas adicionando um bloco de metadados (*Frontmatter*) no topo do arquivo `.md`.

### Como usar:

Insira no início do documento (antes de qualquer conteúdo):

```markdown
---
title: Título da Página
description: Breve resumo para SEO e ferramentas de busca.
tags:
  - flutter
  - tutorial
  - arquitetura
---
```

> As tags facilitam a busca global e geram agrupamentos automáticos de tutoriais relacionados.

---

## 3. Caixas de Aviso e Alertas (`admonition`)

As caixas de aviso (admonitions) destacam informações críticas, dicas e avisos com estilos visuais específicos.

### Sintaxe Básica

A sintaxe utiliza `!!!` seguido do tipo de caixa e do título entre aspas:

```markdown
!!! note "Nota Informativa"
    Este é um aviso do tipo nota para informações contextuais.

!!! tip "Dica Útil"
    Utilize atalhos para aumentar sua produtividade.

!!! info "Informação"
    Detalhes sobre a configuração do ambiente.

!!! warning "Atenção"
    Revise as credenciais antes de prosseguir.

!!! danger "Perigo / Cuidado"
    A execução deste comando apagará o banco de dados.

!!! success "Sucesso"
    Operação concluída com êxito!

!!! failure "Falha"
    Não foi possível conectar ao servidor.

!!! bug "Bug Conhecido"
    Existe uma issue aberta referente a este comportamento.

!!! example "Exemplo Prático"
    Veja como instanciar a classe no código abaixo.

!!! quote "Citação"
    "A simplicidade é o último grau de sofisticação." — Leonardo da Vinci
```

### Caixas de Aviso sem Título

Para remover o título personalizado e manter apenas o ícone padrão:

```markdown
!!! note ""
    Esta caixa de aviso não possui cabeçalho de texto.
```

---

## 4. Caixas Retráteis / Colapsáveis (`pymdownx.details`)

Permite criar blocos que o usuário pode expandir ou recolher clicando no cabeçalho.

- `???`: Caixa **fechada / recolhida** por padrão.
- `???+`: Caixa **aberta / expandida** por padrão.

```markdown
??? note "Clique para expandir os detalhes (Fechado por padrão)"
    Conteúdo oculto que só aparece quando o usuário clica para abrir.

???+ tip "Dica Importante (Aberto por padrão)"
    Este bloco já vem expandido, mas pode ser recolhido pelo usuário.
```

---

## 5. Blocos de Código Avançados (`highlight` & `superfences`)

O MkDocs está configurado com destaque de sintaxe inteligente, botão de cópia automática, títulos de arquivos e numeração de linhas.

### 5.1 Bloco com Título e Numeração de Linhas

Adicione `title="nome_do_arquivo"` e `linenums="1"`:

````markdown
```python title="main.py" linenums="1"
def saudacao(nome: str) -> str:
    """Retorna mensagem de boas-vindas."""
    return f"Olá, {nome}!"

if __name__ == "__main__":
    print(saudacao("Desenvolvedor"))
```
````

### 5.2 Destacando Linhas Específicas (`hl_lines`)

Você pode iluminar linhas-chave usando `hl_lines="linha"` ou intervalos `hl_lines="inicio-fim"`:

````markdown
```dart title="exemplo.dart" hl_lines="2 6-7"
void main() {
  final mensagem = "Linha destacada individualmente!";
  print(mensagem);
  
  for (int i = 0; i < 3; i++) {
    // Estas duas linhas
    // também estão destacadas
  }
}
```
````

### 5.3 Código em Linha com Sintaxe Colorida (`pymdownx.inlinehilite`)

Você pode aplicar coloração de sintaxe em códigos inline usando o prefixo `#!linguagem`:

```markdown
Execute o comando `#!bash git checkout -b feature/nova-tela` no terminal.
A variável de ambiente padrão é `#!python os.getenv("API_KEY")`.
```

### 5.4 Código Aninhado dentro de Admonitions (`superfences`)

Com o `pymdownx.superfences`, você pode colocar blocos de código completos dentro de caixas de aviso (lembrando de indentar o conteúdo com 4 espaços):

````markdown
!!! example "Exemplo de Endpoint"
    Requisição para autenticação:
    
    ```json
    {
      "email": "dev@alternatribe.com",
      "token": "xyz123abc"
    }
    ```
````

---

## 6. Abas de Conteúdo Alternadas (`pymdownx.tabbed`)

Permite organizar códigos ou instruções em abas selecionáveis (ex.: comparar linguagens, sistemas operacionais ou fluxos):

````markdown
=== "Dart / Flutter"

    ```dart
    void main() {
      runApp(const MeuApp());
    }
    ```

=== "TypeScript"

    ```typescript
    const iniciarApp = (): void => {
      console.log("App iniciado");
    };
    ```

=== "Python"

    ```python
    def iniciar_app():
        print("App iniciado")
    ```
````

> **Importante:** Indente o conteúdo de cada aba com 4 espaços após a linha `=== "Nome da Aba"`.

---

## 7. Listas de Tarefas / Checklists (`pymdownx.tasklist`)

Crie checklists visuais e limpas para tutoriais e procedimentos operacionais:

```markdown
- [x] Configurar o ambiente de desenvolvimento
- [x] Clonar o repositório oficial
- [ ] Rodar as migrações do banco de dados
- [ ] Executar os testes automatizados (`pytest` / `flutter test`)
```

**Resultado:**
- [x] Configurar o ambiente de desenvolvimento
- [x] Clonar o repositório oficial
- [ ] Rodar as migrações do banco de dados
- [ ] Executar os testes automatizados

---

## 8. Imagens, Zoom e Atributos (`glightbox` & `attr_list`)

O plugin **Glightbox** permite abrir imagens e prints com efeito de zoom fluido ao clicar. Combinado com o **`attr_list`**, é possível controlar tamanho, alinhamento e legendas.

### 8.1 Inserção Básica com Zoom Automático

```markdown
![Interface do Sistema](caminho/para/imagem.png)
```

### 8.2 Definindo Dimensões e Alinhamento com `attr_list`

```markdown
<!-- Imagem redimensionada para 500px de largura -->
![Diagrama de Arquitetura](caminho/para/diagrama.png){ width="500" }

<!-- Imagem alinhada ao centro com sombra ou classe -->
![Fluxo de Telas](caminho/para/fluxo.png){ width="70%" align=center }
```

---

## 9. Organização de Pastas e Menus (`awesome-pages`)

Com o plugin **`awesome-pages`**, a estrutura de navegação do menu lateral pode ser controlada criando um arquivo chamado `.pages` dentro de qualquer pasta de documentação.

### Exemplo de arquivo `.pages`:

```yaml
title: Módulo Flutter
arrange:
  - index.md
  - setup.md
  - arquitetura.md
  - ...
```

- `title`: Define o nome amigável exibido na barra de navegação lateral.
- `arrange`: Define a ordem exata em que os arquivos e subpastas aparecerão.
- `...`: Garante que arquivos não listados explicitamente continuem aparecendo ao final.

---

## 10. Resumo de Boas Práticas

1. **Um Único H1 (`#`)**: Comece o documento com `# Título Principal` e use `##` para seções subsequentes.
2. **Indentação de 4 Espaços**: Conteúdos dentro de admonitions (`!!!`), abas (`===`) e listas aninhadas devem ser indentados com 4 espaços.
3. **Nomeação de Arquivos**: Prefira letras minúsculas e hífens para nomes de arquivos (ex: `guia-instalacao.md`).
4. **Legibilidade**: Use títulos de blocos de código (`title="..."`) para indicar onde o código deve ser salvo ou executado.
