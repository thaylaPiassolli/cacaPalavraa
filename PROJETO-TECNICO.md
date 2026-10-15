# Projeto técnico — jogo casual de adivinhação de palavras

## 1. Objetivo deste documento

Descrever uma implementação proposta para a primeira versão do jogo, integrando as três direções de interface do painel de protótipos:

- **Palavra em foco:** fluxo de jogo direto, com o tabuleiro como elemento central.
- **Clareza primeiro:** controles nativos, alto contraste, foco visível e pistas identificadas sem depender exclusivamente de cor.
- **Caderno de bordo:** papel, tinta, tipografia e detalhes editoriais discretos, sem competir com a leitura do tabuleiro.

Este documento detalha arquitetura, modelo de domínio, apresentação, armazenamento, acessibilidade, validação e etapas de implementação. As decisões nele identificadas como **proposta** complementam o [PRD](./PRD.md), mas não substituem as decisões de produto que ainda estão pendentes.

## 2. Escopo técnico do MVP

### Incluído

- Aplicação web responsiva, em português brasileiro.
- Partidas ilimitadas nas três dificuldades já definidas no PRD.
- Vocabulário local para validação de palpites e seleção de respostas.
- Entrada por teclado físico e campo de texto acessível.
- Teclado virtual clicável como opção de interface em telas de toque, sujeito à validação de produto.
- Feedback posicional por letra, com tratamento correto de letras repetidas.
- Tutorial inicial, resumo da partida, estatísticas locais e compartilhamento sem resposta.
- Persistência local sem conta ou serviços de backend.

### Fora do MVP

- Desafio diário sincronizado, placares ou partidas multiplayer.
- Contas, autenticação, sincronização entre dispositivos e backend.
- Anúncios, pagamentos, definições, dicas ou modos adicionais.
- Telemetria comportamental. Qualquer analytics requer decisão explícita de privacidade e definição de métricas.

## 3. Decisões e premissas de implementação

| Tema | Proposta técnica | Relação com o PRD |
|---|---|---|
| Stack | React, TypeScript e Vite; CSS próprio com propriedades customizadas | O repositório ainda não possui stack ou dependências definidos. |
| Renderização | Aplicação client-side, estática, sem API | O MVP é ilimitado e não precisa sincronizar um desafio comum. |
| Persistência | `localStorage` versionado para estatísticas e preferências mínimas | Persistência local ainda está pendente no PRD; proposta sujeita a aprovação. |
| Partida em andamento | Persistir a partida atual localmente para permitir recarregar a página sem perder o jogo | Proposta de UX; confirmar antes de assumir como requisito. |
| Teclado virtual | Teclas clicáveis na interface, além de entrada por teclado físico | Presente em duas direções visuais, mas não obrigatório no PRD. Deve ser validado por teste de usabilidade. |
| Repetição de respostas | Evitar repetir as últimas respostas na mesma dificuldade enquanto houver opções disponíveis | Política pendente no PRD; regra recomendada para partidas ilimitadas. |
| Fonte de palavras | Arquivos JSON versionados no projeto, sujeitos a curadoria e validação de licença | Fonte, licença e processo editorial estão pendentes no PRD. |
| Interface visual | Fundo de papel claro e neutro, superfícies limpas, grade central e detalhes editoriais leves | Combinação dos três mockups; manter legibilidade e reduzir decoração quando disputar atenção. |

Estas escolhas permitem entregar um MVP que funciona sem cadastro, rede ou backend. Se sincronização, desafio comum ou proteção contra manipulação virar requisito, a arquitetura precisará de API e armazenamento no servidor.

## 4. Arquitetura proposta

### 4.1 Visão geral

