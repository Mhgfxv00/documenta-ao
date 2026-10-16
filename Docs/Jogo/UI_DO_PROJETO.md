# UI do projeto — tudo o que o jogo precisa mostrar

> **A UI é a maior trava do projeto (aviso de 07/10/2026).** A ordem de ataque é a §6 deste documento; o aviso completo e os riscos estão em [`../Projeto/ESTADO_DO_PROJETO.md`](../Projeto/ESTADO_DO_PROJETO.md) e a pergunta que destrava, em [`../Projeto/DECISOES.md`](../Projeto/DECISOES.md) (P1).

| | |
|---|---|
| **O que este documento responde** | Que telas o jogo precisa, o que cada uma mostra e chama, o que já existe e o que falta |
| **Quando abrir** | Ao começar ou migrar uma tela, ao decidir o que fazer primeiro, ou quando um sistema novo pergunta "e onde o jogador vê isso?" |
| **O que ele NÃO é** | Guia de montagem no Editor (isso é o [`MENUS_UGUI.md`](MENUS_UGUI.md)) nem referência dos sistemas ([`REFERENCIA_DE_SISTEMAS.md`](REFERENCIA_DE_SISTEMAS.md)) |
| **Para quem** | Quem cuida da UI (frente "outro lado do grupo", ver [`COMO_TRABALHAR.md`](../Projeto/COMO_TRABALHAR.md) §1) e quem escreve sistema e precisa saber o que a tela consome |
| **Atualizado em** | 30/09/2026 — contado no código e nos assets desta data, não de memória; notas de 07/10 sobre os placeholders e a entrada única; links e referências de documento corrigidos em 09/10 (os números de conteúdo da §2.1 são de 30/09) |

**Este documento parte da NECESSIDADE, não do que já existe.** A pergunta de cada linha é "o jogo precisa mostrar isto?" (pelo design declarado no
[GDD](GDD.md), pelo que o backlog promete e pelo que os sistemas já calculam), e só depois "e existe?". Tela que ainda não tem código nem sistema
entra do mesmo jeito, marcada como **não existe**.

**Como ler.** A §1 são as regras que valem para toda tela. A **§2 é o inventário de necessidades** por pilar do jogo, cada uma com a tela que a
atende e se ela existe hoje. A §2.1 é o retrato do que já está construído. A §3 é o catálogo: **uma seção por tela**, com o que ela mostra, de onde vem
o dado, o que pode pedir ao sistema e o estado atual. A §4 lista o que cruza todas as telas. A §5 são os requisitos que não são de tela nenhuma
(desempenho, acessibilidade). A §6 é a ordem sugerida. A §7 é o que precisa de decisão.

---

> **Comparar as duas pilhas (14/10).** No Editor ou numa build de desenvolvimento, dê Play a partir do `Bootstrap` e clique no botão **UI** no canto de cima, à direita (`ComparadorDeUI`). Cada linha é uma tela; **uGUI** e **Toolkit** abrem a versão respectiva. É ferramenta de decisão da P1, não tela do jogo, e some das builds finais.

## 1. Regras que valem para toda tela

1. **A UI pede, o sistema altera.** Nenhuma tela muda dado de jogo por conta própria: ela chama a porta do sistema
   (`SistemaBatalha`, `SistemaInventario`, `SistemaLoja`, `SistemaSave`, `SistemaDialogo`, `ExecutorDeRoteiro`) e **redesenha quando o sistema avisa**
   (evento). Se a tela precisa de uma regra que não existe, a regra vai para o sistema, não para o `.cs` da tela.
2. **Uma fonte de verdade.** Equipe, caixa, mochila, dinheiro e Pokédex moram em `DadosJogador`. Não existe `PartyManager` de tela: criar um é criar a
   segunda cópia que diverge na primeira captura.
3. **Input só pela `EscutaDeInput`**, na Action Map própria (`Assets/Scripts/Inputs.inputactions`). Nunca guardar `entrada.Mapa` (quebra em
   silêncio depois de uma recriação) e nunca `Input.GetKey*`. A tecla **E** é `Interagir` e o atalho da Pokédex só abre quando não há nada por
   perto para interagir (`AtalhoPokedex`).
4. **Estado do jogo pelo `GerenciadorEstados`** (explorando, diálogo, batalha, menu). Toda tela que abre pergunta a ele e devolve o estado ao fechar.
   Foi a falta dessa pergunta que fez a Pokédex abrir no meio da batalha.
5. **Texto em PT-BR literal.** Não há localização (decisão vigente); texto novo continua literal, mas sempre num lugar só da tela, para uma
   localização futura não precisar caçar.
6. **Toda tela navegável por teclado e por controle**, sem mouse: `Navigate`, `Submit`, `Cancel` da mapa `UI`. Mouse é bônus.
7. **Sem `Find` por quadro, sem LINQ, sem alocar por quadro** (regra de runtime do projeto). Lista longa é **montada sob demanda e reaproveitada**.
8. **Máquina mínima de referência:** Ryzen 5 4600G (gráfico integrado), 16 GB, HD + SSD. Toda meta é contra ela.
9. **Falha que o jogador precisa ver fala com o jogador** (`Narrar`, `UltimoErro`, mensagem na tela), não `Debug.Log`.
10. **Não commitar o que a Unity regravou sem intenção** (UXML/USS e cenas): confira o `git diff`.

---

## 2. O que o jogo precisa mostrar (existindo ou não)

**Origem** diz de onde vem a necessidade: **GDD** ([`GDD.md`](GDD.md), o jogo como ele é), **Backlog** (o que o projeto já decidiu que falta), **Sistema** (o jogo já
calcula e o jogador não vê — a varredura de 05/09, em [`BACKLOG_DO_README_ATE_2026-09.md`](../Historico/BACKLOG_DO_README_ATE_2026-09.md)) ou **Alvo** (o alvo declarado: mundo em várias áreas, 2 a 18 jogadores em rede).
**Hoje**: ✅ existe · 🟡 existe pela metade ou como placeholder · ❌ não existe.

### A. Entrar, sair e configurar

| Necessidade | Origem | Tela (§3) | Hoje |
|---|---|---|---|
| Título com Continuar, Novo jogo, Opções, Créditos, Sair | GDD §8 | 3.1.1 | 🟡 provisória |
| Criar o personagem (gênero, aparência, **nome**) | GDD, Alvo | 3.1.2 | 🟡 sem nome |
| Escolher slot, salvar, carregar, apagar, com confirmação | GDD §8 | 3.1.3 | 🟡 sem confirmação de sobrescrever |
| **Carregamento com progresso** (a carga leva 8 a 62 s no HD; mundo em cenas por área) | Alvo, medido | 3.1.6 | ❌ |
| Confirmação e erro de save chegam ao jogador | Backlog | 3.1.3 | 🟡 só erro |
| Opções: vídeo, áudio, controles com **rebind**, dificuldade, escala de texto | GDD §6, Backlog | 3.1.4 | 🟡 duplicada, sem rebind |
| Pausa e Créditos | GDD | 3.1.5 | ✅ (Toolkit) |
| Ajuda e tutorial (o que fazer, teclas, primeiras horas) | Alvo | 3.2.1 | 🟡 só a linha de ajuda do HUD |

### B. Explorar o mundo

