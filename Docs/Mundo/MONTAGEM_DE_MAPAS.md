# Montagem de mapas e conteúdo — receitas, peças e lacunas

| | |
|---|---|
| **O que este documento responde** | Como montar à mão um mapa, uma casa, uma rota, um treinador, uma loja ou um trecho de história — e até onde isso vai |
| **Quando abrir** | Antes de montar qualquer coisa no mundo. Comece pela §1 (resposta curta) e pela §4 (receitas) |
| **Procura um prefab?** | [`CATALOGO_DE_PREFABS.md`](CATALOGO_DE_PREFABS.md): índice de todos, por tipo, e a cena-galeria para ver todos lado a lado |
| **Nunca montou um mapa?** | Comece por [`PRIMEIRO_MAPA_PASSO_A_PASSO.md`](PRIMEIRO_MAPA_PASSO_A_PASSO.md) (13/10): chão, porta, NPC e mato, com cliques. Este documento assume que você já sabe o que é Inspector e trigger |
| **Fora do escopo** | Modelos 3D e texturas do mapa; desempenho (ver [`MAPAS_E_DESEMPENHO.md`](MAPAS_E_DESEMPENHO.md)) |
| **Conferido em** | 07/10/2026, lendo o código e os assets. As receitas **não foram executadas de novo** para este documento: onde algo não foi conferido, está escrito |

Os mapas deste projeto são **provisórios**: o que vale é o que se aprende montando. Prefira o mínimo que resolve e registre o aprendizado (§9).

---

## 1. Dá para montar qualquer mapa à mão? Resposta curta

**Dá para montar, sem escrever código, um mapa de exploração comum**: terreno com portas entre cenas, mato com encontros, NPCs que falam,
treinadores, lojas, curandeiro, PC, itens no chão, placas, história por roteiro, e os obstáculos de Cortar, Força e Quebra-Pedra.
As receitas estão na §4.

**Não dá, ainda, para montar** (faltam peças — lista na §7): barranco, Pokémon andando no mapa,
música ou clima por zona, Flash/Voar/Cachoeira no campo e Surf de verdade.

**Duas coisas pedem código de Editor** (não há menu de "criar peça"): montar muitas peças de uma vez (o exemplo é
`OmegaKantoMulticenaNpcs`, §8) e **dividir um mapa grande em áreas** (a ferramenta `OmegaKantoMulticena` divide um mapa que já existe;
não cria um do zero). Criar uma área nova **à mão** é possível (receita R11).

---

## 2. Vocabulário

| Termo | Significa |
|---|---|
| **Bootstrap** | A cena inicial do jogo. É a **dona** do jogador, da câmera, da UI e dos sistemas. As cenas de mapa **não** trazem nada disso |
| **Cena de jogo** | Qualquer cena de mapa (rua, casa, rota). Contém só o conteúdo dela |
| **Multicena** | O mapa dividido em **uma cena base** (sempre carregada) e **uma cena por área** (carregada por proximidade) |
| **Área** | Uma dessas cenas pequenas, com um retângulo no mundo |
| **Porta** | Qualquer `PontoTransicao`: porta, escada, saída. Guarda o destino |
| **Flag** | Um texto guardado no save ("treinador_vencido_x"). Roteiros e NPCs leem e escrevem flags |
| **Roteiro** | Um asset `RoteiroData`: lista de passos (falar, dar item, batalhar, mover NPC…). É a história |
| **Tabela de encontros** | Asset `TabelaEncontros`: quais Pokémon aparecem, com que peso e nível |
| **MO** | O mesmo que HM: Corte, Surfar, Força etc. |
| **Tag** | A etiqueta do GameObject. O jogo **não usa mais** as tags `Interagivel` e `Grama` (removidas em 14/10/2026): quem é interagível é quem tem o componente |

---

## 3. Como o jogo está montado (leia uma vez)