```text
┌──────────────────────────── Aplicação web ────────────────────────────┐
│                                                                       │
│  Interface React                                                      │
│  ├─ Página de jogo e controles acessíveis                             │
│  ├─ Tabuleiro, teclado, tutorial e resumo                              │
│  └─ Seleção de dificuldade e estatísticas                             │
│              │                                                        │
│              ▼                                                        │
│  Domínio puro (TypeScript)                                            │
│  ├─ máquina de estados da partida                                     │
│  ├─ normalização de palavras e validação                               │
│  ├─ avaliação posicional, inclusive letras repetidas                 │
│  └─ cálculo de estatísticas e cartão de compartilhamento             │
│              │                                                        │
│      ┌───────┴──────────┐                                             │
│      ▼                  ▼                                             │
│  Banco de palavras   Adaptadores do navegador                         │
│  JSON versionado     ├─ localStorage                                  │
│                      ├─ Web Share API                                 │
│                      └─ Clipboard API / fallback                      │
└───────────────────────────────────────────────────────────────────────┘
```

O domínio deve ser independente de React e do DOM. Assim, avaliação de palpites, normalização e estatísticas podem ser testadas como funções determinísticas sem montar a interface.

### 4.2 Estrutura inicial sugerida

```text
src/
  app/
    App.tsx
    GamePage.tsx
  components/
    DifficultySelect.tsx
    GameBoard.tsx
    GuessRow.tsx
    GuessInput.tsx
    OnScreenKeyboard.tsx
    HintLegend.tsx
    FirstRunTutorial.tsx
    GameSummary.tsx
    StatisticsPanel.tsx
    ShareResultButton.tsx
    StatusMessage.tsx
  domain/
    game.ts
    evaluateGuess.ts
    normalizeWord.ts
    statistics.ts
    shareCard.ts
    types.ts
  data/
    words-pt-BR.json
    word-data.schema.json
  hooks/
    useGame.ts
    usePersistentState.ts
  styles/
    tokens.css
    global.css
    game.css
  test/
    fixtures/
```

Testes próximos aos módulos podem usar sufixos `.test.ts` e `.test.tsx`. A estrutura é uma sugestão de organização, não uma exigência de criar cada arquivo antecipadamente.

## 5. Modelo de domínio

### 5.1 Configuração de dificuldade

```ts
type Difficulty = "easy" | "medium" | "hard";

type DifficultyConfig = {
  id: Difficulty;
  label: string;
  wordLength: 4 | 5 | 6;
  maxAttempts: 5 | 6 | 7;
};

const DIFFICULTIES: Record<Difficulty, DifficultyConfig> = {
  easy: { id: "easy", label: "Fácil", wordLength: 4, maxAttempts: 5 },
  medium: { id: "medium", label: "Médio", wordLength: 5, maxAttempts: 6 },
  hard: { id: "hard", label: "Difícil", wordLength: 6, maxAttempts: 7 },
};
```

A configuração deve ser a única fonte para exibir tamanho e tentativas, evitando divergência entre texto, tabuleiro e regras.

### 5.2 Tipos principais

```ts
type LetterResult = "correct" | "present" | "absent";

type EvaluatedLetter = {
  character: string;
  result: LetterResult;
};

type Guess = {
  display: string;
  normalized: string;
  evaluation: EvaluatedLetter[];
};

type GameStatus = "playing" | "won" | "lost";

type GameSession = {
  id: string;
  difficulty: Difficulty;
  answerId: string;
  answerDisplay: string;
  answerNormalized: string;
  guesses: Guess[];
  status: GameStatus;
  startedAt: number;
  completedAt?: number;
};
```

O segredo da partida pertence ao estado do jogo e não deve ser exposto em elementos da interface ou no cartão de compartilhamento antes da conclusão. Como o MVP roda no cliente e o vocabulário é distribuído ao navegador, isso não oferece proteção contra inspeção do código; proteção contra trapaça não faz parte do escopo.

## 6. Regras de palavras

### 6.1 Normalização

Criar uma função pura `normalizeWord(input)` com comportamento definido e coberto por testes:

1. Normalizar Unicode para decomposição canônica (`NFD`).
2. Remover marcas diacríticas combinantes.
3. Converter para minúsculas usando locale português (`pt-BR`).
4. Validar que o resultado contém somente letras do alfabeto aceito.
5. Contar letras por pontos de código Unicode, não por unidades UTF-16.

Exemplo conceitual:

```ts
function normalizeWord(input: string): string {
  return input
    .normalize("NFD")
    .replace(/\p{Diacritic}/gu, "")
    .toLocaleLowerCase("pt-BR")
    .normalize("NFC");
}
```