| Necessidade | Origem | Tela (§3) | Hoje |
|---|---|---|---|
| HUD: local, dinheiro, hora, período e clima, aviso | GDD §7 | 3.2.1 | 🟡 em duas metades |
| "Aperte E para interagir" | GDD | 3.2.2 | ✅ |
| Diálogo com NPC | GDD | 3.2.3 | ✅ linear |
| **Perguntas em conversa** (escolha e texto) | GDD, Backlog | 3.2.4 | ✅ `EscolhaDeRoteiroUI` (provisória, uGUI) |
| Avisos curtos ("Pegou X", "Nada no mato") | Backlog | 3.2.5 | 🟡 cada tela improvisa |
| Transição de cena, área e batalha | GDD §3 | 3.2.6 | 🟡 |
| **Mapa do mundo e localização** (mundo em várias áreas conectadas) | Alvo | 3.2.7 | ❌ |
| **Objetivo atual** (para onde ir agora) | Backlog (missões) | 3.2.7 | ❌ |
| **Obstáculos de campo** ("Deseja usar Cortar?", Forçar, Surfar, quebrar pedra) | Sistema | 3.2.8 | ❌ |
| **Pesca** (esperando, fisgou, escapou) | Sistema | 3.2.8 | ❌ |
| Encontro selvagem: entrada em batalha | GDD §2 | 3.2.6 | 🟡 |
| **Consequência de derrota** ("sem Pokémon em condições", voltar ao curandeiro) | Backlog | 3.2.9 | ❌ |
| Curandeiro | GDD | 3.4.5 | 🟡 sem feedback |

### C. Gerenciar a equipe e o que ela carrega

| Necessidade | Origem | Tela (§3) | Hoje |
|---|---|---|---|
| Equipe: status, HP, item, ordem | GDD §4 | 3.3.2 | ✅ (2×) |
| **Resumo do indivíduo**: natureza, IV/EV (ou julgamento), gênero, XP em número, amizade, habilidade e sua origem | Sistema (varredura 05/09) | 3.3.2 | ❌ |
| **Progresso de evolução** (amizade, críticos, vezes que usou um golpe: são condições de evolução) | Sistema | 3.3.2 | ❌ |
| Golpes do Pokémon (tipo, categoria, poder, precisão, PP, descrição) | GDD §5 | 3.3.2 | 🟡 |
| Mochila por categoria, usar, equipar, descartar | GDD §7 | 3.3.3 | 🟡 (só a Toolkit tem categoria) |
| Pokédex como **ferramenta de decisão** (atributos base, habilidades possíveis, golpes por nível, onde encontrar, formas) | Sistema | 3.3.4 | 🟡 |
| PC / caixa (mover, soltar, caixa cheia) | GDD, Backlog | 3.3.6 | ✅ (2×) |
| **Aprender ou esquecer golpe** (nível, MT, tutor) | GDD §7 | 3.4.3 | 🟡 só dentro da batalha e da mochila |
| **Evolução** (antes → depois, cancelar) | GDD §7 | 3.4.6 | ❌ |
| Usar item de habilidade, MT, pedra, item de forma | Sistema | 3.3.3, 3.4.7 | 🟡 |

### D. Batalha

| Necessidade | Origem | Tela (§3) | Hoje |
|---|---|---|---|
| Quadro de cada posto (1×1 e dupla): HP, nível, status, item | GDD §5 | 3.5.1 | ✅ |
| **Estágios de atributo, forma atual, substituto, marca de proteção** | Sistema | 3.5.1 | 🟡 (estágios e status ✅; o resto ❌) |
| **Clima, terreno e efeitos de cada lado** (Espinhos, telas, Vento a Favor, Trick Room…) | Sistema | 3.5.2 | ❌ |
| **Golpe de vários turnos explicado** (carregando, recarregando, preso repetindo) | Sistema | 3.5.3 | ❌ (lê-se como bug) |
| Painel de ações e golpes com motivo de bloqueio | GDD §5 | 3.5.3, 3.5.4 | 🟡 |
| Efetividade do golpe contra o alvo | Sistema | 3.5.4 | ❌ |
| Alvo em dupla | GDD §5 | 3.5.5 | ✅ |
| Forma em batalha (Mega, Deoxys) | GDD §5 | 3.5.6 | 🟡 ciclo de botão |
| Itens e troca de Pokémon | GDD §5 | 3.5.7 | ✅ |
| Captura | GDD §2 | 3.5.8 | 🟡 sem animação |
| **Registro/histórico da batalha** (o log tipado já existe) | Sistema | 3.5.9 | ❌ |
| **Resultado** (XP, nível, atributos, golpe novo, dinheiro, Pay Day) | GDD §7 | 3.5.10 | 🟡 só narração |
| Batalha de treinador (introdução, revanche, recompensa) | GDD | 3.5.11 | 🟡 |
| **Batalha em rede** (lobby, aguardando, desconexão, latência) | Alvo, GDD §11 | 3.5.12 | ❌ |

### E. Progressão e economia

| Necessidade | Origem | Tela (§3) | Hoje |
|---|---|---|---|
| **Insígnias** (estojo, ao ganhar uma) e ginásios | Backlog | 3.3.7 | ❌ (o cartão mostra "0" para sempre) |
| **Missões e conquistas** (4 estados, objetivo, progresso, recompensa) | Backlog | 3.3.8 | ❌ |
| **Estatísticas de jogo** (base das missões e do cartão) | Backlog | 3.3.5 | ❌ |
| Cartão de treinador com conteúdo real | GDD | 3.3.5 | 🟡 mínimo |
| Loja: comprar, vender, quantidade, **categorias e busca** | GDD | 3.4.1 | 🟡 lista única |
| Tutor de golpe (Pokémon → golpe → preço → esquecer) | GDD | 3.4.2 | 🟡 pela caixa de escolha |
| Trocador (Pokémon por Pokémon) | GDD | 3.4.4 | ❌ |

### F. Mundo compartilhado (alvo: 2 a 18 jogadores)

| Necessidade | Origem | Tela (§3) | Hoje |
|---|---|---|---|
| Ver outros jogadores no mundo (nome sobre o avatar) | Alvo | 3.7 | ❌ |
| Lista de jogadores e convite de batalha ou troca | Alvo | 3.7 | ❌ |
| Chat e emotes | Alvo | 3.7 | ❌ (não planejado; desejável) |
| Estado da conexão (latência, desconectado, reconectando) | Alvo | 3.7 | ❌ |

### G. Sistema, qualidade e acessibilidade

| Necessidade | Origem | Onde | Hoje |
|---|---|---|---|
| Sons de UI (mover, confirmar, cancelar, erro) | Backlog | §5 | ❌ (gancho existe, sem arquivo) |
| Animação de UI e de batalha | Backlog | §5 | ❌ |
| Escala de texto, contraste, não depender só de cor | Alvo (hardware baixo, mobile futuro) | §5 | 🟡 status e tipos têm sigla |
| Painel de análise da IA (só desenvolvimento) | Backlog | 3.6 | ✅ (janela do Editor) |

### H. Mecânicas de formato e conteúdo novo (09/10/2026)

O que Z-move, Dynamax, Tera, bagas e formas presas ao item pedem de tela. O ciclo do botão de forma da `BatalhaUI` já percorre Mega, Z, Dynamax e Tera, mas o resto não existe.

| Necessidade | Origem | Tela (§3) | Hoje |
|---|---|---|---|
| **Escolher o tipo Tera** de cada Pokémon (e ver o tipo Tera no resumo) | Sistema | 3.3.2 | ❌ (`Pokemon.DefinirTipoTera` existe; só o teste chama) |
| Mostrar na batalha **quem está Dynamax** (turnos que faltam), **Terastalizado** (tipo, coroa) e **Stellar** | Sistema | 3.5.1 | ❌ |
| Mostrar **que o lado já usou** a mecânica de formato (só uma por batalha) | Sistema | 3.5.3 | ❌ |
| **Golpe Z / Max** no lugar do golpe comum (nome, tipo, poder) ao armar a mecânica | Sistema | 3.5.4 | ❌ |
| Dizer **por que a mecânica não está disponível** (sem item-chave, pedra/cristal na mão, já usada) | Sistema | 3.5.6 | ❌ |
| Feedback ao **equipar ou remover** item que força forma (placa, Memory, Rusted Sword…) e ao recusar a troca de dono | Sistema | 3.3.3 | ❌ |
| Bagas: aviso quando a baga é comida em batalha (Gluttony, Ripen, Bug Bite, Teatime) | Sistema | 3.5.9 | 🟡 só narração |

---

## 2.1 Retrato do que já está construído

O projeto tem **duas pilhas de UI** que coexistem, com `DadosJogador` em comum:

| Pilha | Onde | O que tem |
|---|---|---|
| **uGUI** (Canvas + TextMeshPro) | `Assets/Scripts/UI/` (17 arquivos, ~3.300 linhas) | Título, batalha, diálogo, interação, HUD de ambiente, loja, cartão, slots de save, configurações, e o menu do jogador (mochila, equipe, Pokédex, caixa) |
| **UI Toolkit** (UXML + USS) | `Assets/Omega/Game/UI/` (6 UXML, 7 USS) + `Assets/Omega/Game/Runtime/UI/` (10 arquivos) | Casca do jogo (menu, criador de personagem, opções, créditos, pausa, HUD), e **de novo** equipe, mochila, PC e Pokédex |

Quatro telas existem **nas duas pilhas** (equipe, mochila, PC/caixa, Pokédex). É a duplicação que o backlog chama de A4: toda mudança em um lado diverge do
**Placeholders uGUI: desligados em 07/10, religados em 12/10 (pedido do dono do projeto, DECISOES T42).** Em 07/10 todo placeholder uGUI que não era de batalha, diálogo ou loja foi desativado, para o outro lado do grupo ver o que faltava; sem eles não havia como salvar, ver o HUD, o aviso "aperte E" nem escolher a dificuldade. Em 12/10 voltaram a ficar ligados no prefab `UI_Manager` e na `Bootstrap`: `AmbienteUI`, `InteracaoUI`, `TrainerCardMenu` (agora com estatísticas e missões em aberto), `SlotsDeSaveMenu`, `ConfiguracoesMenu` (a dificuldade volta a ser escolhível; o dropdown de IVs, que nunca foi autorado, agora é criado pelo painel) e `TelaDeTitulo`. As quatro telas órfãs (`PartyMenu`, `MochilaMenu`, `PokedexMenu`, `CaixaMenu`) também foram religadas, mas continuam sem ninguém que as abra: o `Menu` abre as versões em Toolkit. Todos seguem marcados com `[PlaceholderDeUI]`; procure por ele para achar o que é provisório.
**Entrada (desde 12/10/2026, DECISOES T51).** O jogo começa sempre pela cena `Bootstrap`, que mostra o título uGUI (`TelaDeTitulo`): Continuar, Novo jogo, Sair. O "Novo jogo" carrega a `OmegaMenu` já no criador de personagem em UI Toolkit (`GameUI` + `GameBootstrap`, pelo pedido `Inicializacao.NovoJogoJaEscolhido`), sem mostrar o título Toolkit; o "Voltar" do criador leva a ele. Rodando dentro do jogo, o `GameBootstrap` esconde o jogador da `Bootstrap` (com a câmera dele), põe o estado em `Menu` e, no "Começar", devolve o jogador e carrega `cenaDoMundo` (`KantoBase`, a multicena, nascendo em Pallet; antes era a `DevScene`). Sem a `Bootstrap`, a `OmegaMenu` mantém o modo protótipo de antes. Teste: `EntradaUnicaPlayModeTests`.
outro. **Decisão pendente** (§7): a UI Toolkit substitui a uGUI? Enquanto isso, **tela nova não deve nascer nas duas**.

A pergunta em conversa (escolha e texto do roteiro e dos cards) deixou de ser IMGUI em 11/10: é a `EscolhaDeRoteiroUI`, uGUI, montada por código na primeira vez que alguém pergunta (escolher inicial, tutor, nome do rival, "quer curar?", troca de habilidade). Teclado e controle andam pelos botões.

Números do conteúdo que a UI apresenta (contados em 30/09): **mais de mil espécies** (a Pokédex Toolkit lê 1.338 do catálogo; formas agrupam sob a base), **513 habilidades**, **922 golpes** gerados,
**2.681 itens** (479 com preço, 2.202 de preço 0 que nenhuma loja vende), 3 slots de save.

| Tela | uGUI | UI Toolkit | Estado |
|---|---|---|---|
| Título | `TelaDeTitulo` | `GameUI` (tela-menu) | provisória nos dois; **a uGUI é a entrada** desde 12/10, a Toolkit aparece no "Voltar" do criador; a definitiva vem da branch de assets |
| Criador de personagem | — | `GameUI` (tela-criador) | só Toolkit |
| Opções | `ConfiguracoesMenu` | `GameUI` (tela-opcoes) | **duplicada**, com opções diferentes (§3.1.4) |
| Créditos, Pausa | — | `GameUI` | só Toolkit |
| HUD de exploração | `AmbienteUI` (período/clima) | `GameUI` (tela-hud) | **duas metades do mesmo HUD** |
| Interação ("aperte E") | `InteracaoUI` | — | ok |
| Diálogo (linear) | `DialogoUI` | — | ok, sem ramificação |
| Escolha/texto do roteiro | `EscolhaDeRoteiroUI` | — | provisória, montada por código |
| Menu principal | `Menu` (controlador) | `MenuPrincipalUIToolkit` | **duplicado** |
| Equipe | `PartyMenu` | `EquipeUIToolkit` | uGUI ligada mas sem quem a abra (placeholder); **vale a Toolkit** |
| Mochila | `MochilaMenu` | `MochilaUIToolkit` | uGUI ligada mas sem quem a abra (placeholder); **vale a Toolkit** (tem categorias) |
| Pokédex | `PokedexMenu` | `PokedexUIToolkit` | uGUI ligada mas sem quem a abra (placeholder); **vale a Toolkit** (mais completa) |
| PC / Caixa | `CaixaMenu` | `PcUIToolkit` | uGUI ligada mas sem quem a abra (placeholder); **vale a Toolkit** |
| Cartão de treinador | `TrainerCardMenu` | — | provisório; desde 12/10 mostra estatísticas e missões em aberto |
| Slots de save | `SlotsDeSaveMenu` | — | ok |
| Loja | `LojaMenu` | — | ok, sem categoria nem busca |
| Batalha | `BatalhaUI`, `EscolhaDeBatalhaMenu` | — | completa no básico; **faltam clima, terreno, campo** (§3.5) |
| Visor 3D de Pokémon | — | `VisorPokemon3D` | serve às telas Toolkit |
| Evolução, tutor de golpe, trocador, resultado de batalha | — | — | **não existem como tela** (§3.4, §3.5) |

---

## 3. Catálogo de telas

Cada seção segue o mesmo molde: **Objetivo · Mostra · Chama · Avisa (evento) · Casos de borda · Hoje.**

### 3.1 Entrada no jogo

#### 3.1.1 Tela de título
- **Objetivo:** porta de entrada. Continuar, Novo jogo, Opções, Créditos, Sair.
- **Chama:** `SistemaSave.ExisteSave(slot)` para habilitar "Continuar"; abre os slots (§3.1.3) em vez de carregar direto; "Novo jogo" abre o criador (§3.1.2).
- **Casos de borda:** sem nenhum save, "Continuar" desabilitado (não escondido: o jogador entende por quê); save de versão **mais nova** que a build é recusado com
  frase (`SistemaSave.UltimoErro`) — a tela mostra a frase, não uma exceção.
- **Hoje:** `TelaDeTitulo` (uGUI) e a tela-menu do `GameUI` (Toolkit), ambas provisórias.

#### 3.1.2 Criador de personagem
- **Objetivo:** escolher gênero e peças de aparência; confirmar e começar. Botão aleatório.
- **Mostra:** pré-visualização 3D do avatar (a mesma técnica do `VisorPokemon3D`), lista de peças, nome.
- **Chama:** `PersonagemEscolhido` / `PersonagemNoSave` (Omega). A aparência é gravada no save (formato 3+); saves antigos usavam `PlayerPrefs`.
- **Hoje:** `GameUI` (tela-criador: masculino/feminino, lista, voltar, aleatório, confirmar). Falta o **nome do personagem** como campo próprio.