1. **Quem manda é a cena `Bootstrap`.** Jogador, câmera, UI e sistemas nascem nela uma única vez e atravessam as trocas de cena
   (`Inicializacao`, `ObjetoPersistente`). Uma cena de mapa **não deve** ter cópia do jogador.
2. **Dar Play direto numa cena de mapa funciona**: um gancho de Editor (`MontarAoDarPlayNumaCenaDeJogo`) carrega o Bootstrap por cima, de forma aditiva. Use isso para testar (R1).
3. **Quem troca de cena é o `SistemaTransicao`.** Ninguém chama `SceneManager.LoadScene` direto. A **porta** diz para onde
   vai e onde o jogador aparece; não existe "ponto de entrada" esperando na cena de destino.
4. **O jogador interage com trigger.** Todo objeto interagível precisa de um `Collider` com **Is Trigger** ligado e de um componente que seja
   um `Interagivel` (`Npc`, `ItemColetavel`, `PontoTransicao`…), no próprio objeto ou num pai. Sem o gatilho o objeto simplesmente não responde, sem erro.
5. **Dado é asset, comportamento é componente.** O mapa só aponta para assets de `Assets/Data` (tabelas, treinadores, lojas,
   roteiros). Asset novo precisa estar no `CatalogoJogo` e ter caminho até o jogador (§5).
6. **Norte do mundo é −Z.** Pallet fica perto de x 20, z 100; a Rota 1 segue para −Z até Viridian (z ≈ −26).

---

## 4. Receitas

Cada receita diz: **o que fazer**, **o que conferir** e **o erro típico**.

### R1. Testar uma cena

1. Abra a cena e dê Play. O Bootstrap entra sozinho.
2. Se a cena não tem `SpawnDaCena`, o jogo cria um automático (no centro do maior chão). Para escolher onde o jogador nasce, ponha um
   `SpawnDaCena` e posicione. Ele cola o jogador no chão com um raio de cima para baixo, então o ponto não precisa estar rente ao chão.
3. Para a multicena, abra a **base e a área juntas** (aditivo): sozinha, a área não tem chão e o raio não acha nada.

**Erro típico:** o jogador cai no limbo. Causa: cena sem chão com colisor, ou o ponto de nascer fora dele.

### R2. Ligar dois lugares com uma porta (e o caminho de volta)

Uma porta é um `PontoTransicao` em um objeto com `Collider` trigger. Campos:

| Campo | Para quê |
|---|---|
| `cenaDestino` | Nome da cena (como no Build Settings). **Vazio = outro ponto da mesma cena** |
| `posicaoDestino` | Onde o jogador aparece, em coordenadas do **mundo** da cena de destino |
| `anguloDestino` / `aplicarAngulo` | Para onde ele fica virado ao chegar |
| `aoEncostar` | Ligado: atravessa ao pisar. Desligado: só ao apertar interagir (aí o aviso "Entrar" aparece na tela) |

**Passos**
1. Na cena A, crie a porta de ida: aponte `cenaDestino` para B e `posicaoDestino` para o **ponto de dentro de B**, perto da porta de B (nunca em cima dela).
2. Na cena B, crie a porta de volta: `cenaDestino` = A e `posicaoDestino` = **o ponto de fora da porta de A**.
3. Selecione a porta: a **esfera laranja** do Gizmo mostra onde o jogador vai aparecer. Confirme que está no chão.
4. As duas cenas precisam estar no **Build Settings** (hoje são 23 entradas).

**O que conferir:** entrar e sair devolve o jogador no mesmo lugar. O jogo desarma a porta enquanto o jogador está dentro do trigger de chegada
(`esperandoSaida`); se o ponto de chegada estiver **dentro** de outro trigger de porta, ele só dispara depois que o jogador sair e voltar.