**Ponto de produto a confirmar:** essa normalização também trata `ç` como `c`. O PRD diz que acentos devem ser ignorados, mas não especifica a cedilha. Antes da implementação final do vocabulário, decidir se a equivalência deve incluir cedilha ou se ela deve ser preservada como letra distinta. A contagem de letras deve usar a representação definida após essa decisão.

O texto original normalizado em NFC deve continuar disponível para exibir a resposta com sua grafia editorial, se o banco incluir acentos.

### 6.2 Banco de palavras

Manter respostas e palpites aceitos separados, mesmo que inicialmente compartilhem uma lista:

```json
{
  "schemaVersion": 1,
  "locale": "pt-BR",
  "answers": {
    "4": [{ "id": "exemplo-001", "display": "..." }],
    "5": [{ "id": "exemplo-002", "display": "..." }],
    "6": [{ "id": "exemplo-003", "display": "..." }]
  },
  "acceptedGuesses": {
    "4": ["..."],
    "5": ["..."],
    "6": ["..."]
  }
}
```

O conteúdo real só deve ser incluído depois de verificar licença, origem e política editorial. Um validador de dados deve conferir no build ou em testes:

- tamanho correto por dificuldade, usando a regra oficial de normalização;
- unicidade de IDs e ausência de respostas duplicadas após normalização;
- ausência de nomes próprios, siglas e entradas proibidas pela curadoria;
- respostas também aceitas como palpites;
- pelo menos uma resposta válida por tamanho;
- formato e versão compatíveis com o schema.

Não baixar listas de dicionário ou copiar conteúdo de terceiros sem confirmar licença e direito de redistribuição.

## 7. Avaliação de palpites e letras repetidas

A avaliação deve ocorrer em duas passagens para não contar ocorrências duplicadas além da quantidade presente na resposta:

1. Inicializar todas as letras como `absent`.
2. Comparar letra a letra; marcar correspondências na mesma posição como `correct`.
3. Contabilizar apenas letras ainda não consumidas da resposta.
4. Percorrer as posições restantes do palpite: se houver uma ocorrência ainda disponível na resposta, marcar `present` e consumir uma ocorrência; caso contrário, manter `absent`.

Pseudocódigo:

```text
results = [absent] * tamanho
remainingCounts = contador das letras da resposta

para cada posição:
  se palpite[posição] == resposta[posição]:
    results[posição] = correct
    remainingCounts[resposta[posição]] -= 1

para cada posição ainda não correta:
  letra = palpite[posição]
  se remainingCounts[letra] > 0:
    results[posição] = present
    remainingCounts[letra] -= 1
```

Complexidade: O(n), sendo `n` entre 4 e 6 letras. O algoritmo precisa ser testado com respostas e palpites que contenham repetições em posições corretas e incorretas.

## 8. Máquina de estados da partida

### Estados

- **Playing:** aceita edição do palpite atual e submissão.
- **Won:** resposta descoberta; entradas desabilitadas; resumo exibido.
- **Lost:** tentativas esgotadas; resposta revelada; resumo exibido.

### Transições

```text
startGame(difficulty) ──> Playing
Playing + editar ───────> Playing
Playing + palpite curto ─> Playing + erro de validação
Playing + palavra inválida > Playing + erro (sem consumir tentativa)
Playing + palpite válido > Playing + linha avaliada
Playing + palpite correto > Won
Playing + última tentativa incorreta > Lost
Won/Lost + nova partida ─> Playing (nova sessão)
```

As ações do domínio devem rejeitar submissões quando a sessão não está em `Playing`, quando o comprimento não corresponde à dificuldade ou quando o vocabulário não reconhece o palpite. Erros esperados de input devem virar mensagens explícitas; falhas inesperadas de armazenamento não devem ser tratadas como sucesso silencioso.

Selecionar outra dificuldade durante uma partida ativa pode descartar progresso. O PRD não define esse comportamento; proposta: pedir confirmação antes de abandonar uma partida não concluída, ou desabilitar a troca até o fim. Decidir durante o desenho funcional.

## 9. Persistência local e estatísticas

