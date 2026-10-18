---
sidebar_position: 2
title: "Instalação"
description: "Instale o Hermes Agent com os pacotes desktop, os instaladores a partir do código-fonte, Docker, Nix ou o pacote APT do Termux"
---

# Instalação

Coloque o Hermes Agent para funcionar em menos de dois minutos!

:::tip Suporte a plataformas
Para a matriz completa de suporte a plataformas (quais sistemas operacionais, métodos de
distribuição e recursos restritos a cada plataforma são suportados), consulte **[Suporte a plataformas](./platform-support.md)**.
:::

## Instalação rápida {#quick-install}
### Pacotes desktop no macOS ou no Windows {#desktop-packages-on-macos-or-windows}

Baixe o pacote da sua plataforma no
[site do Hermes](https://hermes-agent.nousresearch.com/).

- **Windows:** abra o arquivo `.appinstaller` baixado com o Windows App Installer.
  Ele instala o pacote MSIX assinado e registra a origem das atualizações.
  Pacotes da Microsoft Store são gerenciados separadamente pela Store.
- **macOS:** abra o DMG e copie o `Hermes.app` para Aplicativos. O artefato
  ZIP contém o aplicativo assinado usado pelo atualizador automático.

Os pacotes incluem o agente, o Python, as dependências suportadas e as interfaces
já compiladas. A primeira execução não compila esse runtime base. O acesso aos
provedores e as integrações opcionais ainda podem exigir acesso à rede.

O instalador de bootstrap `Hermes-Setup` é diferente: ele baixa uma instalação a
partir do código-fonte e compila o aplicativo desktop. Consulte [Hermes Desktop](../user-guide/desktop.md).

Não existe um download do Desktop exclusivo para uso remoto. Para usar o Desktop com um
backend do Hermes em outra máquina, instale um dos pacotes acima e conecte-o
em **Settings → Gateways**; consulte
[Conectar a um backend remoto](../user-guide/desktop.md#connecting-to-a-remote-backend).

:::note
O instalador para macOS é **somente para Apple Silicon**. O macOS em processadores x86 (Intel) [não é uma plataforma suportada](./platform-support.md#unsupported).
:::

### Sem o Hermes Desktop: {#without-hermes-desktop}
Para instalar somente a linha de comando, sem o Hermes Desktop, execute:

#### Linux / macOS / WSL2 {#linux--macos--wsl2}
```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

#### Windows (nativo) {#windows-native}

Execute no PowerShell:
```powershell
iex (irm https://hermes-agent.nousresearch.com/install.ps1) 
```

Se quiser instalar e executar o Hermes Desktop depois de uma instalação somente de linha de comando, basta executar
```bash
hermes desktop
```

### Android / Termux {#android--termux}

Use o [pacote APT do Termux](./termux.md) em dispositivos Android aarch64.
Configure o repositório assinado dele antes de executar `pkg install hermes-agent`.
Os scripts de desktop/servidor não são o caminho de instalação no Termux.

### O que o instalador a partir do código-fonte faz {#what-the-source-installer-does}

Os scripts clonam o código-fonte, fazem o bootstrap do uv e delegam a preparação
das dependências ao PM. O PM fornece versões fixadas de Python, Node.js, npm, ripgrep e FFmpeg. A
instalação a partir do código-fonte seleciona o extra `all` do Python, e não todos os extras opcionais.
O PM também instala por padrão as ferramentas de navegador e de uso do computador: o `agent-browser`
com seu Chromium fixado e o `cua-driver` (o driver de uso do computador, no macOS,
no Windows e no Linux com glibc). Se um download falhar, a instalação ainda é concluída e
mostra o comando para tentar de novo. O driver de navegador padrão (browser-harness, o
motor do Browser Use CLI) é uma dependência Python comum, então toda instalação,
inclusive a do aplicativo Desktop, já o inclui.
As demais ferramentas opcionais usam os caminhos de instalação próprios de cada recurso.

Uma instalação nova fica na versão estável mais recente, o canal que o `hermes update`
segue a partir de então (consulte [Atualização](./updating.md)). Para instalar a ponta de
`main`, passe `--channel main` no POSIX ou `-Channel main` no Windows
(ou defina `HERMES_INSTALL_CHANNEL=main` antes, para os comandos de uma linha).

Para deixar de fora as ferramentas de navegador, passe `--skip-browser` no POSIX ou `-SkipBrowser`
no Windows; para o driver de uso do computador, `--skip-computer-use` /
`-SkipComputerUse`. O Hermes lembra dessas escolhas: instalações posteriores e o
`hermes update` não as adicionam de volta. Execute `hermes pm install agent-browser` ou
`hermes pm install cua-driver` para instalá-las e desfazer a escolha.

Os scripts criam um inicializador e preparam o diretório de dados. Execuções interativas
também chamam a configuração inicial e a configuração do gateway. `--non-interactive` no POSIX, ou
`-NonInteractive` no Windows, pula as etapas que exigem entrada do usuário. A etapa opcional
`--include-desktop` / `-IncludeDesktop` compila o desktop a partir do código-fonte.

Em um terminal, os scripts mostram uma linha de status por etapa e gravam a saída
do git, do uv e das compilações em `logs/install.log`, dentro do diretório de dados
do Hermes; uma etapa que falha mostra suas últimas linhas e o caminho do log. Em CI (`CI` ou
`GITHUB_ACTIONS` definido), com saída redirecionada, `--verbose` / `-Verbose` ou
`HERMES_INSTALL_VERBOSE=1`, tudo é exibido por completo.

#### Estrutura da instalação {#install-layout}

| Método | Código | Ponto de entrada da CLI | Dados padrão do usuário |
|---|---|---|---|
| Script POSIX a partir do código-fonte | `~/.hermes/hermes-agent/` | Wrapper `~/.local/bin/hermes` | `~/.hermes/` |
| Script Windows a partir do código-fonte | `%LOCALAPPDATA%\hermes\hermes-agent\` | `%LOCALAPPDATA%\hermes\bin\` | `%LOCALAPPDATA%\hermes\` |
| Pacote desktop | Dentro do pacote do aplicativo instalado | Inicializadores empacotados; aliases de execução no Windows | Diretório de dados padrão do Hermes na plataforma |
| Docker | `/opt/hermes/` | Entrypoint da imagem e shim `hermes` | `/opt/data/` montado |
| APT do Termux | `$PREFIX/lib/hermes-agent/` | Links simbólicos em `$PREFIX/bin/` | `~/.hermes/` |

`HERMES_HOME` seleciona os dados do usuário. O `--dir` do script POSIX seleciona o
checkout do código-fonte de forma independente. O Windows oferece `-HermesHome` e `-InstallDir`.
Executar o script POSIX como root não seleciona automaticamente uma estrutura FHS:
ele usa o diretório home do root, a menos que você informe um caminho explícito para o código-fonte.

O repositório de ferramentas do PM e as gerações de Python de cada instalação têm ciclos de vida separados.
Consulte [Gerenciamento de pacotes](../reference/package-management.md) para saber onde ficam.
Não remova a raiz de dados para reparar a instalação do aplicativo.

### Depois da instalação {#after-installation}

Recarregue o shell e comece a conversar:

```bash
source ~/.bashrc   # or: source ~/.zshrc
hermes             # Start chatting!
```

Para reconfigurar opções individuais depois, use os comandos dedicados:

```bash
hermes model          # Choose your LLM provider and model
hermes tools          # Configure which tools are enabled
hermes gateway setup  # Set up messaging platforms
hermes config set     # Set individual config values
hermes config get     # Inspect individual config values
hermes setup          # Or run the full setup wizard to configure everything at once
```

:::tip Caminho mais rápido: Nous Portal
Uma única assinatura cobre mais de 300 modelos e também o [Tool Gateway](../user-guide/features/tool-gateway.md) (busca na web, geração de imagens, TTS, navegador na nuvem). Sem precisar gerenciar uma chave para cada ferramenta:

```bash
hermes setup --portal
```

Esse comando faz o seu login, define a Nous como provedor e ativa o Tool Gateway de uma só vez.
:::

:::tip Já usa o Hermes em outra máquina?
Você não precisa refazer a sua configuração do zero. Restaure um backup completo com `hermes import` (consulte [Exportar o Hermes para outra máquina](../reference/faq.md#exporting-hermes-to-another-machine)) ou traga um único agente com `hermes profile import` (consulte [Mover um único perfil para outra máquina](../reference/faq.md#moving-a-single-profile-to-another-machine)). Observe que a exportação de perfil exclui as credenciais de propósito, então uma exportação sozinha não é um backup completo — [`hermes backup` vs. `hermes profile export`](../reference/faq.md#hermes-backup-vs-hermes-profile-export) explica qual usar.
:::

---

## Pré-requisitos {#prerequisites}

Para o script POSIX a partir do código-fonte, tenha Git, curl, tar e utilitários de SHA-256.
O Windows consegue baixar seu próprio arquivo fixado do Git for Windows quando o Git não está presente.
O script sempre baixa a versão verificada e fixada do uv; um uv que já esteja no seu PATH nunca é usado.

As instalações oficiais atuais rodam em **Python 3.14**. O intervalo mais amplo
`>=3.11,<3.15` no `pyproject.toml` permite que instalações com Python mais antigo
executem o atualizador antes que o PM as migre para o 3.14; isso não garante suporte
atual ao runtime no 3.11–3.13. O PM seleciona as versões das ferramentas gerenciadas a partir de
`pm/lock.json`; ele não adota versões arbitrárias do Node do sistema como
runtime instalado.

Compilações a partir do código-fonte podem exigir um compilador nativo e bibliotecas de desenvolvimento da plataforma.
Compilar o Electron a partir do código-fonte acrescenta os requisitos de módulos nativos do Node. Esses
pré-requisitos de compilação não se aplicam à instalação de um pacote desktop completo.
O Chromium no Linux também exige bibliotecas do sistema fornecidas pela distribuição.

No Linux com glibc, o Node.js gerenciado depende de `libatomic.so.1`, que imagens mínimas de
Debian, Ubuntu e da família RHEL não trazem. Quando a biblioteca está
ausente, o instalador e o `hermes update` instalam o pacote da distribuição
(`libatomic1` no apt e no zypper, `libatomic` no dnf/yum e no apk, `gcc-libs`
no pacman). Eles executam o gerenciador de pacotes diretamente como root ou, caso contrário,
com `sudo -n`. Uma execução interativa pede sua senha do sudo uma vez,
antes de instalar as dependências. Execuções com `--non-interactive` nunca pedem senha. Se
a instalação não puder ser executada, o erro informa o comando exato para o gerenciador
de pacotes encontrado.

:::tip Usuários de Nix
O Nix **não é mais um caminho de instalação com suporte explícito** (apenas melhor esforço). Se você já usa Nix (no NixOS, no macOS ou no Linux), existe um caminho de configuração dedicado com um flake do Nix, um módulo declarativo do NixOS e um modo de contêiner opcional. Consulte o guia **[Configuração do Nix e do NixOS](./nix-setup.md)**.
:::

---

## Instalação manual / para desenvolvimento {#manual--developer-installation}

Para um checkout do código-fonte, comece pelo
[fluxo de trabalho de desenvolvimento do PM](../reference/package-management.md#developer-workflow).
Ele aborda a ativação, os comandos do dia a dia, a atualização de dependências e as limitações atuais do bootstrap.
[Configuração do ambiente de desenvolvimento](../developer-guide/contributing.md#development-setup) aborda o ambiente de testes separado e as verificações.

---

## Instalações sem sudo / com usuário de serviço do sistema {#non-sudo--system-service-user-installs}

Execute o instalador a partir do código-fonte como o usuário de serviço pretendido. O diretório home, o repositório de ferramentas,
a configuração e o inicializador precisam pertencer a esse usuário.

1. Como administrador, instale os pré-requisitos de compilação a partir do código-fonte e as bibliotecas
   do Linux exigidas pelo backend de navegador escolhido.
2. Como usuário de serviço, execute o instalador normal:

   ```bash
   curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
   ```

3. Adicione o diretório real do inicializador ao ambiente do shell do usuário de serviço:

   ```bash
   export PATH="$HOME/.local/bin:$PATH"
   ```

4. Execute `hermes doctor` a partir dessa conta. Use o wrapper instalado, e não um
   caminho fixo `venv/bin/hermes`.
5. Para um serviço de usuário no Linux que precise continuar rodando após o logout, ative o lingering como administrador:

   ```bash
   sudo loginctl enable-linger SERVICE_USER
   ```

O instalador atual a partir do código-fonte não executa a etapa `--with-deps` do Playwright.
Fora a `libatomic` do Node.js (consulte Pré-requisitos), ele não instala
pacotes do sistema. O PM gerencia os binários das ferramentas; o administrador fornece as bibliotecas
do sistema. Um usuário de serviço sem sudo recebe o comando exato da `libatomic` para
pedir a um administrador que o execute. Consulte
[Automação de navegador](../user-guide/features/browser.md) e
[Gateway de mensagens](../user-guide/messaging/index.md).

---

## Solução de problemas {#troubleshooting}

| Problema | Solução |
|---------|----------|
| `hermes: command not found` | Recarregue o shell (`source ~/.bashrc`) ou verifique o PATH |
| `API key not set` | Execute `hermes model` para configurar o provedor, ou `hermes config set OPENROUTER_API_KEY your_key` |
| Configuração ausente após uma atualização | Execute `hermes config check` e depois `hermes config migrate` |

Para mais diagnósticos, execute `hermes doctor` — ele mostra exatamente o que está faltando e como corrigir.

### Diretórios home com links simbólicos e armazenamento externo {#symlinked-home-directories-and-external-storage}

O Hermes suporta um `HERMES_HOME` com link simbólico e subdiretórios do home com links simbólicos,
incluindo `hooks`, `skills`, `sessions` e `logs`. Durante a inicialização do home,
os links de diretório existentes são preservados, e as permissões dos diretórios vinculados
(e de seus descendentes, como `logs/curator`) ficam a cargo do dono.

Se o destino de um link estiver ausente, inacessível ou não for um diretório, a inicialização
é interrompida com um erro de armazenamento que informa o caminho e o destino do link. O Hermes **não**
substitui o link nem cria o destino ausente: isso poderia gravar dados no
disco local enquanto um volume externo ou de NAS estiver desmontado. Verifique o link
informado, restaure a montagem ou corrija o destino e confira as permissões de acesso
antes de tentar de novo. Para um destino de dotfiles criado de propósito, crie-o você mesmo somente
depois de confirmar que o armazenamento pretendido está disponível.

O `hermes doctor` relata essas falhas como problemas de armazenamento, e não como YAML inválido.
Mantenha o seu `config.yaml` atual; executar `hermes setup` não é a solução para um
diretório indisponível. Esta é uma verificação de disponibilidade de diretório, e não um monitor de montagem:
um diretório existente não garante que o volume pretendido esteja montado.

## Detecção automática do método de instalação {#install-method-auto-detection}

Quem cuida das atualizações depende da instalação em execução, e não apenas do seu diretório de dados.
Checkouts do código-fonte usam o caminho de atualização gerenciado pelo Git. Pacotes desktop, Docker,
Nix e pacotes do Termux mantêm o mecanismo de atualização do respectivo pacote.
O `hermes doctor` informa a procedência da instalação. Consulte
[Atualização e desinstalação](./updating.md) antes de alterar arquivos gerenciados por pacotes.