#### 3.1.3 Slots de save
- **Objetivo:** três slots: resumo, salvar, carregar, apagar.
- **Mostra por slot:** nome, dinheiro, equipe (ícones), tempo, cena/local, versão do formato, data. Slot vazio.
- **Chama:** `SistemaSave.Salvar(slot)`, `Carregar(slot)`, `Apagar(slot)`, `ExisteSave(slot)`; mostra `UltimoErro` quando falha.
- **Casos de borda:** **confirmação antes de sobrescrever e de apagar** (destrutivo); save v2 migrado só na memória — o arquivo vira v4 no próximo Salvar (avisar
  "formato antigo, será atualizado"); slot corrompido ou de build mais nova: frase clara, sem apagar.
- **Hoje:** `SlotsDeSaveMenu`. Em aberto: o "Jogo salvo no slot N" ainda é só `Debug.Log` com caminho de arquivo — o jogador deveria ver uma confirmação curta.

#### 3.1.4 Opções
- **Objetivo:** vídeo, áudio, controles, jogo.
- **Mostra:** resolução, tela cheia, qualidade, limite de FPS, volume geral/música/efeitos, sensibilidade da câmera, **dificuldade** (afeta o perfil de IA dos treinadores).
- **Chama:** `PlayerPrefs`; dificuldade pelo sistema de dificuldade; **rebind de teclas** (a mapa `Inputs.inputactions` já suporta) — hoje só existe sensibilidade.
- **Hoje:** duas implementações com opções **diferentes** (uGUI: volume/tela cheia/qualidade/dificuldade; Toolkit: resolução/tela cheia/qualidade/FPS/volume/sensibilidade).
  A definitiva junta as duas listas.

#### 3.1.5 Créditos e Pausa
- **Créditos:** texto rolável. **Pausa:** continuar, opções, voltar ao menu, com o resumo da partida. Hoje só na `GameUI` (Toolkit).

#### 3.1.6 Tela de carregamento — **não existe**
- **Objetivo:** cobrir a carga entre o título e a primeira cena, e a entrada em cada cena grande. **Medido:** até a primeira cena a carga leva **8 s (disco frio) a 62 s** na máquina mínima
  (Kanto3D); a multicena parte isso em cenas de área, mas a carga inicial e o teleporte para uma área distante ainda esperam.
- **Mostra:** barra ou indicador **de verdade** (progresso da carga assíncrona, não um giro fixo), dica/ajuda rotativa, nome do local. Nunca tela preta muda por mais de 1 s.
- **Chama/lê:** `SistemaTransicao` (estado `Transicao`, o jogador só é liberado quando posicionado), `CarregadorDeAreas` (áreas chegando: `CarregadasOuChegando`).
- **Casos de borda:** falha de carga (cena ausente do Build Settings) = frase e volta ao título, não travar; primeira vez em disco frio muito mais lenta que a segunda.

### 3.2 Exploração

#### 3.2.1 HUD
- **Mostra:** nome do jogador, local (cena/área), dinheiro, hora do jogo, período do dia e clima, aviso curto ("Pegou X", "Nada no mato", "Nenhum Pokémon em condições"), ajuda de teclas.
- **Avisa:** `SistemaMundo.OnClimaMudou`, `OnPeriodoMudou`, `OnAmbienteMudou`; `DadosJogador` para dinheiro.
- **Hoje:** dividido entre `AmbienteUI` (período/clima e quanto falta para virar) e a tela-hud do `GameUI`. Deve ser **um** HUD.
- **Ao entrar na multicena:** o "local" muda quando a área muda de cena; o HUD lê a cena de área carregada, não o nome da cena base.

#### 3.2.2 Prompt de interação
- **Mostra:** o texto do `Interagivel.textoInteracao` ("Conversar", "Fazer compras", "Ensinar golpe"…) quando o jogador está no gatilho. Some quando abre diálogo/menu.
- **Hoje:** `InteracaoUI`. Ok.

#### 3.2.3 Caixa de diálogo
- **Mostra:** fala atual, nome de quem fala (opcional), indicador de "há mais".
- **Chama:** `SistemaDialogo` (fila linear). **Avisa:** `OnFala`, `OnDialogoIniciado`, `OnDialogoTerminado`.
- **Casos de borda:** fala longa (quebra), avançar com `Submit`, pular tudo, **não deixar o jogo andar** enquanto ativo.
- **Hoje:** `DialogoUI`. Diálogo **ramificado** não existe (adiado de propósito); a ramificação de hoje é o prompt de escolha do roteiro (§3.2.4).

#### 3.2.4 Prompt de escolha e de texto (roteiro) — **provisório**
- **Objetivo:** perguntar e receber uma opção (`PassoEscolha`) ou um texto curto (`PedirTexto`, nome do rival com limite de 12).
- **Contrato:** `IMundoDoRoteiro.Escolher(contexto, pergunta, opcoes) → índice em contexto.UltimaEscolha`; `PedirTexto(contexto, pergunta, padrao) → contexto.UltimoTexto`.
  O sistema **espera** a resposta; a tela precisa respeitar isso e nunca deixar a pergunta sem saída (`Cancel` escolhe o padrão ou volta, conforme o passo).
- **Consumidores:** escolha de inicial, **tutor de golpe** (Pokémon → golpe → preço), trocador, qualquer NPC com roteiro.
- **Hoje:** `EscolhaDeRoteiroUI` (uGUI, 11/10), chamada por `MundoDoRoteiroNaCena`: pergunta e botões, o primeiro já selecionado, teclado e controle pelos botões. Falta a aparência definitiva e decidir a pilha (§7).

#### 3.2.5 Notificações curtas
- Item obtido, evolução, "salvo", erro de ação. Fila com tempo, não sobrepor diálogo. Hoje cada tela improvisa.

#### 3.2.6 Transição de cena/área
- Fade ao trocar de cena (`SistemaTransicao`) e ao entrar em batalha. Em multicena a **troca de área é invisível**: a UI só precisa não travar o quadro na carga.

#### 3.2.7 Mapa do mundo e objetivo atual — **não existe**
- **Objetivo:** o alvo declarado é um **mundo em várias áreas conectadas**: o jogador precisa saber **onde está**, **para onde ir** e **o que já visitou**.
- **Mostra:** mapa por região/áreas (as áreas da multicena são a divisão natural: Pallet, Viridian, Pewter, Celadon, Lavender, Vermilion, Fuchsia, Cinnabar, Liga, Sevii…), posição do jogador,
  locais visitados (Centros Pokémon, ginásios, PC), e o **objetivo atual** (linha no HUD) quando as missões existirem (§3.3.8).
- **Chama/lê:** posição do jogador e `SpawnDaCena`/`PontoTransicao`; visitados = flags em `DadosJogador`.
- **Casos de borda:** interior (sem mapa, mostra o exterior de onde entrou), área ainda não conhecida (esmaecida), multiplayer (posição de outros jogadores, §3.7).

#### 3.2.8 Obstáculos de campo e pesca — **não existem como tela**
- **Obstáculos** (`ArvoreCortavel`, `RochaQuebravel`, `PedraDeForca`, `ObstaculoDeCampo`, regras em `RegrasDeCampo`): ao interagir, **pergunta** ("Deseja usar Cortar?") e **explica** quando falta
  o Pokémon que sabe o golpe ou a MO. Sim/Não pelo prompt do §3.2.4; a frase de recusa vem da regra, nunca da tela.
- **Pesca** (`PontoDePesca`): estados esperando → fisgou (janela curta para responder) → escapou/pegou. Indicador visual e sonoro do momento certo; nada de precisar ler texto para acertar o tempo.

#### 3.2.9 Derrota e "sem Pokémon em condições" — **só texto**
- **Objetivo:** quando a equipe cai, o jogador precisa entender **o que aconteceu e o que perdeu** (hoje a consequência é só o tempo de voltar ao curandeiro; perder dinheiro ou voltar a um ponto é decisão aberta).
- **Mostra:** frase da derrota, o custo (se houver) e para onde foi levado. Tentar entrar em batalha sem Pokémon em condições mostra a razão ("Nenhum Pokémon em condições"), que hoje já fala com o jogador.

### 3.3 Menu do jogador