**Erros típicos:** (a) a porta de volta aponta para a porta da rua em vez do lugar de fora da porta de A: o jogador sai na porta errada;
(b) cena fora do Build Settings: o Unity dá erro de cena não encontrada e a troca não acontece; (c) destino dentro do collider de outra porta.
(Não conferi qual vence quando a cena de destino também tem um `SpawnDaCena`: ponha o spawn no mesmo lugar da porta de entrada, para os dois concordarem.)

$1
- **Origem do encontro (`origem`):** diz de onde vem o encontro para as Pokébolas especiais. Padrão **Mato**; ponha **Água** numa zona de água (Dive Ball) e **Caverna** numa caverna (Dusk Ball). A pesca já é Pesca sozinha.
$2 NPC que fala, placa e item no chão

- **NPC de falas:** o **corpo** vem de `Assets/Prefabs/Personagens/` (gerado pelo `MontadorDeNpc`, menu `Omega → Personagens NPC → Gerar todos`).
  O componente de interação (`Npc`) vai em um **filho trigger chamado `Interacao`**; o colisor sólido fica na raiz.
  Preencha `falas`; `falasRepetidas` e `flagDeProgresso` fazem o NPC falar outra coisa depois da primeira vez.
- **Placa:** `PlacaComRoteiro` com um roteiro (R8), num objeto com gatilho.
- **Item no chão:** `ItemColetavel` com `item`, `quantidade` e um `idNoMundo` **único** (é o que o save usa para saber que já foi pego).
  Para um **item escondido**, ligue `escondido`: o modelo some e o aviso vira "Examinar" (sem o ícone, que entregaria o que é). O colisor com gatilho continua obrigatório; o jogador só acha andando perto. Não existe o Itemfinder.

**Erro típico:** copiar um `ItemColetavel` e esquecer de trocar o `idNoMundo`: pegar um apaga o outro.

### R5. Treinador

1. Crie o asset **`Pokemon → Treinador`** (`TreinadorData`): nome, classe, time (espécie, nível, golpes, item), `recompensa`,
   perfil de IA, `dupla`, e opções de time por condição (por exemplo "depois da insígnia 2").
2. No mapa, ponha o corpo (R4) e, no filho `Interacao`, o `NpcTreinador` com o `treinador` apontado. Com ele preenchido, a equipe
   inline do componente é ignorada.
3. **Flag de vitória:** o componente gera uma flag única sozinho (`treinador_vencido_<guid>`). Não troque por um nome repetido: dois treinadores
   com a mesma flag viram um só.
4. A **política de revanche** (uma vez, renovável, sem limite) liga a revanche; recompensas de item têm "apenas uma vez" por item. Líder de ginásio e campeão levam `ehMarco` no `TreinadorData`.
5. **Para o treinador enxergar e vir até o jogador**, acrescente `VisaoDeTreinador`.

**O que conferir:** `Pokemon → Validar assets de dados` e `Pokemon → Auditar alcancabilidade do conteudo` (§5).

### R6. Loja, curandeiro e PC

| Peça | Como |
|---|---|
| **Loja** | Asset **`Pokemon → Loja`** (`LojaData`) com a lista de itens. **Item de preço 0 nunca é vendido** (o asset avisa ao salvar). No mapa: `Vendedor` com a `loja` apontada e a referência `menu` preenchida |
| **Curandeiro** | `Curandeiro`. Aceita `falasAoCurar` e `falasSemPrecisar` |
| **PC** | `PontoDeArmazenamento` (precisa de um `Menu` com o `CaixaMenu` na cena; senão mostra a mensagem de erro do campo `mensagemSemMenu`) |
| **Tutor de golpe** | `NpcEnsinaGolpe`; os 14 professores prontos são prefabs em `Assets/Data/Mundo/Professores` |
| **Troca de Pokémon** | `NpcTrocador` |
| **Evolução por local** | `PontoDeEvolucao` |

Todos seguem R4: corpo + filho `Interacao`. As referências a `Menu` e a sistemas (`dialogo`, `inventario`, `dados`) estão no
Inspector; **confira que ficaram preenchidas** depois de colar o componente.