### 9.1 Chaves versionadas

Exemplo:

- `word-game:v1:statistics`
- `word-game:v1:active-session`
- `word-game:v1:preferences`

Formato inicial das estatísticas:

```ts
type StatisticsV1 = {
  version: 1;
  gamesPlayed: number;
  wins: number;
  winningGuessCountTotal: number;
};
```

A média de tentativas nas vitórias é `winningGuessCountTotal / wins`; quando `wins` for zero, apresentar um estado vazio (por exemplo, “—”), nunca `NaN` ou infinito. Registrar uma partida apenas uma vez, quando passa de `playing` para `won` ou `lost`; a transição deve ser idempotente para evitar contagem duplicada após recarregar.

### 9.2 Falhas e migração

- Validar a estrutura lida do armazenamento antes de usar.
- Ignorar somente versões explicitamente migráveis.
- Para dados corrompidos, informar que estatísticas/progresso locais não puderam ser lidos e oferecer início de uma nova partida; não mascarar o problema como se dados tivessem sido restaurados.
- Se `localStorage` estiver bloqueado, indisponível ou cheio, manter a partida em memória e comunicar que o progresso não será mantido após fechar/recarregar.
- Nunca armazenar credenciais ou dados pessoais no MVP.

## 10. Interface e sistema visual

### 10.1 Composição

Desktop:

- cabeçalho contido com nome do jogo e acesso às regras;
- seletor de dificuldade com tamanho da palavra e tentativas visíveis;
- tabuleiro central e legenda imediatamente próxima;
- campo de palpite e ação principal abaixo do tabuleiro;
- painel de estatísticas secundário, sem disputar atenção com o jogo.

Mobile:

- coluna única, sem rolagem horizontal;
- dificuldade e tentativas próximas ao topo;
- tabuleiro centralizado e dimensionado para seis letras;
- entrada, botão de envio e teclado virtual (se aprovado) próximos ao tabuleiro;
- estatísticas abaixo da área principal.

### 10.2 Tokens de design propostos

| Token | Valor proposto | Uso |
|---|---|---|
| `--color-paper` | `#F5F3EC` | Fundo editorial muito suave |
| `--color-surface` | `#FFFFFF` | Cartões e células |
| `--color-ink` | `#17251F` | Texto principal |
| `--color-muted` | `#4F5D55` | Texto secundário com contraste legível |
| `--color-action` | `#176B50` | Ação e foco da dificuldade selecionada |
| `--color-correct` | `#176B50` | Letra na posição correta |
| `--color-present` | `#8B5700` | Letra presente em outra posição |
| `--color-absent` | `#46534B` | Letra ausente |
| `--color-border` | `#77847B` | Contornos de componentes e células |

Usar `system-ui` para texto e uma família monoespaçada do sistema para letras do tabuleiro. O visual de papel e detalhes de caderno deve ficar restrito a textura sutil, contornos e hierarquia tipográfica; remover padrões decorativos em tamanhos pequenos ou quando reduzirem legibilidade. Confirmar todos os pares de contraste com ferramenta e medição antes de declarar conformidade WCAG.

### 10.3 Estados das pistas

Cada estado combina pelo menos dois canais visuais, por exemplo:

- **Correta:** cor + símbolo/check ou contorno sólido diferenciado.
- **Presente:** cor + seta/símbolo ou borda tracejada.
- **Ausente:** cor + cruz ou padrão de preenchimento.

A legenda deve exibir os nomes em texto. O estado também precisa de nome acessível lido por tecnologia assistiva. Não usar cor como única forma de comunicação.

## 11. Acessibilidade e semântica

