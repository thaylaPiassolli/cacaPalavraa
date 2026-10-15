# PRD — Jogo casual de adivinhação de palavras

## 1. Visão geral

Criar um jogo casual, rápido e acessível de adivinhação de palavras em português brasileiro. A pessoa escolhe uma dificuldade, tenta descobrir uma palavra secreta usando pistas por letra e pode jogar novamente sem limite, sem precisar criar uma conta.

## 2. Problema e oportunidade

Pessoas que gostam de desafios rápidos de palavras precisam de uma experiência simples de iniciar e entender, com partidas curtas e feedback claro. Jogos como Wordle, Termo e Contexto demonstram diferentes maneiras de estruturar desafios de palavras, mas a presença de funcionalidades nesses produtos não comprova, por si só, que elas aumentam retenção ou preferência.

## 3. Público-alvo

Pessoas que gostam de desafios rápidos de palavras e jogam casualmente, especialmente em dispositivos móveis. A primeira versão será voltada ao público de português brasileiro.

## 4. Objetivos do produto

- Permitir que a pessoa comece uma partida rapidamente, sem cadastro obrigatório.
- Oferecer uma mecânica de adivinhação compreensível, com feedback claro para cada palpite.
- Viabilizar partidas ilimitadas em três dificuldades.
- Mostrar um resumo do resultado e permitir compartilhá-lo sem revelar a palavra.
- Registrar estatísticas pessoais simples.

### Não objetivos da primeira versão

- Desafio diário compartilhado entre todas as pessoas.
- Monetização, anúncios ou compras.
- Dicas extras, modos sociais competitivos ou placares entre amigos.
- Conteúdo educativo com definições ou exemplos de uso.
- Suporte a português europeu.

## 5. Plataforma e modelo de acesso

- **Plataforma:** aplicação web responsiva, utilizável em celular e computador.
- **Acesso:** jogar sem conta.
- **Conta:** opcional, com o objetivo de sincronizar progresso entre dispositivos; detalhes de cadastro e sincronização ainda precisam ser definidos.
- **Preço:** gratuito e sem anúncios na primeira versão.

## 6. Ciclo principal de jogo

1. A pessoa escolhe uma dificuldade.
2. O jogo apresenta uma palavra secreta e permite inserir palpites com o número de letras esperado.
3. A pessoa envia um palpite válido.
4. O jogo informa, para cada letra, se ela está na posição correta, existe em outra posição ou não ocorre na palavra secreta.
5. A partida termina quando a palavra é descoberta ou quando todas as tentativas são usadas.
6. O jogo revela a palavra e apresenta o resumo da partida.
7. A pessoa pode iniciar outra partida ou compartilhar o resultado sem revelar a resposta.

## 7. Requisitos funcionais

### RF-01 — Dificuldade e tamanho das palavras

O jogo deve oferecer três dificuldades:

| Dificuldade | Tamanho da palavra | Tentativas |
|---|---:|---:|
| Fácil | 4 letras | 5 |
| Médio | 5 letras | 6 |
| Difícil | 6 letras | 7 |

### RF-02 — Palavras e palpites

- As palavras secretas devem ser palavras comuns do português brasileiro.
- Nomes próprios e siglas não devem ser usados como respostas.
- O jogo deve validar os palpites contra um vocabulário aceito.
- Se o palpite não for aceito, o jogo deve exibir um aviso e não consumir uma tentativa.
- Os palpites devem ter o mesmo número de letras da palavra secreta.
- Letras repetidas são permitidas tanto em palavras secretas quanto em palpites.
- A avaliação de letras repetidas deve respeitar corretamente a quantidade de ocorrências na palavra secreta.

### RF-03 — Acentuação

- O jogo deve aceitar entradas com ou sem acentos.
- Acentos devem ser ignorados na comparação entre palpite e palavra secreta.
- A forma de exibição da palavra revelada ainda precisa ser definida.

### RF-04 — Feedback

Após cada palpite válido, o jogo deve classificar cada letra como:

- correta e na posição correta;
- presente na palavra, mas em outra posição;
- ausente da palavra.

O feedback não deve depender exclusivamente de cores; cada estado precisa ter também uma distinção visual ou textual perceptível.

### RF-05 — Tutorial

- Na primeira partida, o jogo deve oferecer um tutorial curto e interativo.
- O tutorial deve explicar o objetivo, as tentativas, o significado das pistas e como enviar um palpite.
- A pessoa deve conseguir entender as regras sem depender somente de cor.

### RF-06 — Fim da partida

- Ao vencer ou esgotar as tentativas, o jogo deve revelar a palavra secreta.
- A tela final deve apresentar o resultado e o resumo da partida.
- Definições e exemplos de uso não fazem parte do escopo definido para a primeira versão.

### RF-07 — Partidas ilimitadas

- A pessoa deve poder iniciar uma nova partida após concluir a atual.
- Não haverá limite diário nem desafio diário na primeira versão.
- A política para evitar repetição de palavras entre partidas ainda precisa ser definida.