### R7. Mato de verdade vs. enfeite

O visual do mato (blocos `Mato_*`) é só malha. **Quem gera encontro é a `AreaDeEncontro` (R3)**; um mato sem ela é decoração.
O Pallet foi montado assim: o script mede a caixa dos blocos e põe a zona por cima.

### R8. História: roteiro, NPC dirigido e gatilho

1. Crie **`Pokemon → Roteiro de Evento`** (`RoteiroData`) e monte os passos. Há **22 tipos**: fala, condição (`PassoSe`), ligar/desligar flag,
   dar item/Pokémon/dinheiro/insígnia, tirar item, batalha, batalha selvagem, curar equipe, mover NPC e jogador, posicionar, virar,
   forçar evolução, alterar e tirar Pokémon, definir variável (10/10, ver [`../Jogo/INTERACAO_POR_CARDS.md`](../Jogo/INTERACAO_POR_CARDS.md) §8), mostrar/esconder NPC, esperar, teleportar, **escolha**, chamar outro roteiro, definir nome.
2. **Quem dispara:**

| Disparo | Peça |
|---|---|
| Falar com um NPC | `NpcComRoteiro` (`roteiro`, e opcionalmente `flagLida` + `roteiroSeFlag` para uma segunda versão depois da flag) |
| Ler uma placa | `PlacaComRoteiro` |
| Pisar numa área | `GatilhoDeArea` (trigger; `flagDeUmaVezEscrita` para rodar uma vez por save) |
| Entrar na cena | `GatilhoAoEntrarNaCena` |
| Scripts do mapa | `ScriptDeMapa` |

3. Um NPC movido por roteiro precisa de `AtorDeRoteiro`.
4. Terminou? **`Pokemon → Validar roteiros`**: ele acusa passo vazio, batalha sem treinador, escolha sem opção e, o mais traiçoeiro,
   **flag lida que ninguém escreve** (um erro de digitação faz o "Se" cair no senão para sempre, sem log).

**Limite conhecido:** a escolha e a pergunta de texto aparecem num prompt provisório (IMGUI). Funciona, mas é feio e não tem teclado
(ver [`UI_DO_PROJETO.md`](../Jogo/UI_DO_PROJETO.md) §3.2.4).

### R9. Obstáculo de MO (Cortar, Força, Quebra-Pedra)

1. Colocar `ArvoreCortavel`, `PedraDeForca` ou `RochaQuebravel`. O **colisor sólido fica na raiz**; um **filho trigger**
   é o que o jogador encontra.
2. Por padrão exige as três coisas do FireRed (§6). Para mudar, preencha `itemExigido`, `golpeExigido`, `insigniaExigida`; vazio = padrão.
3. A insígnia só é dada pelo passo `PassoDarInsignia` de um roteiro. **Sem um roteiro que dê a insígnia, o obstáculo nunca abre.**

### R10. Pesca

`PontoDePesca`: trigger na beira d'água. Usa a melhor vara da mochila (`OldRod`, `GoodRod`, `SuperRod`), pega a tabela
**daquela vara**, e sorteia a mordida (`chanceDeFisgar`, padrão 0,75). Sem vara, sem tabela ou sem mordida o jogador ouve a fala correspondente.

### R12. Gelo e esteira

`SuperficieDeCampo` num objeto **com colisor** (um cubo de chão, ou um gatilho fino logo acima do piso). Não precisa de gatilho: o jogador acha a superfície com um raio para baixo saindo dos pés.

- **Gelo:** `aceleracaoNoGelo` (menor = mais escorregadio) e `atritoSemComando` (menor = desliza mais longe). Sem tecla o jogador continua deslizando; ao sair do gelo volta ao controle normal na hora.
- **Esteira:** `velocidadeDaEsteira` somada ao movimento, na `direcaoDaEsteira` (vazio = a frente do objeto). Selecione o objeto: uma seta azul no Scene mostra a direção.
- Só vale com o jogador no chão e podendo se mover (conversa e menu seguram o corpo). Pular sobre a faixa não sofre efeito.