- Usar elementos nativos: `<button>`, `<input>`, `<select>`, `<form>`, listas e cabeçalhos em ordem lógica.
- Cada controle deve ter nome acessível; o input informa, por exemplo, “Seu palpite, 5 letras”.
- O seletor de dificuldade apresenta nome, comprimento e tentativas para cada opção.
- Submissão pode ser realizada pelo botão ou por Enter; o foco permanece no campo ao apresentar erro.
- Usar uma região `aria-live="polite"` para anunciar resultado de palpite válido e uma região de status apropriada para erros. Evitar anunciar todo o tabuleiro repetidamente.
- As células avaliadas expõem letra e estado (“A, presente em outra posição”) a leitores de tela; a linha recém-avaliada pode ser anunciada como resumo conciso.
- Manter ordem de foco previsível, foco visível e ausência de armadilhas de teclado.
- Se o teclado virtual for incluído, suas teclas são botões com rótulos acessíveis, foco visível e operação com Tab/Enter/Espaço. Ele complementa, não substitui, input físico nem campo semântico.
- Alvos de toque devem ser confortáveis; testar especialmente os controles menores em larguras de 320–390 CSS px.
- Respeitar `prefers-reduced-motion`; animações de revelação devem ser opcionais ou minimizadas.
- Tutorial inicial deve poder ser fechado e reaberto pela ajuda; se modal, prender e devolver o foco corretamente.
- Validar manualmente zoom, contraste, leitor de tela e navegação por teclado; automação sozinha não comprova conformidade.

## 12. Compartilhamento

Produzir uma mensagem derivada somente da dificuldade, estado final e matriz de resultados; nunca concatenar `answerDisplay` ou o palpite vencedor.

Fluxo:

1. Montar o texto do cartão em função pura.
2. Tentar `navigator.share({ text })` quando disponível.
3. Se não estiver disponível, copiar com `navigator.clipboard.writeText`.
4. Se o clipboard não existir ou for negado, mostrar o cartão para cópia manual e informar que a cópia automática falhou.
5. Anunciar “Resultado copiado” ou erro em região de status.

O cartão deve evitar spoilers e não incluir dados de conta. Mensagem, símbolos e branding precisam de aprovação de produto. Não criar endpoint de compartilhamento ou rastreamento no MVP.

## 13. Segurança, privacidade e confiabilidade

- Nenhum dado de usuário deve ser enviado a um servidor no MVP.
- O vocabulário faz parte do bundle e pode ser inspecionado; não tratar segredo da resposta como mecanismo de segurança.
- Evitar inserir conteúdo de palavras com `innerHTML`; renderizar como texto.
- Validar e limitar entrada antes da avaliação; normalizar Unicode e rejeitar símbolos fora do alfabeto permitido.
- Não permitir que a mesma sessão seja concluída duas vezes.
- Tratar falhas de persistência, clipboard e leitura do banco com mensagens explícitas e fallback funcional quando possível.
- Manter origem/licença e versão do vocabulário documentadas no repositório, após definição editorial.

## 14. Testes e validação

### 14.1 Testes unitários

- Normalização de maiúsculas, acentos, espaços e caracteres inválidos.
- Regra aprovada de cedilha e contagem Unicode.
- Avaliação de letra correta, presente e ausente.
- Repetição de letras: por exemplo, alvo `ABACA` e palpite `AAAAA` deve marcar as três ocorrências nas posições 1, 3 e 5 como corretas, e as duas outras como ausentes.
- Palpite inválido não consome tentativa.
- Vitória e derrota na quantidade correta de tentativas.
- Cálculo de estatísticas com zero vitórias e com múltiplas partidas.
- Cartão compartilhado sem resposta ou texto que revele a palavra.
- Validação estrutural e editorial do banco de palavras.

### 14.2 Testes de componentes e integração

- Troca de dificuldade atualiza comprimento da entrada, tabuleiro e tentativas.
- Envio válido adiciona linha com feedback e limpa o campo.
- Erro de validação é anunciado sem remover o foco do campo.
- Tutorial aparece na primeira execução e não reaparece após ser concluído (se a preferência for persistida).
- Fim de partida revela resposta e resumo, bloqueando palpites adicionais.
- Nova partida funciona nas três dificuldades.
- Dados restaurados e corrompidos produzem os estados previstos.
- Compartilhamento chama Web Share, clipboard ou fallback manual conforme a capacidade do navegador.

### 14.3 Testes de navegador e acessibilidade

