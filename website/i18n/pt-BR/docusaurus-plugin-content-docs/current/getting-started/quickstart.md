---
sidebar_position: 1
title: "Guia rápido do Hermes Agent"
description: "Sua primeira conversa com o Hermes Agent — da instalação ao chat em menos de 5 minutos"
---

# Guia rápido do Hermes Agent

Os comandos de dependências Python desta página usam um
[checkout do código-fonte preparado pelo PM](../reference/package-management.md#developer-workflow).
Depois de alterar uma dependência, reative o checkout e reinicie o Hermes.

Este guia leva você do zero a uma configuração do Hermes que funciona e aguenta o uso real. Instale, escolha um provedor, confirme que o chat funciona e saiba exatamente o que fazer quando algo der errado.

## Prefere assistir? {#prefer-to-watch}

O **Onchain AI Garage** preparou uma Masterclass que percorre a instalação, a configuração e os comandos básicos — um bom complemento para esta página se você preferir acompanhar em vídeo. Para mais conteúdo, veja a playlist completa [Hermes Agent Tutorials & Use Cases](https://www.youtube.com/playlist?list=PLmpUb_PWAkDxewld5ZYyKifuHxgIbiq2d).

<div style={{position: 'relative', paddingBottom: '56.25%', height: 0, overflow: 'hidden', maxWidth: '100%', marginBottom: '1.5rem'}}>
  <iframe
    style={{position: 'absolute', top: 0, left: 0, width: '100%', height: '100%'}}
    src="https://www.youtube-nocookie.com/embed/R3YOGfTBcQg"
    title="Hermes Agent Masterclass: Installation, Setup, Basic Commands"
    frameBorder="0"
    allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowFullScreen
  ></iframe>
</div>

## Para quem é este guia {#who-this-is-for}

- Para quem está começando agora e quer o caminho mais curto até uma configuração funcionando
- Para quem está trocando de provedor e não quer perder tempo com erros de configuração
- Para quem está configurando o Hermes para uma equipe, um bot ou um fluxo sempre ativo
- Para quem cansou de "instalou, mas continua sem fazer nada"

## O caminho mais rápido {#the-fastest-path}

Escolha a linha que corresponde ao seu objetivo:

| Objetivo | Faça isto primeiro | Depois faça isto |
|---|---|---|
| Só quero o Hermes funcionando na minha máquina | `hermes setup` | Faça um chat de verdade e confirme que ele responde |
| Já sei qual provedor vou usar | `hermes model` | Salve a configuração e comece a conversar |
| Quero um bot ou uma configuração sempre ativa | `hermes gateway setup` depois que a CLI funcionar | Conecte o Telegram, o Discord, o Slack ou outra plataforma |
| Quero um modelo local ou auto-hospedado | `hermes model` → endpoint personalizado | Confirme o endpoint, o nome do modelo e o tamanho do contexto |
| Quero fallback entre vários provedores | `hermes model` primeiro | Adicione roteamento e fallback só depois que o chat básico funcionar |

**Regra prática:** se o Hermes não consegue concluir um chat normal, não adicione mais recursos ainda. Primeiro faça uma conversa simples funcionar e só depois acrescente gateway, cron, skills, voz ou roteamento.

---

## 1. Instale o Hermes Agent {#1-install-hermes-agent}
### Com o instalador do Hermes Desktop no macOS ou no Windows (recomendado) {#with-the-hermes-desktop-installer-on-macos-or-windows-recommended}
Para instalar facilmente os aplicativos de linha de comando e desktop, [baixe o instalador do Hermes Desktop](https://hermes-agent.nousresearch.com/) no nosso site e execute-o.

### Sem o Hermes Desktop: {#without-hermes-desktop}
Para instalar somente a linha de comando, sem o Hermes Desktop, execute:

Em dispositivos Android aarch64, use o [guia do APT do Termux](./termux.md), que é separado.

#### Linux / macOS / WSL2 {#linux--macos--wsl2}
```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

#### Windows (nativo) {#windows-native}

Execute no PowerShell:
```powershell
iex (irm https://hermes-agent.nousresearch.com/install.ps1) 
```

Quando terminar, recarregue o shell:

```bash
source ~/.bashrc   # or source ~/.zshrc
```

Para opções de instalação detalhadas, pré-requisitos e solução de problemas, consulte o [guia de instalação](./installation.md).

## 2. Escolha um provedor {#2-choose-a-provider}

Esta é a etapa mais importante da configuração. Use `hermes model` para fazer a escolha de forma interativa:

```bash
hermes model
```

:::tip Caminho mais fácil: Nous Portal
Uma única assinatura cobre mais de 300 modelos e também o [Tool Gateway](../user-guide/features/tool-gateway.md) (busca na web, geração de imagens, TTS, navegador na nuvem). Em uma instalação nova:

```bash
hermes setup --portal
```

Esse comando faz o seu login, define a Nous como provedor e ativa o Tool Gateway de uma só vez.
:::

:::info Modos de configuração
Em uma instalação nova, o `hermes setup` oferece três modos:

- **Quick Setup (Nous Portal)** — login via OAuth, sem chaves de API para gerenciar; configura um modelo e as ferramentas do Tool Gateway, cobrados na sua [assinatura do Nous Portal](../integrations/nous-portal.md). É o caminho rápido recomendado.
- **Full Setup** — você mesmo percorre cada provedor, ferramenta e opção (usando as suas próprias chaves).
- **Blank Slate** — tudo começa **desativado**, exceto o mínimo necessário para rodar um agente: **provedor e modelo, o toolset File Operations e o toolset Terminal**. Nada de web, navegador, execução de código, visão, memória, delegação, cron, skills, plugins ou servidores MCP — e compressão, checkpoints, roteamento inteligente e captura de memória ficam todos desativados. Depois que essa base mínima é aplicada, você escolhe entre dois caminhos: **começar com tudo desativado** (terminar agora com o agente mínimo) ou **percorrer todas as configurações** (ativar ferramentas, skills, plugins, MCP e mensagens). Escolha este modo quando quiser um agente mínimo e totalmente controlado e pretender ativar exatamente só o que precisa.

O Blank Slate grava uma lista explícita em `platform_toolsets.cli` e também `agent.disabled_toolsets`, para que nada que você não escolheu seja carregado — nem mesmo depois de um `hermes update`. Reative o que quiser depois com `hermes tools`, adicione skills com `hermes skills opt-in --sync` ou ajuste as configurações com `hermes setup agent`.
:::

Boas opções padrão:

| Provedor | O que é | Como configurar |
|----------|-----------|---------------|
| **Nous Portal** | Por assinatura, sem configuração | Login via OAuth com `hermes model` |
| **OpenAI Codex** | Assinatura do ChatGPT ou do Codex, usa modelos Codex | Autenticação por código de dispositivo com `hermes model` → **ChatGPT or Codex Subscription** |
| **Anthropic** | Modelos Claude diretamente — plano Max + créditos de uso extra (OAuth), ou chave de API com pagamento por token | `hermes model` → login via OAuth (exige Max + créditos extras), ou uma chave de API da Anthropic |
| **OpenRouter** | Roteamento entre vários provedores e muitos modelos | Informe a sua chave de API |
| **Fireworks AI** | API de modelos direta, compatível com OpenAI | Defina `FIREWORKS_API_KEY` |
| **Z.AI** | Modelos GLM / hospedados pela Zhipu | Defina `GLM_API_KEY` / `ZAI_API_KEY` (também aceita `Z_AI_API_KEY`) |
| **Kimi / Moonshot** | Modelos de programação e chat hospedados pela Moonshot | Defina `KIMI_API_KEY` (ou a `KIMI_CODING_API_KEY`, específica do Kimi Coding) |
| **Kimi / Moonshot China** | Endpoint da Moonshot na região da China | Defina `KIMI_CN_API_KEY` |
| **Arcee AI** | Modelos Trinity | Defina `ARCEEAI_API_KEY` |
| **GMI Cloud** | API direta com vários modelos | Defina `GMI_API_KEY` |
| **Actual Computer** | Seu próprio hardware como cluster privado de inferência — relay hospedado ou daemon local | Defina `ACTUAL_API_KEY` (relay) ou `ACTUAL_BASE_URL=http://127.0.0.1:8080` (local, sem chave) |
| **MiniMax (OAuth)** | Modelo de ponta da MiniMax via OAuth no navegador — sem chave de API (o nome do modelo em `hermes_cli/models.py` pode mudar entre versões) | `hermes model` → MiniMax (OAuth) |
| **MiniMax** | Endpoint internacional da MiniMax | Defina `MINIMAX_API_KEY` |
| **MiniMax China** | Endpoint da MiniMax na região da China | Defina `MINIMAX_CN_API_KEY` |
| **Alibaba Cloud** | Modelos Qwen via DashScope | Defina `DASHSCOPE_API_KEY` (o Qwen Coding Plan também aceita `ALIBABA_CODING_PLAN_API_KEY`) |
| **Hugging Face** | Mais de 20 modelos abertos por um roteador unificado (Qwen, DeepSeek, Kimi etc.) | Defina `HF_TOKEN` |
| **AWS Bedrock** | Claude, Nova, Llama, DeepSeek pela API Converse nativa | Role do IAM ou `aws configure` ([guia](../guides/aws-bedrock.md)) |
| **Azure Foundry** | Modelos hospedados no Azure AI Foundry | Defina `AZURE_FOUNDRY_API_KEY` + `AZURE_FOUNDRY_BASE_URL` |
| **Google AI Studio** | Modelos Gemini via API direta | Defina `GOOGLE_API_KEY` / `GEMINI_API_KEY` |
| **xAI** | Modelos Grok via API direta | Defina `XAI_API_KEY` |
| **xAI Grok OAuth** | Assinatura SuperGrok / Premium+, sem chave de API | `hermes model` → xAI Grok OAuth |
| **NovitaAI** | Gateway de API com vários modelos | Defina `NOVITA_API_KEY` |
| **Ramp Router** | Gateway de LLM nativo da Responses API, com roteamento entre OpenAI/Anthropic/xAI/... | Defina `RAMP_ROUTER_API_KEY` |
| **Nebius Token Factory** | Modelos abertos na nuvem Nebius AI | Defina `NEBIUS_API_KEY` |
| **StepFun** | Modelos do Step Plan | Defina `STEPFUN_API_KEY` |
| **Xiaomi MiMo** | Modelos hospedados pela Xiaomi | Defina `XIAOMI_API_KEY` |
| **Tencent TokenHub** | Modelos hospedados pela Tencent | Defina `TOKENHUB_API_KEY` |
| **Tencent TokenPlan** | Modelos Tencent Hy via endpoint no estilo da Anthropic | Defina `TOKENPLAN_API_KEY` |
| **Ollama Cloud** | Modelos gerenciados hospedados pelo Ollama | Defina `OLLAMA_API_KEY` |
| **LM Studio** | Aplicativo desktop local que expõe uma API compatível com OpenAI | Defina `LM_API_KEY` (e `LM_BASE_URL`, se não for o padrão) |
| **Qwen OAuth** | OAuth do Qwen Portal no navegador — sem chave de API | `hermes model` → Qwen OAuth |
| **Kilo Code** | Modelos hospedados pela KiloCode | Defina `KILOCODE_API_KEY` |
| **OpenCode Zen** | Acesso pré-pago por uso a modelos selecionados | Defina `OPENCODE_ZEN_API_KEY` |
| **OpenCode Go** | Assinatura de US$ 10/mês para modelos abertos | Defina `OPENCODE_GO_API_KEY` |
| **DeepSeek** | Acesso direto à API da DeepSeek | Defina `DEEPSEEK_API_KEY` |
| **NVIDIA NIM** | Modelos Nemotron via build.nvidia.com ou NIM local | Defina `NVIDIA_API_KEY` (opcional: `NVIDIA_BASE_URL`) |
| **GitHub Copilot** | Assinatura do GitHub Copilot (GPT-5.x, Claude, Gemini etc.) | OAuth com `hermes model`, ou `COPILOT_GITHUB_TOKEN` / `GH_TOKEN` |
| **GitHub Copilot ACP** | Backend de agente Copilot ACP (inicia a CLI `copilot` local) | `hermes model` (exige a CLI `copilot` + `copilot login`) |
| **Vercel AI Gateway** | Roteamento pelo Vercel AI Gateway | Defina `AI_GATEWAY_API_KEY` |
| **Custom Endpoint** | VLLM, SGLang, Ollama ou qualquer API compatível com OpenAI | Defina a URL base + a chave de API |

Para a maioria dos usuários de primeira viagem: escolha um provedor e aceite os padrões, a menos que você saiba por que está mudando. O catálogo completo de provedores, com variáveis de ambiente e etapas de configuração, está na página [Provedores](../integrations/providers.md).

:::caution Contexto mínimo: 64K tokens
O Hermes Agent exige um modelo com pelo menos **64.000 tokens** de contexto. Modelos com janelas menores não conseguem manter memória de trabalho suficiente para fluxos com várias etapas de chamadas de ferramentas e são rejeitados na inicialização. A maioria dos modelos hospedados (Claude, GPT, Gemini, Qwen, DeepSeek) atende a esse requisito com folga. Se você usa um modelo local, defina o tamanho do contexto para pelo menos 64K (por exemplo, `--ctx-size 65536` no llama.cpp ou `-c 65536` no Ollama).
:::

:::tip
Você pode trocar de provedor a qualquer momento com `hermes model` — sem amarras. Para a lista completa de provedores suportados e os detalhes de configuração, consulte [Provedores de IA](../integrations/providers.md).
:::

### Como as configurações são armazenadas {#how-settings-are-stored}

O Hermes separa os segredos da configuração comum:

- **Segredos e tokens** → `~/.hermes/.env`
- **Configurações que não são segredos** → `~/.hermes/config.yaml`

A forma mais fácil de definir os valores corretamente é pela CLI:

```bash
hermes config set model anthropic/claude-opus-4.6
hermes config set terminal.backend docker
hermes config set OPENROUTER_API_KEY sk-or-...
```

Cada valor vai automaticamente para o arquivo certo.

## 3. Faça o seu primeiro chat {#3-run-your-first-chat}

```bash
hermes            # classic CLI
hermes --tui      # modern TUI (recommended)
```

Você verá um banner de boas-vindas com o seu modelo, as ferramentas disponíveis e as skills. Use um prompt específico e fácil de verificar:

:::tip Escolha a sua interface
O Hermes vem com duas interfaces de terminal: a CLI clássica com `prompt_toolkit` e uma [TUI](../user-guide/tui.md) mais nova, com sobreposições modais, seleção com o mouse e entrada sem bloqueio. As duas compartilham as mesmas sessões, comandos de barra e configurações — experimente cada uma com `hermes` e `hermes --tui`.
:::

```
Summarize this repo in 5 bullets and tell me what the main entrypoint is.
```

```
Check my current directory and tell me what looks like the main project file.
```

```
Help me set up a clean GitHub PR workflow for this codebase.
```

**Como saber que deu certo:**

- O banner mostra o modelo/provedor escolhido
- O Hermes responde sem erro
- Ele consegue usar uma ferramenta quando necessário (terminal, leitura de arquivos, busca na web)
- A conversa continua normalmente por mais de uma rodada

Se isso funcionou, você já passou pela parte mais difícil.

## 4. Confirme que as sessões funcionam {#4-verify-sessions-work}

Antes de seguir em frente, confirme que a retomada funciona:

```bash
hermes --continue    # Resume the most recent session
hermes -c            # Short form
```

Isso deve levar você de volta à sessão que acabou de usar. Se não levar, verifique se você está no mesmo perfil e se a sessão foi realmente salva. Isso faz diferença mais tarde, quando você estiver lidando com várias configurações ou máquinas.

## 5. Experimente os principais recursos {#5-try-key-features}

### Use o terminal {#use-the-terminal}

```
❯ What's my disk usage? Show the top 5 largest directories.
```

O agente executa comandos no terminal por você e mostra os resultados.

### Comandos de barra {#slash-commands}

Digite `/` para ver uma lista de autocompletar com todos os comandos:

| Comando | O que faz |
|---------|-------------|
| `/help` | Mostra todos os comandos disponíveis |
| `/tools` | Lista as ferramentas disponíveis |
| `/model` | Troca de modelo de forma interativa |
| `/personality pirate` | Experimenta uma personalidade divertida |
| `/save` | Salva a conversa |

### Entrada com várias linhas {#multi-line-input}

Pressione `Alt+Enter`, `Ctrl+J` ou `Shift+Enter` para adicionar uma nova linha. O `Shift+Enter` exige um terminal que o envie como uma sequência distinta (Kitty / foot / WezTerm / Ghostty por padrão; iTerm2 / Alacritty / terminal do VS Code depois que o protocolo de teclado do Kitty é ativado). `Alt+Enter` e `Ctrl+J` funcionam em qualquer terminal.

### Interrompa o agente {#interrupt-the-agent}

Se o agente estiver demorando demais, digite uma nova mensagem e pressione Enter — isso interrompe a tarefa atual e passa para as suas novas instruções. `Ctrl+C` também funciona.

## 6. Adicione a próxima camada {#6-add-the-next-layer}

Só depois que o chat básico estiver funcionando. Escolha o que você precisa:

### Bot ou assistente compartilhado {#bot-or-shared-assistant}

```bash
hermes gateway setup    # Interactive platform configuration
```

Conecte o [Telegram](../user-guide/messaging/telegram.md), o [Discord](../user-guide/messaging/discord.md), o [Slack](../user-guide/messaging/slack.md), o [WhatsApp](../user-guide/messaging/whatsapp.md), o [Signal](../user-guide/messaging/signal.md), o [e-mail](../user-guide/messaging/email.md), o [Home Assistant](../user-guide/messaging/homeassistant.md) (plugin) ou o [Microsoft Teams](../user-guide/messaging/teams.md).

### Automação e ferramentas {#automation-and-tools}

- `hermes tools` — ajuste o acesso às ferramentas por plataforma
- `hermes skills` — navegue e instale fluxos de trabalho reutilizáveis
- Cron — só depois que o seu bot ou a sua configuração da CLI estiver estável

### Terminal em sandbox {#sandboxed-terminal}

Por segurança, rode o agente em um contêiner Docker ou em um servidor remoto:

```bash
hermes config set terminal.backend docker    # Docker isolation
hermes config set terminal.backend ssh       # Remote server
```

Em sandboxes Docker, você também pode ativar o **proxy de saída com injeção de credenciais**, para que a sandbox nunca veja as suas chaves de API reais — apenas tokens opacos de proxy que só funcionam por trás de um daemon local que intercepta o TLS. Consulte [Proxy de saída](../user-guide/egress/iron-proxy.md). A configuração é `hermes egress setup && hermes egress start`; o `hermes setup terminal` também indica esse recurso para quem usa Docker. Modal, SSH, Daytona e Singularity ainda não estão integrados.

### Modo de voz {#voice-mode}

Execute `hermes tools` e configure os provedores de voz. Depois, ative `/voice on`
na CLI e pressione `Ctrl+B` para gravar. O PM cuida dos requisitos suportados
que estiverem faltando; uma alteração de dependência pode exigir reinicialização. O Faster-Whisper
local não está disponível em todas as arquiteturas. Consulte [Modo de voz](../user-guide/features/voice-mode.md).

### Skills {#skills}

Skills são documentos de instruções sob demanda que ensinam o Hermes a realizar uma tarefa específica — fazer deploy no Kubernetes, abrir um PR no GitHub, fazer fine-tuning de um modelo, buscar GIFs. Cada uma é um arquivo `SKILL.md` com um nome, uma descrição e um procedimento passo a passo. O agente lê as descrições curtas sem custo e só carrega o conteúdo completo de uma skill quando uma tarefa realmente pede, então adicionar skills não incha cada requisição.

O Hermes vem com um catálogo de skills incluídas, já instaladas em `~/.hermes/skills/`. Você pode adicionar mais pelo Skills Hub ou escrever as suas.

**Navegue e instale pelo hub:**

```bash
hermes skills browse                      # list everything available
hermes skills search kubernetes           # find skills by keyword
hermes skills install openai/skills/k8s   # install one (runs a security scan first)
```

O argumento de instalação é um slug `source/path` do hub — `openai/skills/k8s` significa a skill `k8s` do catálogo da OpenAI. O `hermes skills browse` mostra os slugs exatos a usar.

**Use uma skill** — toda skill instalada vira automaticamente um comando de barra:

```bash
/k8s deploy the staging manifest          # run the skill with a request
/k8s                                       # load it and let Hermes ask what you need
```

Isso funciona na CLI e em qualquer plataforma de mensagens conectada. Você não precisa instalar tudo de antemão — durante uma conversa normal, o agente escolhe sozinho a skill incluída certa quando uma tarefa corresponde a ela.

Consulte [Sistema de skills](../user-guide/features/skills.md) para escrever as suas próprias skills, usar diretórios externos de skills e ver a lista completa de fontes do hub.

### Servidores MCP {#mcp-servers}

```yaml
# Add to ~/.hermes/config.yaml
mcp_servers:
  github:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: "ghp_xxx"
```

### Integração com editores (ACP) {#editor-integration-acp}

O suporte a ACP vem com os extras padrão `[all]`, então o instalador via curl já o inclui. Basta executar:

```bash
hermes acp
```

(Se você instalou sem `[all]`, execute antes `cd ~/.hermes/hermes-agent && python -c "import pm; pm.sync_venv(['acp'], explicit=True)"`.)

Consulte [Integração com editores via ACP](../user-guide/features/acp.md).

---

## Falhas comuns {#common-failure-modes}

Estes são os problemas que mais fazem perder tempo:

| Sintoma | Causa provável | Solução |
|---|---|---|
| O Hermes abre, mas dá respostas vazias ou quebradas | A autenticação do provedor ou a seleção de modelo está errada | Execute `hermes model` de novo e confirme o provedor, o modelo e a autenticação |
| O endpoint personalizado "funciona", mas devolve lixo | URL base ou nome do modelo errado, ou o endpoint não é de fato compatível com OpenAI | Teste o endpoint em outro cliente primeiro |
| O gateway inicia, mas ninguém consegue mandar mensagens para ele | Token do bot, lista de permissões ou configuração da plataforma incompleta | Execute `hermes gateway setup` de novo e verifique `hermes gateway status` |
| O `hermes --continue` não encontra a sessão antiga | Você trocou de perfil ou a sessão nunca foi salva | Verifique `hermes sessions list` e confirme que está no perfil certo |
| Modelo indisponível ou fallback com comportamento estranho | As configurações de roteamento ou de fallback estão agressivas demais | Deixe o roteamento desligado até o provedor base estar estável |
| O `hermes doctor` aponta problemas de configuração | Valores de configuração ausentes ou desatualizados | Corrija a configuração e teste de novo um chat simples antes de adicionar recursos |

## Kit de recuperação {#recovery-toolkit}

Quando algo parecer estranho, siga esta ordem:

1. `hermes doctor`
2. `hermes model`
3. `hermes setup`
4. `hermes sessions list`
5. `hermes --continue`
6. `hermes gateway status`

Essa sequência leva você rapidamente de "tem algo esquisito" de volta a um estado conhecido.

---

## Referência rápida {#quick-reference}

| Comando | Descrição |
|---------|-------------|
| `hermes` | Começa a conversar |
| `hermes model` | Escolhe o provedor de LLM e o modelo |
| `hermes tools` | Configura quais ferramentas ficam ativas em cada plataforma |
| `hermes setup` | Assistente de configuração completo (configura tudo de uma vez) |
| `hermes doctor` | Diagnostica problemas |
| `hermes update` | Atualiza para a versão mais recente |
| `hermes gateway` | Inicia o gateway de mensagens |
| `hermes --continue` | Retoma a última sessão |

## Próximos passos {#next-steps}

- **[Guia da CLI](../user-guide/cli.md)** — Domine a interface de terminal
- **[Configuração](../user-guide/configuration.md)** — Personalize a sua configuração
- **[Gateway de mensagens](../user-guide/messaging/index.md)** — Conecte Telegram, Discord, Slack, WhatsApp, Signal, e-mail, Home Assistant, Teams e muito mais
- **[Ferramentas e toolsets](../user-guide/features/tools.md)** — Conheça os recursos disponíveis
- **[Provedores de IA](../integrations/providers.md)** — Lista completa de provedores e detalhes de configuração
- **[Sistema de skills](../user-guide/features/skills.md)** — Fluxos de trabalho e conhecimento reutilizáveis
- **[Dicas e boas práticas](../guides/tips.md)** — Dicas para usuários avançados
- **[Migrar para outra máquina](../reference/faq.md#exporting-hermes-to-another-machine)** — O `hermes backup` migra toda a sua configuração (ou [um único perfil](../reference/faq.md#moving-a-single-profile-to-another-machine)); não é preciso refazer tudo do zero