**Erro típico:** colocar o componente num objeto sem colisor: nada acontece e não há erro.

**Ainda não existe:** a sensação de "até bater numa parede" do gelo do cartucho (aqui o atrito para o jogador sozinho) e o piso giratório.

### R11. Criar uma área nova na multicena, à mão

1. Crie uma cena em `Assets/Scenes/Mundo/KantoMulticena/Areas/` com o conteúdo daquele pedaço (casas, rota, NPCs, mato).
2. Abra a **base** (`KantoBase.unity`), selecione o objeto com `CarregadorDeAreas` e acrescente à lista `areas`:
   `cena` = caminho (`Assets/…/*.unity`) e `retangulo` = o retângulo **XZ** que a cena ocupa no mundo.
3. Ponha a cena **e** a base no Build Settings.
4. Em jogo, a área carrega quando o jogador está a **250 m** da borda e descarrega a **400 m** (`carregarA`, `descarregarA`; a folga evita a
   área da divisa "piscar").
5. Rode `KantoMulticenaTests` (PlayMode) para conferir o carregamento.

**Regras:** NPCs e zonas de encontro ficam **na cena da área que cobre o ponto**, nunca na base. A base **não pode herdar dados de
occlusion culling** (fazia sumir o chão perto da Torre Pokémon, 30/09). O chão (terreno e o mar `MarBase`) mora na base.
Peças altas que servem de silhueta no horizonte ficam na base (menu "3. relatorio de pecas altas").

---

## 5. Antes de dar por pronto (checklist)

Nesta ordem, com o Editor aberto:

1. [ ] **Cena no Build Settings** (e, se for multicena, a base também).
2. [ ] **Play na cena**: o jogador nasce no chão, cada porta vai e volta ao lugar certo, cada gatilho responde.
3. [ ] **Todo asset novo está no `CatalogoJogo`.** Asset fora do catálogo **grava no save e some ao carregar, em silêncio**. Menus:
   `Pokemon → Popular catalogo selecionado com todos os assets` (com o catálogo selecionado) e `Omega → Itens - registrar no catalogo`.
4. [ ] **`Pokemon → Validar assets de dados`** (confere ids e referências; roda também como teste).
5. [ ] **`Pokemon → Validar roteiros`**.
6. [ ] **`Pokemon → Auditar alcancabilidade do conteudo`**: acusa asset que **ninguém aponta** (loja que nenhum NPC usa, golpe que ninguém ensina).
   Ele prova que existe um caminho declarado, **não** que o jogador consegue percorrê-lo: isso só se confere jogando.
7. [ ] **Não commite o que a Unity regravou sem querer** (materiais, cenas): confira o `git diff`. **Não commite `Kanto3D.unity` nem `KantoPecas.unity`**
   (ver [`ESTADO_DO_PROJETO.md`](../Projeto/ESTADO_DO_PROJETO.md) §7).
8. [ ] Resolveu algo do backlog: apague a linha de [`PENDENCIAS.md`](../Projeto/PENDENCIAS.md) e explique em [`PENDENCIAS_FECHADAS.md`](../Historico/PENDENCIAS_FECHADAS.md) **no mesmo commit** ([`COMO_TRABALHAR.md`](../Projeto/COMO_TRABALHAR.md) §8).

---

## 6. Referência: Surf, MOs e as regras de campo

### Obstáculos de campo

`ObstaculoDeCampo` é a base de `ArvoreCortavel`, `RochaQuebravel` e `PedraDeForca`. Ao interagir, `RegrasDeCampo.Conferir` exige **as três coisas**:
a **MO** na mochila (id padrão `Hm01` etc.), a **insígnia** que libera o golpe fora de batalha e **um Pokémon da equipe que conhece o golpe**.