- E2E em desktop e viewport móvel, cobrindo começo, submissão, fim e nova partida.
- Navegação integral por teclado e teste de foco visível.
- Leitor de tela em pelo menos uma combinação suportada de navegador/sistema.
- Verificação automática com axe ou ferramenta equivalente, complementada por inspeção manual.
- Contraste calculado para texto, bordas, foco e estados de pista.
- Sem overflow horizontal em 320, 375, 768 e 1280 CSS px.
- Preferência por movimento reduzido respeitada.
- Compatibilidade de Web Share e Clipboard com caminhos de fallback.

## 15. Desempenho e implantação

- O bundle inicial deve carregar interface e vocabulário necessário sem chamadas de API.
- Separar e comprimir arquivos de dados caso o vocabulário cresça significativamente; medir antes de introduzir carregamento dinâmico.
- Aplicar cache padrão de assets estáticos e versionar bundle/dados juntos para evitar incompatibilidade de schema.
- Hospedar em serviço estático HTTPS, a escolher durante a implementação.
- Não é necessário service worker/PWA instalável para cumprir o MVP; decidir após validar uso offline e suporte desejado.
- O comando de build deve falhar em caso de erros TypeScript ou vocabulário inválido.

## 16. Plano de implementação

### Fase 1 — Fundação

- Aprovar stack, tokens visuais e decisões em aberto de palavra/cedilha/persistência.
- Criar aplicação React + TypeScript + Vite e configurar lint, type-check e testes.
- Implementar e validar schema e ferramenta de verificação do banco de palavras.

### Fase 2 — Domínio do jogo

- Implementar normalização e validação de palpites.
- Implementar avaliador de letras com duplicatas.
- Implementar máquina de estados, seleção de palavra, vitória/derrota e estatísticas.
- Escrever testes unitários antes da integração com interface.

### Fase 3 — Interface integrada

- Implementar sistema visual combinado e layout responsivo.
- Integrar input nativo, tabuleiro, legenda e teclado virtual se aprovado.
- Implementar tutorial, estados de erro, resumo e reinício.
- Implementar estatísticas e sessão local versionada.

### Fase 4 — Acessibilidade e compartilhamento

- Adicionar semântica, anúncios de status, foco e nomes dos estados das células.
- Implementar compartilhamento com fallback para cópia manual.
- Revisar motion, contraste, zoom, teclado e leitores de tela.

### Fase 5 — Qualidade e lançamento

- Rodar testes unitários, integração, E2E, análise de acessibilidade e build.
- Conferir licença e qualidade editorial do vocabulário.
- Testar com pessoas do público-alvo em celular e desktop; registrar dificuldades antes de fechar o design.
- Implantar em hospedagem HTTPS e verificar a versão publicada.

## 17. Critérios técnicos de pronto

- `tsc`/type-check e build de produção passam sem erros.
- Testes cobrem as regras essenciais, letras repetidas, acentuação e transições de estado.
- Não há dependência de backend para jogar ou ler estatísticas no MVP.
- Palpites inválidos não consomem tentativa e apresentam erro acessível.
- A avaliação de letras não superconta letras repetidas.
- A resposta não aparece no cartão de compartilhamento.
- Fluxo principal funciona por teclado e toque.
- Não há overflow horizontal nos viewports definidos.
- As mensagens de erro não são ocultadas por fallback silencioso.
- Vocabulário usado tem origem e licença verificadas.

## 18. Decisões pendentes para aprovação

1. Confirmar uso de React/TypeScript/Vite ou escolher uma stack diferente.
2. Definir fonte, licença, tamanho e política de curadoria do banco de palavras.
3. Especificar se `ç` e `c` devem ser equivalentes no mecanismo de normalização.
4. Aprovar persistência local da partida em andamento e política de retenção.
5. Definir se o teclado virtual será incluído na primeira versão.
6. Definir comportamento ao trocar dificuldade durante partida ativa.
7. Aprovar regra para evitar respostas repetidas.
8. Definir formato textual/visual do cartão sem spoiler.
9. Definir hospedagem e se o jogo deve funcionar instalável/offline como PWA.

## 19. Relação com os artefatos do projeto

- Requisitos e decisões de produto: [PRD.md](./PRD.md).
- Painel dos três conceitos e protótipo visual: [prototipos.html](./prototipos.html).

