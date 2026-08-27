---
title: Versionamento Automático no Git
---

# Manual Universal: Git, Servidor Git e Automação de Versão (SemVer)

Este manual foi criado para explicar, de forma simples e acessível (mesmo para quem não tem experiência prévia em programação), como funciona o controle de versão e o que é um servidor Git de forma agnóstica para **qualquer linguagem ou projeto** (como **Angular**, **Java Spring Boot**, **Node.js**, **Python**, **C#**, etc.), trazendo receitas completas e prontas para uso.

---

## 📋 Índice

1. [O que é Git e Servidor Git? (Para Leigos)](#1-o-que-e-git-e-servidor-git-para-leigos)
   * [A analogia do livro e dos rascunhos](#a-analogia-do-livro-e-dos-rascunhos)
   * [Dicionário de termos essenciais](#dicionario-de-termos-essenciais)
2. [O que é SemVer (Versionamento Semântico) e Build Number?](#2-o-que-e-semver-versionamento-semantico-e-build-number)
   * [Entendendo a estrutura MAJOR . MINOR . PATCH](#entendendo-a-estrutura-major-minor-patch)
   * [O que é Build Number (+N) e por que ele é necessário?](#o-que-e-build-number-n-e-por-que-ele-e-necessario)
   * [Tabela prática de decisões do SemVer](#tabela-pratica-de-decisoes-do-semver)
3. [O que são Git Hooks e como funciona a automação?](#3-o-que-sao-git-hooks-e-como-funciona-a-automacao)
4. [Como Configurar em um Projeto Novo (Do Zero)](#4-como-configurar-em-um-projeto-novo-do-zero)
   * [Projeto Genérico (Arquivo VERSION)](#projeto-generico-qualquer-linguagem-arquivo-version)
   * [Flutter / Dart (pubspec.yaml)](#flutter-dart-pubspecyaml)
   * [Angular / React / Node.js (package.json)](#angular-react-nodejs-packagejson)
   * [Java Spring Boot (Maven pom.xml ou Gradle)](#java-spring-boot-maven-pomxml-ou-gradle)
5. [Como Instalar/Ativar em um Projeto Clonado ou Existente](#5-como-instalarativar-em-um-projeto-clonado-ou-existente)
6. [Como usar no dia a dia (Guia Prático)](#6-como-usar-no-dia-a-dia-guia-pratico)
7. [Perguntas Frequentes e Resolução de Problemas](#7-perguntas-frequentes-e-resolucao-de-problemas)

---

## 1. O que é Git e Servidor Git? (Para Leigos)

### A analogia do livro e dos rascunhos

Imagine que várias pessoas estão escrevendo um livro juntas:
* **Sem Git**: As pessoas trocam arquivos por e-mail chamados `projeto_v1.zip`, `projeto_final.zip`, `projeto_final_agora_vai.zip`. Isso causa perda de código e confusão total.
* **Git**: É o controle de versão instalado no seu computador. Ele funciona como uma "máquina do tempo". Toda vez que você salva uma etapa (chamada de **Commit**), o Git registra exatamente o que mudou, quem mudou e quando mudou. Se algo quebrar, você pode voltar para qualquer momento do passado.
* **Servidor Git** (ex: GitHub, Bitbucket, GitLab, Azure DevOps): É a "nuvem central". Ele guarda o repositório mestre. Toda a equipe envia (**Push**) ou baixa (**Pull**) as alterações em relação a essa cópia central na nuvem.

### Dicionário de termos essenciais

| Termo | Significado na prática |
| :--- | :--- |
| **Repositório (Repo)** | A pasta inteira do projeto monitorada pelo Git. |
| **Commit** | Um ponto de salvamento registrado no histórico com uma mensagem descritiva. |
| **Push** | Enviar seus commits locais do seu computador para o **Servidor Git**. |
| **Pull** | Baixar as alterações mais recentes do **Servidor Git** para o seu computador. |
| **Tag** | Uma "etiqueta" colocada em um commit para marcar uma versão lançada oficial (ex: `v1.0.0`). |
| **Branch** | Uma linha de desenvolvimento paralela (ex: `main`, `develop`, `feature/login`). |
| **Clone** | Baixar a cópia inicial do repositório remoto para o seu computador. |

---

## 2. O que é SemVer (Versionamento Semântico) e Build Number?

**SemVer** (*Semantic Versioning*) é o padrão internacional que define como dar números de versão a qualquer sistema de software.

Uma versão completa pode ser representada por `MAJOR.MINOR.PATCH` (ex: `1.4.2`) ou acompanhada do **Build Number** `MAJOR.MINOR.PATCH+BUILD` (ex: `1.4.2+45`).

### Entendendo a estrutura `MAJOR . MINOR . PATCH`

1. **MAJOR (Versão Principal - `1`)**:
   * **Quando alterar?** Quando você faz uma alteração grande que **incompatibiliza** ou **quebra** o que existia antes (ex: refatoração completa do banco de dados, mudança total de APIs ou remoção de recursos).
   * *Regra:* Ao incrementar o MAJOR, o MINOR e o PATCH voltam para zero (ex: de `1.5.3` vai para `2.0.0`).

2. **MINOR (Versão Secundária - `4`)**:
   * **Quando alterar?** Quando novas funcionalidades ou novas telas são adicionadas de forma **compatível** (sem quebrar o que já funcionava).
   * *Regra:* Ao incrementar o MINOR, o PATCH volta para zero (ex: de `1.4.2` vai para `1.5.0`).

3. **PATCH (Correção de Bugs - `2`)**:
   * **Quando alterar?** Quando você apenas corrige um bug, erro de cálculo ou falha de segurança, sem adicionar novos recursos.
   * *Regra:* Incrementa apenas o último número (ex: de `1.4.2` vai para `1.4.3`).

---

### O que é Build Number (`+N`) e por que ele é necessário?

O **Build Number** (ou número de compilação) é o contador numérico que vem após o sinal de mais (`+`), por exemplo: `1.4.2+45`.

* **Identificador Único de Geração do Executável**: Você pode compilar um aplicativo 10 vezes no mesmo dia sem mudar as funcionalidades (SemVer `1.4.2` continua o mesmo), mas cada compilação gerada precisa de um número único sequencial (ex: `+45`, `+46`, `+47`).
* **Exigência de Lojas e Atualizadores**: Lojas digitais (Google Play Store, Apple App Store) e gerenciadores de atualização (Windows/Linux/Mobile) exigem obrigatoriamente que qualquer novo instalador enviado tenha um Build Number **estritamente maior** que o instalador anterior.
* **O Build Number é zerado ao mudar MAJOR ou MINOR?** **NÃO, ele NUNCA é zerado!**
  * Diferente do MINOR e PATCH (que voltam a zero quando o MAJOR sobe), o Build Number precisa ser **estritamente crescente durante toda a vida útil do aplicativo**.
  * Se você estiver na versão `1.9.0+50` e lançar a `2.0.0+1` (zerando o build), lojas como a Google Play (`versionCode`) e a Apple App Store (`CFBundleVersion`) **rejeitarão o envio imediatamente**, pois já existe um histórico com build 50.
* **Como a automação resolve isso?** Nosso script de Git Hook calcula o Build Number automaticamente baseado no número total de commits no repositório (`git rev-list --count HEAD + 1`). Dessa forma, mesmo ao pular de `1.9.0` para `2.0.0`, o build number continuará crescendo naturalmente (ex: `1.9.0+50` ➡️ `2.0.0+51`) sem qualquer risco de rejeição ou intervenção manual!

---

### Tabela prática de decisões do SemVer

| O que foi feito no projeto? | Tipo de Release | Exemplo de Mudança |
| :--- | :--- | :--- |
| Corrigiu um bug ou erro em uma tela/API | `patch` | `1.0.0+5` ➡️ `1.0.1+6` |
| Adicionou uma nova funcionalidade (ex: Pix, relatórios, filtro novo) | `minor` | `1.0.1+6` ➡️ `1.1.0+7` |
| Reescreveu o sistema, trocou o banco ou removeu compatibilidade | `major` | `1.4.2+20` ➡️ `2.0.0+21` |

---

## 3. O que são Git Hooks e como funciona a automação?

Um **Git Hook** é um script automatizado disparado pelo Git quando ocorrem eventos específicos (como antes de fazer um commit ou antes de um push).

Nesta solução, usamos o hook **`pre-commit`**:
* **O que ele faz?** Executa automaticamente no seu computador toda vez que você digita `git commit`.
* **Automação do Build Number / Versão**: Ele atualiza o arquivo de versão do projeto dinamicamente com base no histórico de commits (`git rev-list --count HEAD + 1`).
* **Vantagem**: Evita esquecimento humano e garante que todo commit tenha seu número de build atualizado automaticamente.

---

## 4. Como Configurar em um Projeto Novo (Do Zero)

Para configurar a automação em um projeto novo, você precisa de apenas **3 arquivos**:
1. `install.sh` (na raiz do projeto): script que ativa os hooks no Git local.
2. `scripts/pre-commit`: hook que atualiza o build number a cada commit.
3. `release.sh` (na raiz do projeto): script que valida testes, incrementa o SemVer, gera tag e faz push.

Escolha a tecnologia do seu projeto abaixo para ver a receita completa com os arquivos prontos:

---

### Projeto Genérico (Qualquer linguagem / Arquivo `VERSION`)

Use esta opção para Python, PHP, C#, Rust, Go ou qualquer projeto que não use um gerenciador de pacotes específico.

#### 1. Criar a pasta e o arquivo de versão inicial
```bash
mkdir -p scripts
echo "1.0.0" > VERSION
```

#### 2. Criar o arquivo `scripts/pre-commit`
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR=$(git rev-parse --show-toplevel)
cd "$ROOT_DIR"

if [[ -f VERSION ]]; then
  CURRENT_VERSION=$(cat VERSION)
  BASE_VERSION=${CURRENT_VERSION%%+*}
  BUILD_NUMBER=$(( $(git rev-list --count HEAD) + 1 ))
  NEW_VERSION="${BASE_VERSION}+${BUILD_NUMBER}"
  echo "$NEW_VERSION" > VERSION
  git add VERSION
fi
```

#### 3. Criar o arquivo `install.sh`
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR=$(cd "$(dirname "$0")" && pwd)
cd "$ROOT_DIR"

if [[ ! -d .git ]]; then
  echo "Erro: Execute na raiz de um repositório Git." >&2
  exit 1
fi

if [[ ! -f scripts/pre-commit ]]; then
  echo "Erro: scripts/pre-commit nao encontrado." >&2
  exit 1
fi

chmod +x scripts/pre-commit
git config core.hooksPath scripts

echo "core.hooksPath configurado para scripts com sucesso!"
```

#### 4. Criar o arquivo `release.sh`
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR=$(cd "$(dirname "$0")" && pwd)
cd "$ROOT_DIR"

BUMP_TYPE="${1:-}"

if [[ "$BUMP_TYPE" != "major" && "$BUMP_TYPE" != "minor" && "$BUMP_TYPE" != "patch" ]]; then
  echo "Uso: ./release.sh <major|minor|patch>" >&2
  exit 1
fi

if [[ -n "$(git status --porcelain)" ]]; then
  echo "Erro: Existem alteracoes nao comitadas. Faca commit antes de gerar release." >&2
  exit 1
fi

VERSION_STR=$(cat VERSION)
VERSION_ONLY=${VERSION_STR%%+*}

IFS='.' read -r MAJOR MINOR PATCH <<< "$VERSION_ONLY"

case "$BUMP_TYPE" in
  major) MAJOR=$((MAJOR + 1)); MINOR=0; PATCH=0 ;;
  minor) MINOR=$((MINOR + 1)); PATCH=0 ;;
  patch) PATCH=$((PATCH + 1)) ;;
esac

BUILD_NUMBER=$(( $(git rev-list --count HEAD) + 1 ))
NEW_VERSION="$MAJOR.$MINOR.$PATCH+$BUILD_NUMBER"
TAG_NAME="v$MAJOR.$MINOR.$PATCH"

if git rev-parse -q --verify "refs/tags/$TAG_NAME" >/dev/null; then
  echo "Tag ja existe: $TAG_NAME" >&2
  exit 1
fi

echo "$NEW_VERSION" > VERSION
echo "Versao atualizada para: $NEW_VERSION"

git add VERSION
git commit -m "chore: bump version to $NEW_VERSION"
git tag "$TAG_NAME"
git push origin HEAD
git push origin "$TAG_NAME"

echo "Release $TAG_NAME enviada com sucesso!"
```

#### 5. Dar permissão e ativar
```bash
chmod +x install.sh release.sh scripts/pre-commit
./install.sh
```

---

### Flutter / Dart (`pubspec.yaml`)

No Flutter, a versão é controlada na linha `version: 1.2.3+45` do `pubspec.yaml`.

#### 1. Criar a pasta `scripts`
```bash
mkdir -p scripts
```

#### 2. Criar o arquivo `scripts/pre-commit`
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR=$(git rev-parse --show-toplevel)
cd "$ROOT_DIR"

CURRENT_VERSION=$(grep -m1 '^version:' pubspec.yaml | awk '{print $2}')
BASE_VERSION=${CURRENT_VERSION%%+*}
BUILD_NUMBER=$(( $(git rev-list --count HEAD) + 1 ))
NEW_VERSION="${BASE_VERSION}+${BUILD_NUMBER}"

sed -i "s/^version: .*/version: ${NEW_VERSION}/" pubspec.yaml
git add pubspec.yaml
```

#### 3. Criar o arquivo `install.sh`
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR=$(cd "$(dirname "$0")" && pwd)
cd "$ROOT_DIR"

if [[ ! -d .git ]]; then
  echo "Erro: Execute na raiz de um repositório Git." >&2
  exit 1
fi

chmod +x scripts/pre-commit
git config core.hooksPath scripts

echo "core.hooksPath configurado para scripts com sucesso!"
```

#### 4. Criar o arquivo `release.sh`
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR=$(cd "$(dirname "$0")" && pwd)
cd "$ROOT_DIR"

BUMP_TYPE="${1:-}"

if [[ "$BUMP_TYPE" != "major" && "$BUMP_TYPE" != "minor" && "$BUMP_TYPE" != "patch" ]]; then
  echo "Uso: ./release.sh <major|minor|patch>" >&2
  exit 1
fi

if [[ -n "$(git status --porcelain)" ]]; then
  echo "Erro: Limpe o git status antes de iniciar a release." >&2
  exit 1
fi

echo "Executando flutter analyze e testes..."
flutter analyze
flutter test --no-pub

VERSION_STR=$(grep -m1 '^version:' pubspec.yaml | awk '{print $2}')
VERSION_ONLY=${VERSION_STR%%+*}
BUILD_NUMBER=${VERSION_STR##*+}

IFS='.' read -r MAJOR MINOR PATCH <<< "$VERSION_ONLY"

case "$BUMP_TYPE" in
  major) MAJOR=$((MAJOR + 1)); MINOR=0; PATCH=0 ;;
  minor) MINOR=$((MINOR + 1)); PATCH=0 ;;
  patch) PATCH=$((PATCH + 1)) ;;
esac

BUILD_NUMBER=$(( $(git rev-list --count HEAD) + 1 ))
NEW_VERSION="$MAJOR.$MINOR.$PATCH+$BUILD_NUMBER"
TAG_NAME="v$NEW_VERSION"

if git rev-parse -q --verify "refs/tags/$TAG_NAME" >/dev/null; then
  echo "Tag ja existe: $TAG_NAME" >&2
  exit 1
fi

sed -i "s/^version: .*/version: $NEW_VERSION/" pubspec.yaml

git add pubspec.yaml
git commit -m "chore: bump version to $NEW_VERSION"
git tag "$TAG_NAME"
git push origin HEAD
git push origin "$TAG_NAME"

echo "Release do Flutter $TAG_NAME enviada com sucesso!"
```

#### 5. Dar permissão e ativar
```bash
chmod +x install.sh release.sh scripts/pre-commit
./install.sh
```

---

### Angular / React / Node.js (`package.json`)

Em projetos baseados em Node.js (Angular, React, Vue, NestJS), a versão é armazenada no arquivo `package.json`.

#### 1. Criar a pasta `scripts`
```bash
mkdir -p scripts
```

#### 2. Criar o arquivo `scripts/pre-commit`
```bash
#!/bin/bash
set -euo pipefail

# Garante que o package.json seja adicionado ao commit caso tenha sido alterado
git add package.json package-lock.json 2>/dev/null || true
```

#### 3. Criar o arquivo `install.sh`
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR=$(cd "$(dirname "$0")" && pwd)
cd "$ROOT_DIR"

chmod +x scripts/pre-commit
git config core.hooksPath scripts

echo "core.hooksPath configurado para scripts com sucesso!"
```

#### 4. Criar o arquivo `release.sh`
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR=$(cd "$(dirname "$0")" && pwd)
cd "$ROOT_DIR"

BUMP_TYPE="${1:-}"

if [[ "$BUMP_TYPE" != "major" && "$BUMP_TYPE" != "minor" && "$BUMP_TYPE" != "patch" ]]; then
  echo "Uso: ./release.sh <major|minor|patch>" >&2
  exit 1
fi

if [[ -n "$(git status --porcelain)" ]]; then
  echo "Erro: Limpe as alteracoes pendentes antes de gerar release." >&2
  exit 1
fi

echo "Executando testes e linter..."
npm run lint || true
npm test -- --watch=false || true

# Incrementa versao no package.json via npm
NEW_VERSION=$(npm version "$BUMP_TYPE" --no-git-tag-version)
TAG_NAME="$NEW_VERSION"

echo "Nova versao gerada: $NEW_VERSION"

git add package.json package-lock.json
git commit -m "chore: bump version to $NEW_VERSION"
git tag "$TAG_NAME"
git push origin HEAD
git push origin "$TAG_NAME"

echo "Release do Angular/Node $TAG_NAME concluida!"
```

#### 5. Dar permissão e ativar
```bash
chmod +x install.sh release.sh scripts/pre-commit
./install.sh
```

---

### Java Spring Boot (Maven `pom.xml` ou Gradle)

#### Para projetos Maven (`pom.xml`):

##### 1. Criar a pasta `scripts` e o `install.sh`
```bash
mkdir -p scripts
chmod +x scripts
```

##### 2. Criar o arquivo `release.sh` para Maven
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR=$(cd "$(dirname "$0")" && pwd)
cd "$ROOT_DIR"

BUMP_TYPE="${1:-patch}"

if [[ "$BUMP_TYPE" != "major" && "$BUMP_TYPE" != "minor" && "$BUMP_TYPE" != "patch" ]]; then
  echo "Uso: ./release.sh <major|minor|patch>" >&2
  exit 1
fi

if [[ -n "$(git status --porcelain)" ]]; then
  echo "Erro: Limpe o projeto antes de gerar release." >&2
  exit 1
fi

echo "Executando testes unitarios no Maven..."
mvn test

CURRENT_VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)

IFS='.' read -r MAJOR MINOR PATCH <<< "$CURRENT_VERSION"

case "$BUMP_TYPE" in
  major) MAJOR=$((MAJOR + 1)); MINOR=0; PATCH=0 ;;
  minor) MINOR=$((MINOR + 1)); PATCH=0 ;;
  patch) PATCH=$((PATCH + 1)) ;;
esac

NEW_VERSION="$MAJOR.$MINOR.$PATCH"
mvn versions:set -DnewVersion="$NEW_VERSION" -DgenerateBackupPoms=false

git add pom.xml
git commit -m "chore(java): release v$NEW_VERSION"
git tag "v$NEW_VERSION"
git push origin HEAD
git push origin "v$NEW_VERSION"

echo "Release Java Maven v$NEW_VERSION enviada com sucesso!"
```

#### Para projetos Gradle (`build.gradle`):
No Gradle, a versão fica em `version = '1.0.0'` no `build.gradle`. O `release.sh` executa `./gradlew check test` e atualiza a linha `version = ...` usando `sed`.

---

## 5. Como Instalar/Ativar em um Projeto Clonado ou Existente

Se você **já clonou** um repositório que possui os scripts (`install.sh`, `release.sh`, `scripts/pre-commit`), você não precisa criar nada!

Basta rodar **um único comando** na raiz do projeto:

```bash
./install.sh
```

### O que esse comando faz?
1. Garante permissão de execução aos scripts (`chmod +x`).
2. Configura o Git local (`git config core.hooksPath scripts`) para que o hook de `pre-commit` passe a rodar automaticamente na sua máquina.

> 💡 **Lembrete**: Essa configuração do Git é local por clone. Sempre que clonar o projeto em uma nova máquina ou pasta, execute `./install.sh` uma única vez.

---

## 6. Como usar no dia a dia (Guia Prático)

### 1. Trabalho Diário (Commits)
Desenvolva seu código normalmente:
```bash
git add .
git commit -m "feat: adicionado filtro de busca"
git push origin main
```
* O Git hook ajusta e adiciona o arquivo de versão automaticamente antes de fechar o commit.

### 2. Gerar uma Release Oficial
Quando o código for publicado em homologação ou produção:
```bash
# Para correções de bugs (ex: 1.0.0 -> 1.0.1):
./release.sh patch

# Para novas funcionalidades compatíveis (ex: 1.0.0 -> 1.1.0):
./release.sh minor

# Para grandes reformulações ou quebras de compatibilidade (ex: 1.0.0 -> 2.0.0):
./release.sh major
```

---

## 7. Perguntas Frequentes e Resolução de Problemas

### 1. "Permission denied" ao tentar rodar um `.sh`
Dê permissão de execução executando:
```bash
chmod +x install.sh release.sh scripts/pre-commit
```

### 2. Como verificar se o hook está ativo?
Execute no terminal:
```bash
git config core.hooksPath
```
Se retornar `scripts`, está ativo. Para testar na prática, faça um commit e verifique se o arquivo de versão foi alterado automaticamente.

### 3. O `release.sh` falhou nos testes, e agora?
O script é projetado para **não prosseguir** se houver erro nos testes ou no linter. Corrija os testes que falharam, faça commit das correções e tente rodar o `./release.sh` novamente.

### 4. O meu projeto não é Flutter, nem Angular, nem Java. Como usar?
Basta seguir a **Opção A (Projeto Genérico)** descrita na Seção 4, que utiliza o arquivo `VERSION`.

### 5. Preciso rodar `install.sh` de novo se fizer `git clone` novamente?
**Sim.** A configuração `core.hooksPath` é local a cada clone do repositório. Sempre que clonar o projeto em uma máquina nova ou em outra pasta, execute `./install.sh` uma única vez.

### 6. Como rodar no Windows?
No Windows, utilize o terminal **Git Bash** (instalado junto com o Git) ou o **WSL** (Windows Subsystem for Linux) para executar os comandos `.sh`.