| Golpe | MO | Insígnia | Obstáculo no jogo |
|---|---|---:|---|
| Cortar | `Hm01` | 2 | `ArvoreCortavel` |
| Voar | `Hm02` | 3 | — (só a regra) |
| Surfar | `Hm03` | 5 | provisório (abaixo) |
| Força | `Hm04` | 4 | `PedraDeForca` |
| Flash | `Hm05` | 1 | — (só a regra) |
| Quebra-Pedra | `Hm06` | 6 | `RochaQuebravel` |
| Cachoeira | `Hm07` | 7 | — (só a regra) |

Se falta algo, o jogador ouve qual (ex.: "É preciso a insígnia 2 para usar Cortar.").

### Surf: regra pronta, mecânica provisória

- *Regra* (`RegrasDeCampo.PodeSurfar`): MO03 (`Hm03`) na mochila **e** insígnia 5 **e** um Pokémon da equipe com Surf. As flags antigas `HM_SURF`/`TM_SURF` também liberam.
- *Mar* (`MarSemLimbo`): a água não tem colisor. Um **piso invisível** fica 1 m abaixo do nível do mar (dá para entrar até a cintura). Sem Surf, onde o fundo
  passa de 1,4 m o jogador volta ao último ponto seguro. Com Surf, o piso sobe para a superfície. O próprio código chama isso de **placeholder**.
- *Falta:* o estado "montado" (animação, modelo, trava de movimento), entrar e sair da água pela margem, e UI.

---

## 7. O que ainda não dá para montar

**Peças de mapa**

| Falta | Observação |
|---|---|
| Barranco (pular de um degrau, só em um sentido) | O jogador tem pulo livre (`PlayerMovement`), mas não existe a peça |
| Piso giratório | Gelo e esteira existem (R12); o giratório não |
| Itemfinder | O item escondido existe (R4, `escondido`), mas não há como detectá-lo |
| Pokémon visível andando no mapa | Não existe; encontros são só mato, água e pesca |
| Música por zona | `SistemaAudio` só toca exploração, batalha e vitória, globais; e há **zero arquivos de áudio** |
| Clima por zona | O clima é global (`SistemaMundo`). Falta "chove só nesta rota" |
| Flash, Voar, Cachoeira no campo | Estão definidos em `RegrasDeCampo`, mas nenhum obstáculo ou comportamento os usa |
| Surf de verdade (estado montado) | §6 |

**Sistemas de jogo:** missões, relembrar golpes, creche e ovos, ginásio como sistema (hoje só o passo `PassoDarInsignia`; o campo `badges` não é alimentado de outro jeito).

**Ferramentas:** não há menu "criar peça" para zona de mato, porta ou placa, nem fluxo de "criar mapa multicena do zero". As receitas da §4 são à mão no Inspector.

---

## 8. Referência: o que existe

### Peças de mapa (`Assets/Scripts/World`)

| Peça | Para quê |
|---|---|
| `PontoTransicao` | Porta, escada, saída de área (R2) |
| `SpawnDaCena` | Onde o jogador nasce (R1) |
| `AreaDeEncontro` | Mato, caverna, água (R3) |
| `PontoDePesca` | Beira d'água (R10) |
| `ItemColetavel` | Item no chão, visível ou escondido (R4) |
| `SuperficieDeCampo` | Gelo e esteira (R12) |
| `ArvoreCortavel`, `RochaQuebravel`, `PedraDeForca` | Obstáculos de MO (R9) |
| `Curandeiro`, `PontoDeArmazenamento`, `PontoDeEvolucao`, `Vendedor` | Centro Pokémon, PC, local de evolução, loja (R6) |
| `ObjetoPersistente` | Estado de objeto que sobrevive à troca de cena |

### NPCs