O menu abre por `AlternarMenu` (Escape) e não abre em batalha nem em diálogo. Toda tela filha herda de `PainelMenu` (slide + fade) e o `Menu` manda no movimento do jogador.

#### 3.3.1 Menu principal
- Lista: Pokédex, Equipe, Mochila, Cartão, Salvar, Opções, Sair. **Chama:** cada `Abrir*`. **Hoje:** `Menu` (uGUI) e `MenuPrincipalUIToolkit`, duplicados.

#### 3.3.2 Equipe e resumo do Pokémon
- **Objetivo:** ver e organizar a equipe; é também o **seletor de alvo** de item, de MT e de tutor.
- **Lista (até 6):** ícone/retrato, apelido/nome, nível, barra de HP, status (`StatusData.Sigla` + `Cor`), item segurado, gênero.
- **Ficha (resumo):** ver a §4.1 — nome, espécie **e forma**, tipos, gênero, natureza, habilidade (com marca **oculta** e a **concedida por item**), item segurado (com o
  estado "desativado" na batalha), golpes com PP/tipo/categoria/poder/precisão, atributos, XP/nível seguinte, amizade, e o **progresso de evolução** (amizade, críticos acertados e vezes que usou um golpe são **condições de evolução**: sem isso o jogador não percebe que está perto de uma). Visor 3D opcional.
- **Ações:** trocar de lugar, dar/tirar item segurado (`SistemaInventario.RetirarHeldItem`, só fora de batalha), ver resumo, **usar item no Pokémon**.
- **Casos de borda:** Pokémon desmaiado (cinza), em forma escolhida (Deoxys), de Mega; apelido longo; ficha cheia ao aprender golpe (§3.4.3).
- **Hoje:** `PartyMenu` e `EquipeUIToolkit`. **Nenhuma mostra natureza nem gênero** (M4 deixou isso "com o outro lado do grupo").

#### 3.3.3 Mochila
- **Categorias** vindas do **tipo** do asset (`ItemCura`, `ItemPokebola`, `ItemEvolucao`, `ItemAtributo`, `HeldItemData`, `ItemEnsinaGolpe`, `ItemChave`, `ItemDeHabilidade`…),
  não de um campo novo. Lista com quantidade, detalhe com `Descricao`, botão Usar/Equipar/Descartar.
- **Chama:** `SistemaInventario.Usar(item, alvo)`, `Adicionar`, `Remover`; MT/MO: `ConfirmarEnsino(item, alvo, indiceParaEsquecer)`. **Avisa:** `OnInventarioMudou`.
- **Casos de borda:** item que pede alvo abre a Equipe como seletor; uso recusado devolve `UltimaMensagem` (mostrar); **MO não gasta**; MT com ficha cheia abre a escolha do golpe a esquecer (§3.4.3);
  **item composto** (`CapsulaDeHabilidade:<id>`): o nome vem da habilidade — o nome exibido já é montado pelo item, a tela só lê `NomeExibicao`/`Descricao`;
  item de história (preço 0, `ItemChave`) não é vendável nem descartável.
- **Hoje:** `MochilaMenu` (sem abas) e `MochilaUIToolkit` (com categorias). A Toolkit é a referência.

#### 3.3.4 Pokédex
- **Lista:** > 1.000 espécies, montada **em blocos** (criar tudo de uma vez trava o Editor). Estados: não visto / visto / capturado.
- **Ficha:** imagem, número, nome, tipos, categoria, altura, peso, habitat, descrição, **formas** (uma entrada só por espécie: o agrupamento é `EspecieData.formaDe`;
  o resto é aba), **atributos base**, habilidades da espécie (com marca oculta), onde encontrar, evoluções e como (condições), golpes por nível. É a tela onde essas informações ajudam a **decidir** (hoje é só um registro de coleção).
- **Chama:** `DadosJogador` (espécies registradas), `CatalogoJogo`. **Casos de borda:** forma alternativa vista mas base não (usa o melhor estado entre a entrada e as formas); espécies sem fonte.
- **Hoje:** `PokedexMenu` e `PokedexUIToolkit`; a Toolkit tem tira de ícones, ficha e abas.

#### 3.3.5 Cartão de treinador
- Nome, retrato, tempo de jogo, **insígnias**, Pokédex (vistos/capturados), dinheiro. **Insígnias é campo morto** (nada incrementa fora de `PassoDarInsignia`); estatísticas de jogo não existem. Hoje mínimo.

#### 3.3.6 PC / Caixa
- Equipe à esquerda, **grade da caixa** no centro (páginas de 30 sobre uma lista plana), ficha à direita. Mover, trocar, soltar (com confirmação), **caixa cheia** (avisa antes de capturar).
- **Hoje:** `CaixaMenu` e `PcUIToolkit`. Capacidade da caixa não é configurável.

#### 3.3.7 Insígnias e ginásios — **não existe**
- **Objetivo:** o campo `DadosJogador.badges` é salvo e mostrado ("Insígnias: 0") mas **nada o incrementa**: o jogo não tem ginásio nem líder. Quando existir, o jogador precisa **ver o que ganhou**.
- **Mostra:** **estojo** com as 8 insígnias (obtida/não, nome, líder, cidade, o que libera), **tela de recompensa** ao ganhar uma (`PassoDarInsignia`: liga a flag `FLAG_BADGE0N_GET` e sobe `badges`),
  e, no ginásio, o que falta (líder, treinadores do caminho).
- **Depende de:** decisão de produto (§7): virar sistema de ginásios, ou eixo de **escala de nível** (padrão R.O.W.E.), ou tirar o campo.

#### 3.3.8 Diário de missões e conquistas — **não existe**
- **Objetivo:** hoje só existem **flags booleanas** (aconteceu ou não). O backlog decidiu o modelo de **quatro estados** (não iniciada, ativa, concluída, falhada) com condição declarativa e **progresso por contagem**, e
  isso pede **estatísticas de jogo** que também não existem.
- **Mostra:** lista por estado, objetivo, **progresso** ("3/5"), recompensa, o local (liga com o mapa §3.2.7) e um aviso curto quando uma missão muda de estado (§3.2.5). Conquistas: lista, desbloqueada/bloqueada, data.
- **Chama/lê:** o sistema de missões (a criar) sobre `DadosJogador.Flags` e as estatísticas. **A tela não define missão**: só lê o estado e mostra.

### 3.4 Interações de mundo

#### 3.4.1 Loja
- **Mostra:** dinheiro, lista de `LojaData.ItensVendidos` (nome, ícone, **preço**, quantidade que você tem), detalhe, **seletor de quantidade**, total, comprar/vender (venda = metade do preço, mínimo 1).
- **Chama:** `SistemaLoja` (compra/venda); abre por `Vendedor.Interact` → `Menu.AbrirLojaDoMundo(loja)`. **Avisa:** `OnLojaMudou`.
- **Casos de borda:** dinheiro insuficiente (desabilita e diz), item que a loja não compra (`PodeVender`), **lista enorme** — a loja de teste tem **479 itens**: precisa de **categorias, busca e lista virtualizada**;
  mochila cheia (não há limite hoje, mas a tela não deve assumir).
- **Hoje:** `LojaMenu` (uGUI). Sem categoria, sem busca.

#### 3.4.2 Tutor de golpe (`NpcEnsinaGolpe`)
- **Fluxo:** escolher Pokémon da equipe (Equipe como seletor) → lista dos golpes que **aquele** Pokémon aprende por tutor (`EspecieData.AprendePorTutor`), com custo (grátis, dinheiro ou item) e "já sabe"/"já ensinado" →
  confirmar preço → se a ficha está cheia, escolher o golpe a esquecer (§3.4.3) → cobra **só depois**.
- **Resultados que a tela distingue** (`ResultadoDeEnsino`): Inválido, Incompatível, JáSabe, JáEnsinado, SemPagamento, Aprendeu, FichaCheia. Cada um tem frase própria.
- **Hoje:** roda pela caixa de escolha (§3.2.4) — **não existe tela própria**. São 14 tutores no mapa.

