# Interação por cards — protótipo

| | |
|---|---|
| **O que este documento responde** | O que é o sistema de interação por cards, como usar, como estender e o que ainda falta para ele substituir o roteiro |
| **Estado** | **Protótipo** (07/10/2026). Convive com o `RoteiroData` sem mexer nele; nada do jogo depende dele, só a demo |
| **Referência** | O sistema "Diretor" de cards de um RPG independente (vídeo em `Downloads/claude/todo/sistema de interacao`). Escopo combinado: **a estrutura e o funcionamento** do grafo de interação; **sem** câmera, cinemática, animação e movimentação de objetos |
| **Escopo (10/10)** | Chama todos os sistemas do jogo (batalha, loja, tutor, troca, cura, evento de roteiro) sem substituí-los; cobre qualquer interação e evento roteirizado. Editar pelo Inspector é aceito. Ver §7 |

---

## 1. A ideia

Uma interação (conversa com NPC, evento, cutscene simples) é um **grafo de cards**. Cada card faz uma coisa e diz por
qual **saída** seguir; cada saída aponta para outro card. A diferença para o roteiro de hoje é que o roteiro é uma
**árvore** (listas aninhadas em `PassoSe`/`PassoEscolha`): no grafo, dois caminhos podem **voltar ao mesmo card** — o
menu do vendedor que reaparece depois de cada opção — sem repetir passos.

O grafo **não tem executor próprio**: roda dentro do `ExecutorDeRoteiro` como conversa avulsa
(`ExecutorDeRoteiro.IniciarConversa`). Por isso herda a trava de controle, a fala, a escolha e a batalha do roteiro.

## 2. As peças

| Peça | Arquivo | Papel |
|---|---|---|
| `InteracaoData` | `Assets/Scripts/Interacao/InteracaoData.cs` | O asset: lista de cards e o card inicial; `Problemas()` valida o grafo |
| `CardDeInteracao` | idem | O nó: id, posição no editor, entrada nomeada, conteúdo, saídas, configuração |
| `ConfiguracaoDeCard` | idem | Progresso, som e sinais (antes ou depois do card) |
| `ConteudoDeCard` e os tipos | `CardsDeInteracao.cs` | O que o card faz (tabela abaixo) |
| `ExecucaoDeInteracao` | `ExecucaoDeInteracao.cs` | Anda pelo grafo; corta laço sem espera (2.000 cards) e aninhamento (16) |
| `ProgressoDeInteracao` | idem | Onde cada interação parou, gravado como flag `interacao_progresso:<asset>=<id>` (o save não muda de formato) |
| `CanalDeSinais` / `ReceptorDeSinal` | idem / `ReceptorDeSinal.cs` | O card emite um **nome**; um componente na cena escuta e reage (UnityEvent ou liga/desliga um objeto) |
| `NpcComInteracao` | `NpcComInteracao.cs` | Interagível que roda uma interação: começa no progresso salvo, senão na entrada, senão no card inicial |
| **Inspector** (editor principal, T44) | `Assets/Scripts/Editor/Interacao/InspetorDeInteracao.cs` | Fluxo em texto (`InteracaoData.Esboco`), problemas, card inicial, lista de cards com tipo, campos, saídas como lista suspensa ("fim", os cards existentes, "novo card…") e configurações; subir, descer, duplicar, apagar |
| Cards de sistema (T43) | `Assets/Scripts/Interacao/CardsDeSistema.cs` | Batalha, selvagem, cura, troca e tutor: chamam o código dos sistemas donos e devolvem o resultado como saída |
| Editor de grafo | `Assets/Scripts/Editor/Interacao/` | Janela `Pokemon → Interacao (prototipo) → Editor de interacao` (GraphView) |

### Tipos de card