### RF-08 — Estatísticas

O jogo deve registrar estatísticas simples:

- número de partidas;
- número de vitórias;
- média de tentativas nas vitórias.

As estatísticas devem funcionar sem exigir uma conta. Persistência local, retenção e sincronização dos dados ficam pendentes de decisão técnica/produto.

### RF-09 — Compartilhamento

- A pessoa deve poder compartilhar um cartão de resultado sem revelar a palavra secreta.
- O compartilhamento não deve exigir conta.
- O conteúdo exato do cartão e os mecanismos de compartilhamento suportados ficam pendentes.

## 8. Requisitos não funcionais e qualidade

- A interface deve se adaptar a telas móveis e de computador.
- As ações essenciais devem ser operáveis por teclado.
- O feedback deve ser compreensível sem depender apenas de cor.
- O contraste de texto e os tamanhos dos alvos interativos devem ser verificados conforme critérios aplicáveis da WCAG 2.2.
- O jogo deve tratar de modo consistente acentos e letras repetidas.
- A lista de palavras e as regras de validação devem evitar respostas inválidas ou ambíguas, conforme critérios editoriais ainda a estabelecer.

## 9. Métricas de sucesso

As métricas específicas e as metas numéricas ainda não foram definidas. Candidatas para instrumentação e validação:

- taxa de conclusão do tutorial e início da primeira partida;
- taxa de vitórias e distribuição de tentativas por dificuldade;
- abandono de partidas antes do fim;
- partidas iniciadas por pessoa;
- retorno de pessoas ao jogo;
- uso do compartilhamento e visitas originadas por resultados compartilhados;
- taxa de palpites rejeitados.

Essas métricas são propostas para avaliação, não metas ou resultados esperados. As metas devem ser estabelecidas após definir instrumentação e obter uma linha de base.

## 10. Critérios de aceitação da primeira versão

- Uma pessoa consegue iniciar e completar uma partida sem criar conta.
- As três dificuldades usam os tamanhos e limites de tentativas definidos na seção 7.
- Entradas acentuadas e não acentuadas são tratadas de forma equivalente na comparação.
- Palpites inválidos são rejeitados com mensagem e sem consumir uma tentativa.
- Letras repetidas recebem feedback consistente com a quantidade de ocorrências.
- Cada palpite válido mostra os três estados de feedback definidos.
- A partida termina ao acertar a palavra ou esgotar tentativas e revela a resposta.
- O jogo permite iniciar outra partida sem limite diário.
- As estatísticas simples são registradas.
- O resultado pode ser compartilhado sem incluir a palavra secreta.
- Tutorial, feedback e ações principais são compreensíveis sem depender exclusivamente de cor.

## 11. Decisões em aberto

1. **Banco de palavras:** fonte, licença, curadoria, palavras aceitas como palpites e processo de revisão.
2. **Dificuldade:** critérios para classificar respostas como fáceis, médias ou difíceis, além do tamanho.
3. **Repetição:** política para evitar ou permitir palavras secretas repetidas em partidas sucessivas.
4. **Persistência:** armazenamento local das estatísticas, prazo de retenção e comportamento ao limpar dados do navegador.
5. **Conta opcional:** método de autenticação, dados sincronizados e fluxo de migração do progresso local.
6. **Compartilhamento:** formato do cartão, canais suportados e uso de emojis ou grade de pistas.
7. **Interface:** identidade visual, tema claro/escuro, teclado virtual e suporte a teclado físico.
8. **Privacidade e analytics:** dados coletados, consentimento, retenção e política de privacidade.
9. **Metas de sucesso:** valores-alvo, período de avaliação e segmentação por dificuldade.

## 12. Referências de pesquisa

As referências abaixo foram consultadas em 15 de outubro de 2026. Elas servem como exemplos de mecânicas e funcionalidades existentes, não como prova causal de impacto em retenção, aquisição ou receita.

- [Wordle — The New York Times](https://www.nytimes.com/games/wordle/index.html): desafio diário com seis tentativas para uma palavra de cinco letras; a página também apresenta arquivo de jogos como benefício de assinatura.
- [Connections — The New York Times](https://www.nytimes.com/games/connections): desafio diário de agrupar palavras por relações em comum.
- [Contexto](https://contexto.me/en/): adivinhação por proximidade semântica, com modos diário e ilimitado e acesso a jogos anteriores.
- [Termo](https://term.ooo/): jogo em português com pistas por posição, seis tentativas e explicações sobre acentos e letras repetidas.
- [Sobre o Termo](https://term.ooo/sobre/): contexto sobre o léxico e a origem do jogo.
- [WCAG 2.2 — Uso de cor](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html): orientação para não transmitir informação apenas por cor.
- [WCAG 2.2 — Teclado](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html): requisito de operabilidade por teclado.
- [WCAG 2.2 — Contraste mínimo](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html): critérios de contraste de texto.
- [WCAG 2.2 — Tamanho mínimo de alvos](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html): critérios para alvos interativos.