#### 3.4.3 Aprender golpe com a ficha cheia
- **Mostra:** o golpe novo (tipo, categoria, poder, precisão, PP, descrição) e os 4 conhecidos; escolher qual esquecer, ou desistir.
- **Chama:** `SistemaInventario.ResolverGolpePendente(alvo, indiceParaEsquecer, out mensagem)` — **a mesma função** para nível, MT, tutor. Desistir não gasta nada.
- **Hoje:** existe dentro da `BatalhaUI` (`painelAprenderGolpe`) e dentro da mochila (uso de MT). **Falta como tela única reutilizável** (nível fora de batalha, tutor).

#### 3.4.4 Trocador (`NpcTrocador`)
- Escolher qual Pokémon da equipe trocar e o que se recebe; mostra o que o NPC oferece. Evoluções por troca dependem disso (o validador cobra que exista um trocador). **Sem tela.**

#### 3.4.5 Curandeiro, PC no mapa, ponto de armazenamento
- Curandeiro: fala + animação/feedback de cura, sem menu. Ponto de armazenamento: abre o PC (§3.3.6).

#### 3.4.6 Evolução
- **Mostra:** "Algo está acontecendo…", sprite antes → depois, novo nome, **habilidade nova se mudou**, golpes que aprendeu na evolução, opção de **cancelar** (tecla `Cancel`).
- **Chama:** `SistemaEvolucao.TentarEvoluir(...)` devolve uma mensagem; hoje o resultado é só texto. A tela precisa do **antes/depois** (espécie anterior e nova) — pedir o evento/estrutura ao sistema.
- **Hoje:** **não existe tela**; a evolução só narra.

#### 3.4.7 Usar item de habilidade / item de forma
- Usar `CapsulaDeHabilidade:<id>` no Pokémon: confirmação com a habilidade atual → nova, e o efeito ("para sempre; a Mega usa a dela enquanto dura"). Recusa se já tem.

### 3.5 Batalha

`BatalhaUI` escuta o `SistemaBatalha`; **nunca altera** estado (regra §1.1). Formatos: 1×1 e **dupla** (2×2, também de treinador). Ritmo: `RitmoDeBatalha` (o teste usa `Instantaneo`).

#### 3.5.1 Quadro de cada posto (aliado, aliado 2, adversário, adversário 2)
- Sprite (frente/costas), nome, nível, **barra de HP** (números só do jogador), barra de **XP** (só aliado), **status** (sigla + cor), **item segurado** (ícone, esmaecido se desativado), destaque de quem escolhe agora (dupla).
- **Falta:** gênero, **estágios de atributo** (▲▼ por atributo), **forma atual** (Mega/Deoxys), **substituto**, marca de "protegido", contagem do **Canto da Morte**, tipo assumido.

#### 3.5.2 Campo (**não existe hoje**)
- **Clima** em vigor (inclusive **forte e o dono**: Desolate Land, Primordial Sea, **Ventos fortes**) e turnos restantes; **terreno** (Campo Elétrico/Grama/Nebuloso/Psíquico); **efeitos de cada lado** (Espinhos e suas camadas, telas Reflect/Light Screen, Vento a Favor, Trick Room, Gravidade, Salas…) com turnos.
- **Chama/lê:** `SistemaBatalha.ClimaEmVigor`, `DonoDoClimaForte`, `TerrenoAtual`, `CampoAliado`, `CampoInimigo` (lista de `EfeitoDeCampoAtivo` com `dados`, `turnosRestantes`, `camadas`).
- **Hoje:** a `BatalhaUI` tem **zero** referência a clima, terreno ou campo (o backlog já registra "UI mostra clima/terreno do mundo na batalha").

#### 3.5.3 Painel de ações
- **Lutar / Mochila / Pokémon / Fugir.** Em batalha de treinador, Fugir vira desabilitado com a frase. **Chama:** `EscolherGolpe`, `EscolherTroca(indice)`, `EscolherFuga()`, item pela mochila de batalha. `OnPedirAcao` liga o painel; ele se fecha sozinho quando a escolha é aceita.
- **Casos de borda:** escolha recusada (retorna `false`: mostrar o motivo, não fechar); **duas escolhas por turno em dupla** (um posto de cada vez: `AliadoDaVez`); forçado (carga, repetição, recarga) — **não há escolha**, o turno segue sozinho, e **a tela precisa dizer por quê** ("X está carregando…", "X precisa recarregar", "X está preso repetindo Y por N turnos" — `TravaDeAcao`): hoje lê-se como bug.

#### 3.5.4 Golpes
- Lista dos 4 (ou menos): nome, tipo (cor), **categoria** (físico/especial/status), **PP** atual/máx, poder e precisão; desabilitado quando sem PP, **proibido** (Taunt, Encore, Disable, Torment, Assault Vest…) ou **preso a uma trava** — sempre com o **motivo** (a mesa devolve `motivo`).
- **Falta:** **efetividade** contra o adversário atual (super/pouco/imune) — dado disponível em `CalculoDano.Efetividade`, mas a UI deve pedir uma estimativa **sem consumir sorteio**.

#### 3.5.5 Alvo do golpe (dupla)
- Para golpes de **um alvo** com mais de um candidato: lista dos candidatos (`OponentesEmCampo` / `AliadosEmCampo`), nome + HP. Golpes de área, de si mesmo e de aliado único resolvem sozinhos. `AbrirEscolhaDeAlvo` já existe; falta indicar **quem seria desviado** por chamariz (Follow Me) — só depois de escolher, não antes.

#### 3.5.6 Forma em batalha
- **Um botão** que percorre **nenhuma → Mega (se o item da) → cada forma alternável** (Deoxys) e mostra qual está escolhida: "Forma: Mega" / "Forma: <nome>".
- **Chama:** `SistemaBatalha.PodeAtivarFormaEmBatalha`, `FormasAlternaveis`, `PodeTrocarDeForma`; envia junto do golpe: `EscolherGolpe(indice, alvo, ativarForma, trocarParaForma)`. A troca é **livre** (não gasta o turno) e **permanente** (Deoxys); a Mega é temporária.
- **Hoje:** um botão de ciclo na `BatalhaUI`. Um seletor próprio (lista) é decisão de UI aberta (§7).

#### 3.5.7 Itens de batalha e troca de Pokémon
- Mochila filtrada por `UsavelEmBatalha`; equipe com estado (HP, status, desmaiado). Troca **forçada** (o ativo caiu) sem opção de voltar; troca **voluntária** com Cancelar.

#### 3.5.8 Captura
- Escolher bola e alvo; animação/feedback (tremidas), resultado, "capturado! vai para a caixa/equipe", pergunta de apelido (usa §3.2.4 texto). Sem bola, sem captura de treinador.

#### 3.5.9 Narração e registro
- Linha de narração (uma por vez, ritmo do `RitmoDeBatalha`) e **histórico rolável** (`Registro.Falas()`) para o jogador reler; **pular** narração. Mensagens novas frequentes: "X perdeu Y!" (item derrubado), "X mudou para Deoxys-Speed!", "Moedas se espalharam", "repetiu a danca".

#### 3.5.10 Resultado
- Vitória: XP ganho por Pokémon (barra subindo), **subiu de nível** (atributos antes/depois), **aprendeu golpe** (§3.4.3), dinheiro do treinador, **moedas do Pay Day**, item de Pickup. Derrota: frase e retorno ao curandeiro. Fuga/captura: encerra.

#### 3.5.11 Batalha de treinador
- Introdução (nome, classe, time restante como bolinhas), falas de desafio/derrota, recompensa (`NpcTreinador`), **revanche** quando permitida.

#### 3.5.12 Batalha em rede (ainda sem tela)
- `BatalhaEmRede` já traduz escolhas e estado para dados. Falta: lobby/convite, "aguardando o outro jogador", desconexão, e o indicador de **latência**. Multiplayer é em rede, não local.

### 3.6 Ferramentas de desenvolvimento