| Card | Equivale no vídeo | Saídas | Faz |
|---|---|---|---|
| **Fala** | Dialog | 1 | Fala com prefixo de quem fala; aceita `{jogador}`, `{rival}` e as marcas de gênero . **Emoção** (vai para a tela e vira o sinal `emocao:<quem>:<Emocao>`), **avançar sozinha** em N segundos e **não pode pular** (o botão é ignorado; exige o tempo) |
| **Escolha** | Choice | 1 por opção | Pergunta e segue pela opção escolhida |
| **Se** | Test | 2 (sim/não) | Qualquer `CondicaoDeRoteiro` (flag, item, insígnia, Pokémon…) |
| **Passo** | — | 1 | **Qualquer um dos 22 passos de roteiro** (dar item, batalha, curar, teleportar, dinheiro…) |
| **Loja** | Market | 1 | Abre a loja (um `LojaData` **ou** a lista de itens no próprio card, montada na hora) e só segue quando o jogador fecha |
| **Esperar** | Wait | 1 | Segundos |
| **Emitir sinal** | Generic Events | 1 | Emite um sinal para a cena |
| **Esperar sinal** | Wait (por sinal) | 1, ou 2 com limite de tempo | Para até a cena (ou um ramo paralelo) emitir o sinal |
| **Entrar** | Enter Director | 0, ou 1 com "voltar depois" | Pula para um card (id ou entrada nomeada) desta ou de outra interação; com "voltar depois", roda a outra até o fim e continua |
| **Paralelo** | Enter Parallel | 1 | Começa outra interação **sem esperar** por ela. Feito para sons, sinais e esperas: dois ramos falando ao mesmo tempo disputariam a caixa de diálogo |
| **Nada** | None / Empty | 1 | Junção de fios ou rascunho |
| **Fim** | End | 0 | Termina |
| **Batalha** | — | "venceu" (+ "perdeu" com `semBlackout`) | Batalha de treinador com os mesmos dados do `PassoBatalha`; sem `semBlackout`, perder termina a interação e o jogador fica onde está, com a equipe desmaiada (sem apagão, sem perder dinheiro, sem teleporte: DECISOES T48) |
| **Batalha selvagem** | — | "venceu", "capturou", "fugiu" (+ "perdeu") | Pokémon fixo (lendário, Snorlax), como o `PassoBatalhaSelvagem` . **As duas batalhas aceitam clima e terreno desde o início** (12/10): `Clima inicial` (a luta começa com esse clima no lugar do clima do mundo e o mantém até o fim; Qualquer = o do mundo), `Terreno` fixo e `Terrenos por contexto` (muda com o clima e o período do mundo; o mais específico que vale ganha) |
| **Curar equipe** | — | "curou", "nao precisava" | A cura do `Curandeiro`, e marca a última cura (hoje nada leva o jogador para lá) |
| **Troca** | — | "trocou", "nao tem" | A troca do `NpcTrocador` (equipe, depois caixa; flag; evolução por troca). As falas do NPC ficam nos cards em volta |
| **Tutor** | — | 1 | A conversa inteira do `NpcEnsinaGolpe`, a partir do prefab do professor (`Assets/Data/Mundo/Professores`) |

**Por que sinal em vez de UnityEvent no card:** um asset não consegue apontar para objeto de cena, e um UnityEvent
chama qualquer coisa sem passar pelo validador nem pelo determinismo que o multiplayer em rede exige. O sinal é um nome:
o card não conhece a cena, a cena escuta. O UnityEvent continua existindo, **do lado da cena** (`ReceptorDeSinal`), onde
é seguro.

## 3. Como usar

1. **Criar:** `Assets → Create → Pokemon → Interacao (prototipo de cards)`.
2. **Editar:** selecione o asset; o **Inspector** é o editor principal (fluxo em texto no topo, cada saída escolhida numa lista suspensa). O grafo é opcional: botão **Abrir no grafo (opcional)**.
   - Botão direito no fundo: **Novo card**. O primeiro card criado vira o inicial.
   - Arraste de uma **saída** (direita) até uma **entrada** (esquerda) para ligar.
   - Abas do card: **Card** (o tipo e seus campos, e a entrada nomeada) e **Configurações** (progresso, som, sinais).
   - Botão direito no card: definir como inicial, duplicar, copiar id.
   - **Validar** (barra de cima): fio quebrado, id repetido, card sem tipo, card que nenhum caminho alcança.
3. **Pôr no mundo:** um objeto com gatilho e o componente `NpcComInteracao`, como o filho
   `Interacao` dos NPCs da multicena.
4. **Reagir na cena:** `ReceptorDeSinal` com o mesmo nome do sinal.

**Novo tipo de card:** uma classe `[Serializable]` filha de `ConteudoDeCard`, com `Executar` (e `QuantasSaidas` /
`NomeDaSaida` se tiver mais de uma saída). O editor e o seletor de tipo acham sozinhos.

## 4. A demo (KantoMulticena)

Menu `Pokemon → Interacao (prototipo) → Montar demo na KantoMulticena` (reaplicável). Põe em Pallet, ao lado do vendedor de
teste, o **Mercador** (`PrototipoInteracao/Mercador_PrototipoDeCards` em `Kanto_Pallet.unity`) com
`Assets/Data/Mundo/Interacoes/Prototipo/InteracaoDemo_Mercador.asset`:

- apresentação só na primeira vez (progresso: depois começa em "Então, o que manda?");
- menu que volta para si: **Comprar** (loja de teste), **Conversar** (sinal que acende uma lâmpada ao lado), **Presente**
  (condição "já ganhou?" e escolha 2000 de dinheiro × informação, uma vez só), **Sair** (dispara um ramo paralelo que
  pisca a lâmpada duas vezes com o jogador já livre).

## 5. Testes

- **EditMode** `InteracaoPrototipoTests` (12): sequência, menu em laço, condição, passo de roteiro, progresso, entrar e
- **EditMode** `InteracaoCardsDeSistemaTests` (12): batalha (vencer, perder com e sem blackout), selvagem (capturar, fugir), cura (curou, não precisava), troca (tem, não tem), loja avulsa, esboço do fluxo e as saídas nomeadas. Suíte EditMode inteira 633/633 em 10/10.
  voltar, corte de laço, sinais, validação, a demo inteira, e o editor de grafo (nós, fios, criar, duplicar).
- **PlayMode** `InteracaoPrototipoPlayModeTests` (1, categoria `Mapa`): o Mercador jogado na KantoMulticena com fala,
  escolha, loja, presente, progresso e ramo paralelo reais. 6/6 rodadas em 07/10.

## 6. O que falta para sair de protótipo

| Falta | Por quê |
|---|---|
| **UI de escolha** | **Feita em 11/10:** `EscolhaDeRoteiroUI`, uGUI montada por código na primeira escolha (sem prefab nem cena para autorar, então não espera a decisão da pilha de UI). Teclado e controle andam pelos botões; mouse e toque clicam. Fica o pedido de **nome**, que ainda é IMGUI |
| Câmera, animação, movimentação de objetos por card | Fora do escopo combinado |
| Validador do projeto lendo os grafos | Hoje o grafo se valida sozinho (`Problemas()`), mas o `ValidadorDeRoteiro` não cruza as flags de dentro do card com as do resto do jogo de forma especial (ele lê o YAML, então pega `flagLida`/`flagEscrita` do mesmo jeito) |
| ~~Decisão: migrar os roteiros~~ | **Decidido em 10/10: não migrar.** Os dois formatos convivem: roteiro (árvore) para o que já existe, cards para o novo |
| Editor de grafo: GraphView é API experimental da Unity | Tudo que depende dela está em `GrafoDeInteracao.cs`; trocar de API não toca o jogo |
| Conferência visual do Inspector e da janela do grafo | Os testes cobrem o comportamento; o Inspector novo (10/10) ainda não foi aberto à mão |


---

## 7. O que os vídeos de referência mostram (revisado em 10/10/2026) e o escopo novo

Fonte: as transcrições completas em `Downloads/claude/todo/sistema de interacao` (`video3.txt`, a demonstração do RPG; `video4.txt`, o "sistema diretor" de cards; `transcricao.txt`, os trechos que originaram o protótipo; 45 capturas de tela). O autor do vídeo constrói o editor de cards **sem biblioteca**, só com a API básica de extensão do editor da Unity, e diz que o sistema é "universal": uma sequência de cartões executados um após o outro, no estilo dos RPGs visual-novel.

**Escopo decidido pelo dono do projeto em 10/10:** o sistema de interação **não substitui os sistemas de domínio**, mas **chama todos eles**: batalha, loja com os dados associados, tutor, troca, cura, evento de roteiro. É o coerente: o card orquestra, o sistema dono executa. O objetivo é cobrir qualquer interação e qualquer evento roteirizado do jogo. **Editar pelo Inspector é aceito**; o GraphView não é obrigatório (ver §7.2).

### 7.1 O que o vídeo tem, e onde estamos