`Npc` (falas), `NpcTreinador` (batalha; `VisaoDeTreinador` para quem enxerga e vem), `NpcEnsinaGolpe` (professor de golpe), `NpcTrocador` (troca),
`NpcComRoteiro` e `AtorDeRoteiro` (qualquer NPC dirigido por roteiro), `PlacaComRoteiro`. O corpo vem de `Assets/Prefabs/Personagens/`.

### Ferramentas de Editor (menus)

| Menu | Para quê |
|---|---|
| `Omega → Personagens NPC → Gerar todos` (e "+ prévia PNG") | Gera os corpos de NPC |
| `Omega → Kanto multicena - 1. relatorio das areas` / `2. separar em cenas por area` / `3. relatorio de pecas altas` | Divide um mapa grande já pronto em áreas |
| `Omega → Kanto multicena - 4. tutores, vendedor, treinador, spawn e mato em Pallet` | **Exemplo de montagem por script**: reaplicável, coloca NPCs, spawn e a zona de mato em Pallet |
| `Omega → Kanto multicena - 5. historia de Pallet (fatia migrada da Kanto3D)` | Migra a história de Pallet para a multicena |
| `Omega → Historia Pallet (roteiros + NPCs)`, `Importar historia FireRed (...)` (os menus não têm acento) | Gera roteiros, NPCs, placas e encontros da história |
| `Omega → Kanto - extrair pecas e montar cena nova`, `reaplicar regras nos prefabs`, `simplificar colisores dos predios` | Montagem do mapa em prefabs (`KantoPecas`) |
| `Pokemon → Validar assets de dados`, `Validar roteiros`, `Auditar alcancabilidade do conteudo`, `Popular catalogo…` | Conferência (§5) |

### Conteúdo (assets em `Assets/Data`)

Espécies, golpes, habilidades, itens, held items, lojas, treinadores, tabelas de encontro, roteiros, status, climas, terrenos, perfis de IA,
professores. Números em [`ESTADO_DO_PROJETO.md`](../Projeto/ESTADO_DO_PROJETO.md) §2.4.

---

## 9. Registrando o que se aprende

Mapas são provisórios. Ao mexer em um, anote aqui ou em [`MAPAS_E_DESEMPENHO.md`](MAPAS_E_DESEMPENHO.md) o que descobriu (armadilhas de chão,
de ferramenta, de medição). Exemplo de 30/09: o menu do Unity CLI estoura 30 s (o Editor segue rodando; espere pelo arquivo de relatório que a
ferramenta escreve em `Temp/`).

---

## Apêndice. Exemplo de ponta a ponta: a casa do rival em Pallet

Tarefa: uma casa em Pallet com um rival que fala, uma loja, e uma batalha ao ser desafiado.

1. **Cena** `CasaDoRival.unity` com um piso de colisor e um `SpawnDaCena` perto da porta. Adicione-a ao **Build Settings**.
2. **R2:** na rua de Pallet, uma porta na frente da casa: `cenaDestino = CasaDoRival`, `posicaoDestino` dentro da sala, perto da porta. Na casa, a porta de saída:
   `cenaDestino` = a cena de Pallet, `posicaoDestino` = o ponto na rua em frente à porta. Cheque a esfera laranja nas duas.
3. **R5:** asset `Pokemon → Treinador` "Rival" com o time. No corpo `Rival` (de `Assets/Prefabs/Personagens/`), o filho `Interacao` com `NpcTreinador`.
4. **R8:** um roteiro "Rival_Apresentacao" (fala, batalha, fala, ligar flag `rival_derrotado`) e um `NpcComRoteiro` com `flagLida = rival_derrotado` e o
   `roteiroSeFlag` com a fala pós-vitória. Rode **Validar roteiros**.
5. **R6:** `Pokemon → Loja` com os itens de preço > 0, e um `Vendedor` com a loja e o `Menu` preenchidos.
6. **§5:** catálogo, validador, auditor, Play na cena (entrar, falar, batalhar, comprar, sair).
7. **Commit** com o `.meta` de cada arquivo, sem as cenas gigantes.