- **Modo desenvolvedor** (`ModoDesenvolvedor`, `OmegaModoDev`): não faz parte do produto; fica fora do build de jogador.
- **Análise de IA** (`SistemaBatalha.OnAnaliseDeIA`): painel de depuração com as ações candidatas e a nota de cada uma — útil ao balanceamento, opcional.

---
### 3.7 Mundo compartilhado (alvo: 2 a 18 jogadores) — **nada existe ainda**

O alvo declarado é multijogador **em rede** (não local). A batalha já é dado serializável (`AcaoDeRede`, foto do estado, árbitro), e o mundo terá **co-presença** (`AvatarRemoto`, `DadosJogador.Local`). A UI precisa de:

- **Presença:** nome (e talvez título/insígnia) sobre cada avatar remoto, legível a distância e sem poluir; ocultável nas Opções.
- **Lista de jogadores** da instância/área (nome, local) e **interação com outro jogador**: convidar para **batalha** (com o formato 1×1 ou dupla) ou **troca**, aceitar, recusar, cancelar.
- **Batalha em rede:** ver §3.5.12. **Troca em rede:** as duas pontas escolhem o Pokémon, **as duas confirmam**, e o resultado fecha junto (nunca uma ponta sozinha).
- **Estado da conexão:** conectado, latência, reconectando, desconectado (com o que se perdeu). **Nunca travar o quadro esperando a rede.**
- **Chat e emotes:** **não planejados**; ficam como desejáveis. Se entrarem, moderação e bloqueio são requisito, não enfeite.
- **Regra de determinismo:** a UI de batalha em rede mostra o que o **árbitro** decidiu, nunca o palpite local; o ritmo de narração é separado da regra.


## 4. O que cruza todas as telas

### 4.1 Dados do Pokémon que uma tela pode precisar

De `Pokemon` (leia por propriedade, nunca por campo):

| O que | De onde | Cuidado |
|---|---|---|
| Nome / apelido | `Nome`, `Apelido` | apelido vazio → nome da espécie |
| Espécie e **forma** | `Especie`, `EspecieBase`, `EstaTransformado` | Deoxys **troca a espécie de verdade** (some o "voltar"); Mega é temporária |
| Nível, XP | `Nivel`, `Experiencia`, `XpNesteNivel`, `XpParaSubir` | |
| HP | `HpAtual`, `HpMaximo`, `PorcentagemHp`, `Desmaiado` | |
| Tipos | `TipoPrimario`, `TipoSecundario`, `TemTipoAssumido` | tipo assumido (Protean/Camouflage) some ao sair de campo |
| Gênero | `Genero` | sem gênero existe (Magnemite, lendários) |
| Habilidade | `Habilidade`, `TemHabilidadeOculta`, `HabilidadeConcedida`, `TemHabilidadeCopiada` | ordem de precedência: copiada (Trace) > concedida > slot; **de Mega vale a da forma** |
| Item segurado | `HeldItem` (o que é do Pokémon), `HeldItemEmUso`, `ItemDesativado` | mostre **o item**; se `ItemDesativado`, esmaecido + "perdeu" só em batalha |
| Golpes | `Golpes` (`GolpeConhecido`: `golpe`, `ppAtual`) | PP do asset (`golpe.Pp`) é o máximo |
| Status | `StatusAtivos` (`StatusData.Sigla`, `Cor`, `NomeExibicao`) | volátil some ao sair de campo; **sigla + cor** (nunca só cor) |
| Estágios | `Estagio(AtributoBatalha)` | −6…+6; só em batalha |
| Atributos | `AtributoEmBatalha`, base e efetivo | mostrar o **efetivo** só em batalha |
| Amizade | `Amizade` | |
| Natureza | `Natureza` (favorecido/prejudicado) | **hoje nenhuma tela mostra** |
| IV / EV | `Iv(...)`, `Ev(...)` | decisão de produto: mostrar? (§7) |

### 4.2 Eventos que a UI escuta (e nada mais)

`GerenciadorEstados.OnEstadoMudou` · `SistemaBatalha.OnBatalhaIniciada`, `OnMensagem`, `OnPedirAcao`, `OnAtualizarStatus`, `OnAliadoTrocado`,
`OnBatalhaTerminada`, `OnPedirEscolhaDeGolpe`, `OnAnaliseDeIA` · `SistemaDialogo.OnFala`, `OnDialogoIniciado`, `OnDialogoTerminado` ·
`SistemaInventario.OnInventarioMudou` · `SistemaLoja.OnLojaMudou` · `SistemaMundo.OnAmbienteMudou`, `OnClimaMudou`, `OnPeriodoMudou` ·
`DadosJogador.OnEquipeMudou` · `ExecutorDeRoteiro.OnRoteiroIniciado`, `OnRoteiroTerminado`.
**Toda inscrição em `OnEnable` tem a desinscrição em `OnDisable`** — e o `OnDisable` não pode assumir que o sistema ainda existe (ao sair do Play Mode ele já foi destruído: dois `NullReferenceException` de
`AtalhoPokedex.OnDisable` e `AnimadorDoJogador.OnDisable` aparecem a cada saída).

### 4.3 Input e navegação

- **Mapas:** `Player` (Move, Look/Camera, Correr, Pular, Interagir, Espreitar, Dançar, AlternarMenu, ProxInteragivel) e `UI` (Navigate, Submit, Cancel, Point, Click, RightClick, MiddleClick, ScrollWheel).
- **Convenções:** `Submit` confirma, `Cancel` volta um nível (nunca fecha o jogo), `AlternarMenu` fecha tudo, `Navigate` move o foco. Foco **sempre visível** e sempre em **um** elemento ao abrir a tela.
- **Teste de teclado exige a Game View focada:** sem foco o Input System descarta o evento. Teste de UI por teclado deve declarar isso e pular (não falhar) quando o foco falta.
- **Dica de teclas** em cada tela (rodapé), lida da action ativa (teclado ou controle).

### 4.4 Elementos que se repetem (componentes)

Construir **uma vez** e reusar: barra de HP (cor por faixa), ícone de status (sigla+cor), pílula de **tipo** (cor+texto), ficha de golpe (`UtilUI` já monta), seletor de Pokémon (Equipe), seletor de quantidade, lista com foco, caixa de confirmação (Sim/Não com padrão seguro),
toast (§3.2.5), e o retrato de Pokémon (sprite pelo caminho em `Resources`, **nunca referência direta** — ver §5).

---

## 5. Requisitos que não são de tela nenhuma

**Desempenho (máquina mínima).**
- **Imagens de espécie não podem ser referenciadas por asset carregado no bootstrap.** O `CatalogoJogo` vive na partida inteira e a Unity carrega tudo que ele referencia: retrato e folha de batalha moram em `Resources` e a espécie guarda o **caminho** (`SpritesDoCatalogo`). Foi o que tirou 985 → 77 MB de textura antes do título. Tela nova **carrega sob demanda e libera** (o cache é solto no fim da batalha).
- **Listas longas** (Pokédex > 1.000, loja de 479, mochila): montar em blocos/virtualizar; nunca instanciar tudo. A Pokédex já mostra o padrão.
- **Sprites de batalha:** 845 folhas sem compressão (~759 MB descomprimidas); tiras acima de 4.096 px (a maior tem 13.625) precisarão virar grade para mobile. A UI pede o quadro certo, não a folha.
- **Sem GC por quadro:** nada de `string` montada em `Update`; atualize o texto **só quando o valor muda**.
- **Meta:** medir a abertura de cada tela **numa build**, não no Editor (o HD mecânico oscila 2,5× entre rodadas). "Não medi" quando não mediu.

**Acessibilidade.**
- Nunca **só cor**: status tem sigla, tipo tem texto, efetividade tem palavra. Contraste mínimo 4,5:1 no texto de corpo.
- Escala de texto ajustável (Opções); tamanho mínimo legível a 1080p **e** em 720p (a máquina mínima roda baixo).
- Frases curtas; sem depender de áudio (não há áudio importado ainda, então isso vale hoje por acidente).