| No vídeo | Estado aqui |
|---|---|
| Card inicial ("card zero"); saltar para qualquer card; laços (menu que volta) | ✅ card inicial, entrada nomeada, laço |
| Card de **fala** com quem fala, cor do título e **emoção** | ✅ quem fala e emoção (10/10); cor do título é da tela de diálogo |
| Card de **escolha** com N saídas | ✅ |
| Card de **pular sozinho** depois de N segundos (*time skip*) | ✅ `Esperar`, e a própria fala avança sozinha e pode não deixar pular (10/10) |
| Card de **mercado**: lista de itens com id e preço, limite de compra por vez | ✅ `Loja`; **conferir** o limite por compra |
| NPC de **escolha irreversível** (aceita presente × recusa, uma vez só) | ✅ a demo do Mercador faz isso |
| **Cutscene** de NPCs conversando sem input do jogador, com voz | 🟡 pelo `Paralelo` e `Esperar sinal`; faltam som de fala e câmera |
| **Aba de câmera**: interpolação entre os pontos A e B, com pré-visualização na cena e botão "pegar da câmera da cena" | ❌ |
| **Aba de eventos**: mover objeto (com "fantasma" na cena), instanciar, animação, som, eventos pré-programados | 🟡 mover NPC/jogador, posicionar, virar e esconder já são `Passo`s do roteiro; instanciar e animação ❌; som já está na configuração do card |
| **Eventos genéricos**: chamar qualquer função pública | 🟡 deliberadamente por **sinal** (nome), não por chamada livre: ver §2 |
| Mexer no estado do jogador, ordem de execução | ✅ `Passo` (flag, item, dinheiro…) e sinais antes/depois |
| Editor visual de nós | 🟡 GraphView, dispensável (§7.2) |

### 7.2 O GraphView não é necessário

Nada do jogo depende dele: está inteiro em `Editor/Interacao/GrafoDeInteracao.cs` (349 linhas) e `JanelaDeInteracao.cs` (104), e o runtime só lê o asset (`InteracaoData`). Editar sem ele funciona com um **Inspector próprio**: lista de cards com a seta de tipo, os campos do conteúdo (já são `[SerializeReference]`), as saídas como **lista suspensa dos ids** (com um resumo do card de destino) e o botão Validar. O custo real é a leitura de ramificações grandes; mitiga-se com um **resumo em texto** do fluxo (o caminho a partir do card inicial, em árvore indentada) e com o `Problemas()` que já acusa fio quebrado e card inalcançável. O GraphView pode ficar como visualizador opcional, sem investimento novo. A API é experimental na Unity (já registrado em §6).

### 7.3 O que muda para câmera, movimento e preview na cena

O vídeo guarda os cards **na cena**, por isso arrasta objetos da cena para o card. Aqui a interação é um **asset**, e um asset **não pode apontar para objeto de cena**. A saída é a mesma que o roteiro já usa para os atores: o card guarda um **nome de âncora** (`AtorDeRoteiro` já resolve por id) e a cena tem um componente de âncora com o *gizmo* para posicionar. A pré-visualização do vídeo (fantasma do objeto, câmera A/B) viria do componente de âncora, não do asset.

### 7.4 Ordem sugerida para ampliar o protótipo

1. ~~Inspector próprio como editor principal (§7.2).~~ **Feito em 10/10.**
2. **Feito em 10/10 em parte** (batalha, selvagem, cura, troca, tutor, loja avulsa). Faltam armazenamento (PC), evolução por ponto e dar Pokémon como card nomeado. Cards que **chamam sistemas de domínio** que ainda faltam: batalha de treinador com as opções de A12, tutor, troca, cura, armazenamento, evolução, mover. Hoje `Passo` cobre os 22 passos do roteiro; um card nomeado para cada sistema evita que o autor procure dentro de `Passo`.
3. UI de escolha de verdade (depende de P1).
4. Âncoras de cena e card de câmera, depois animação e instanciar (feitos: §8.1).
5. ~~Converter os roteiros.~~ **Decidido em 10/10: não migrar.**

---

## 8. Eventos que a interação chama (10/10/2026)

Os cards chamam **ações** pelo card **Passo** (qualquer passo de roteiro) e **perguntas** pelo card **Se** (qualquer condição de roteiro). Cada passo novo vale para os cards e para os roteiros ao mesmo tempo. "Liberar ou marcar algo que outra coisa verifica" é o par flag/variável escrita aqui e lida por uma condição em outro lugar.

| Para | Passo (ação) | Condição (pergunta) |
|---|---|---|
| Dar e tirar item | `PassoDarItem`, `PassoTirarItem` | `SeTemItem` |
| Dar e tirar Pokémon | `PassoDarPokemon`, **`PassoTirarPokemon`** (nunca o último da equipe) | `SeTemPokemonNaEquipe`, **`SeTemPokemon`** (nível mínimo, caixa) |
| Mudar um Pokémon | **`PassoAlterarPokemon`**: subir nível, ensinar golpe, equipar/tirar item, amizade, apelido, forma, curar, tipo Tera | |
| Forçar evolução | **`PassoEvoluirPokemon`** (sem checar condição; destino vazio = a primeira evolução) | |
| Marcar e liberar | `PassoLigarFlag`, `PassoDesligarFlag` | `SeTemFlag`, `SeNaoTemFlag` |
| Informação do personagem (números) | **`PassoDefinirVariavel`** (definir ou somar; guardada como flag `var:nome=valor`, o save não muda) | **`SeVariavel`** (=, ≠, ≥, ≤, >, <) |
| Dinheiro e insígnias | `PassoDarDinheiro` (negativo tira), `PassoDarInsignia` | **`SeTemDinheiro`**, `SeInsigniasPeloMenos` |
| Nome | `PassoDefinirNome` | |
| Combinar perguntas | | `SeTodasDestas`, `SeQualquerUmaDestas` |

Em negrito, o que entrou em 10/10 (`Roteiro/PassosDePersonagem.cs`, testes em `InteracaoEventosTests`). O que mexe em **cena** (mover, virar, mostrar/esconder ator, teleportar) já existia como passo; câmera, animação e instanciar continuam em §7.3.

### 8.1 Acrescentado na fase 2 (10/10/2026)

| Para | Peça | Detalhe |
|---|---|---|
| PC | card **Armazenamento** | Abre a caixa e segue quando o jogador fecha |
| Evolução por lugar/evento | card **Evolução por evento** | Saídas "evoluiu" e "nada"; a mesma regra do `PontoDeEvolucao` |
| Dar Pokémon completo | card **Dar Pokémon** | Usa a **ficha completa** do membro de time (golpes, item, IV, EV, natureza, habilidade, gênero, tipo Tera, apelido); saídas "na equipe" e "na caixa" |
| Câmera de cena | card **Câmera** + **Devolver câmera** | Vai até uma `AncoraDeCena` em N segundos (0 = corte). Não mexe na câmera do jogo: cria uma temporária e desliga a principal; a interação sempre devolve ao terminar |
| Mover objeto | card **Mover objeto** | Leva o objeto de uma âncora até a pose de outra |
| Âncora | componente `AncoraDeCena` | Id + gizmo com o enquadramento; é o "preview na cena" do vídeo |
| Missões | `MissaoData` + `PassoMissao` + `SeMissao` | Quatro estados (não iniciada, ativa, concluída, entregue) e etapas; o diário lê `MissoesDoJogador.EmAberto` e o catálogo lista todas |
| Estatísticas | `EstatisticasDoJogador` | Batalhas vencidas e perdidas, treinadores vencidos, capturas, fugas e passos, produzidas pela batalha e pelo contador de passos; `SeVariavel` com "estat:<nome>" pergunta |
| Repelente | item `ItemRepelente` | Fora dos cards: usado da mochila. Ver o desenho de encontros em [`MODULOS_DO_JOGO.md`](MODULOS_DO_JOGO.md) |
| Animar um objeto | card **Animar** | Mexe no `Animator` do objeto da âncora (ou de um filho ou pai dele): gatilho, booleano, inteiro, decimal, ou toca um estado pelo nome (com transição). Espera um tempo fixo, ou o estado terminar (até 30 s); sem espera, a interação segue enquanto a animação toca. Sem âncora ou sem `Animator`: aviso e segue |
| Criar um objeto | card **Instanciar** | Cria um prefab na pose de uma âncora, na cena da âncora (sai junto quando a área descarrega). Com "id do objeto", o criado ganha uma âncora própria, e os cards de mover, animar e remover o acham. Rodar de novo troca o objeto (ou deixa, com "substituir" desligado). O prefab é **referência direta**, de propósito: a interação é um asset de cena e não entra no catálogo, então não há registro de prefabs. Não vai para o save: se a cena o quer depois de carregar, a interação roda de novo, decidida por uma flag |
| Apagar o criado | card **Remover objeto** | Apaga só o que um card Instanciar criou. Um objeto que a cena trouxe nunca é apagado por card (aviso e nada acontece) |
| Trocar a habilidade | card **Trocar habilidade** | O NPC oferece as habilidades **dele** a qualquer espécie (decisão de 12/10). Pela caixa de escolha: o Pokémon da equipe (com a habilidade que tem), a habilidade (as do NPC que ele ainda não tem, mais Voltar) e a confirmação com a descrição. A troca fica no indivíduo, evolução não a muda e o save a grava (`Pokemon.ConcederHabilidade`). Cada NPC leva a própria lista; saídas "trocou" e "não trocou". Ainda não há preço |
| Ligar/desligar o que já está na cena | card **Ativar objeto** | Ligar, Desligar ou Alternar. Objeto que **começa desligado** não se registra: ponha a âncora num pai ligado e escreva o filho no card. A âncora de um objeto desligado depois continua achável |