**Áudio de UI.** O sistema (`SistemaAudio`) existe desde 05/09, mas **nenhum arquivo de som foi importado**. A tela deve ter o gancho (mover foco, confirmar, cancelar, erro, item, cura) mesmo sem o som, para não ser refeita quando ele chegar.

**Animação.** Zero animação de gameplay hoje. Toda transição de tela é slide+fade do `PainelMenu` ou `class hidden` do Toolkit. Combine com o restante antes de animar por tela.

**Resolução.** Referência 1920×1080, mínimo 1280×720; âncoras, não posições absolutas. Multiplayer e mobile são metas futuras: nenhuma tela deve assumir mouse ou teclado físico.

**Testes.** Tela nova ganha teste de **PlayMode dirigido pelo jogador**, não só pela API: o padrão é o `SaveViaJogadorTests` (tecla de verdade, `onClick` de verdade). O risco real nunca esteve em chamar a função e sim em o botão não estar ligado. Use o `ArnesDoJogador`.

---

## 6. Ordem sugerida

Ordem = custo de adiar × quantas telas travam por isso.

| # | O que | Por que agora | Depende de |
|---|---|---|---|
| **P0-1** | ~~Prompt de escolha e de texto~~ **feito em 11/10** (`EscolhaDeRoteiroUI`, provisório) | Todo NPC com pergunta passa por ele: tutor, inicial, nome do rival | — |
| **P0-2** | **Tela única de "aprender golpe / esquecer golpe"** | Hoje vive dentro de `BatalhaUI` e da mochila; o tutor e o nível fora de batalha não têm | — |
| **P0-3** | **Campo na batalha** (clima, terreno, efeitos de lado, forma, substituto, **trava de ação explicada**) | Existem no motor e o jogador não vê | — |
| **P0-4** | **Tela de carregamento com progresso** | Carga de 8 a 62 s na máquina mínima; hoje o jogador vê tela parada | — |
| **P1-1** | Loja com categorias, busca e quantidade | 479 itens numa lista só é inutilizável | — |
| **P1-2** | Tutor de golpe (fluxo completo com preço) | 14 NPCs esperam | P0-1, P0-2 |
| **P1-3** | Tela de evolução | Evoluir hoje é só uma frase | pedir `antes/depois` ao sistema |
| **P1-4** | Resultado de batalha (XP, nível, dinheiro, Pay Day) | Feedback de tudo o que a batalha ganhou | — |
| **P1-5** | Equipe/resumo com natureza, gênero, habilidade (oculta/concedida) | Dado existe e não aparece | decisão IV/EV (§7) |
| **P2-1** | Título e criador de personagem definitivos; nome do personagem | Provisórios | branch de assets |
| **P2-2** | Cartão de treinador com conteúdo real; **estojo de insígnias** | Depende de insígnias e estatísticas | decisão de ginásios (§7) e estatísticas |
| **P2-2b** | **Mapa do mundo** e **objetivo atual** | Alvo: várias áreas conectadas; a multicena já as separa | áreas definidas com o grupo do mapa |
| **P2-2c** | **Diário de missões e conquistas** | Só existem flags booleanas | modelo de 4 estados + estatísticas (sistemas a criar) |
| **P2-2d** | Obstáculos de campo (pergunta e recusa) e pesca | Hoje o jogador não é avisado do que falta | P0-1 |
| **P2-2e** | Derrota com consequência visível | Hoje só custa o tempo de voltar | decisão de consequência (§7) |
| **P2-3** | Trocador, curandeiro, item de habilidade | Baixa frequência | — |
| **P2-4** | **Mundo compartilhado** (presença, lista, convites, troca, conexão) e batalha em rede (lobby, aguardando, desconexão) | Alvo declarado: 2 a 18 jogadores | transporte de rede (não existe) |
| **P2-5** | Rebind de teclas; escala de texto; aviso de save | Qualidade de vida | — |
| **fim** | **Unificar as 4 telas duplicadas** numa pilha só | Corta a manutenção pela metade | decisão da pilha (§7) |

---

## 7. Decisões que precisam de resposta

| Decisão | Opções | Sugestão | Custo de errar |
|---|---|---|---|
| **A pilha** — UI Toolkit substitui a uGUI? | (a) Toolkit substitui, migra tela a tela; (b) uGUI fica e a Toolkit vira só a casca; (c) as duas para sempre | (a): a Toolkit já tem as 4 telas do menu **e** categorias/abas; a batalha migra por último, por ser a maior. Enquanto não decidir, **tela nova em Toolkit** | Cada semana com duas pilhas é mudança pela metade |
| **IV/EV na ficha** | mostrar sempre / só "julgar" (estilo "excelente") / nunca | julgamento em palavras; números só em modo dev | Só de produto |
| **Seletor de forma** | ciclo de um botão (hoje) / lista | lista quando houver mais de 2 opções | Baixo |
| **Efetividade no menu de golpes** | sempre / só depois de o adversário ser visto / nunca | só depois de visto (respeita o que o jogador já sabe) | Afeta o desafio |
| **Diálogo ramificado** | adiado; ao voltar, muda o `SistemaDialogo` e §3.2.4 | esperar o material externo | Retrabalho do prompt |
| **Confirmação de save** | mensagem na tela / silencioso | mensagem curta | Baixo |
| **Loja: itens de preço 0** | dar preço / manter fora | decisão de conteúdo (são 2.202) | Loja de teste com 479 |
| **Ginásios e insígnias** | sistema de ginásios / eixo de escala de nível (R.O.W.E.) / tirar o campo `badges` | decidir antes de desenhar o estojo | Tela de um sistema que talvez não exista |
| **Missões** | 4 estados com condição declarativa (decidido no backlog) / só flags | seguir o modelo de 4 estados; a tela só lê | Retrabalho do diário |
| **Consequência de derrota** | nada / perder dinheiro / voltar a um ponto | perder pouco dinheiro e voltar ao Centro, com a frase | Afeta o desafio |
| **Chat e emotes** | não fazer / emotes fixos / chat livre | emotes fixos; chat livre só com moderação | Segurança e custo de moderação |
| **Mapa do mundo** | mapa por região / minimapa / nenhum | mapa por região aberto por tecla | Só de produto |

---

## 8. Checklist de aceite (toda tela)

- [ ] Abre e fecha só por `Submit`/`Cancel`/`AlternarMenu`; **nenhuma ação exige mouse**.
- [ ] Estado do jogo devolvido ao fechar (mesmo saindo por Escape no meio).
- [ ] Inscrição e desinscrição de eventos simétricas; `OnDisable` seguro depois de o sistema morrer.
- [ ] Sem `Find` por quadro; sem alocação por quadro; lista longa em blocos.
- [ ] Toda recusa do sistema vira **frase na tela** (`UltimaMensagem`, `UltimoErro`, `motivo`).
- [ ] Status e tipos com **texto além da cor**.
- [ ] Imagem de espécie por caminho em `Resources`, liberada ao sair.
- [ ] Testada em PlayMode **pelo lado do jogador**; teste de teclado declara a dependência de foco.
- [ ] Abertura medida numa build, com o recorte (cena, máquina, data).
- [ ] `git diff` conferido: nada de asset que a Unity regravou sem intenção.

---

## 9. Referências

- [`MENUS_UGUI.md`](MENUS_UGUI.md) — como montar e ligar os painéis (uGUI) na cena.
- [`COMO_TRABALHAR.md`](../Projeto/COMO_TRABALHAR.md) — portas únicas por assunto, git, testes, desempenho.
- [`PENDENCIAS.md`](../Projeto/PENDENCIAS.md) — A4 (duas UIs), §7 (o que falta de conteúdo e sistemas), §6 (desempenho). [`DECISOES.md`](../Projeto/DECISOES.md) — P1 e as demais.
- `Assets/Omega/Game/UI/*.uxml` e `*.uss` — as telas em UI Toolkit; `Assets/Omega/Game/Runtime/UI/` — o código delas.
- `Assets/Scripts/Systems/SistemaBatalha*.cs` — a API de ação e os eventos que a `BatalhaUI` consome.
