# Como trabalhar no Nel-Ord

| | |
|---|---|
| **O que este documento responde** | Como abrir, rodar, testar e mexer no projeto sem repetir os tropeços que já custaram dias. Cada regra tem o incidente que a motivou entre parênteses |
| **Quando abrir** | No primeiro dia (§0), e antes de mexer em git, conteúdo, save, testes ou merge |
| **O que ele NÃO é** | A situação do projeto ([`ESTADO_DO_PROJETO.md`](ESTADO_DO_PROJETO.md)), o que falta ([`PENDENCIAS.md`](PENDENCIAS.md)) nem o que cada sistema faz ([`../Jogo/REFERENCIA_DE_SISTEMAS.md`](../Jogo/REFERENCIA_DE_SISTEMAS.md)) |
| **Atualizado em** | 09/10/2026 (§0 e §6 reescritos; §9 conferida em 09/10; o resto, 26/09 e 07/10) |

> **A UI é a maior trava do projeto hoje** — ver o aviso em [`ESTADO_DO_PROJETO.md`](ESTADO_DO_PROJETO.md) e a decisão P1 em [`DECISOES.md`](DECISOES.md).

---

## 0. Primeiro dia: abrir, jogar, testar

**O que instalar.** Unity Hub com o editor **`6000.5.10f1`** (Unity 6, URP); Git e **Git LFS** (`git lfs install` uma vez por máquina: `.glb`, `.fbx`, `.png`, `.jpg`, `.tga`, `.wav`, `.ogg` vão pelo LFS). Referência de máquina: Ryzen 5 4600G, 16 GB, projeto num SSD (no HD a `Kanto3D` carregou em 62 a 159 s).

**Baixar.** `git clone https://github.com/Mhgfxv00/Nel-Ord.git` e `git checkout mapa/blocos` — **a branch de trabalho é `mapa/blocos`**; a `main` está 197 commits atrás e em outra versão do Unity (ver [`PENDENCIAS.md`](PENDENCIAS.md) C5). Na primeira abertura a Unity importa tudo e gera `Library/` (demora; não vai para o git).

**Abrir.** Unity Hub → *Add project from disk* → a pasta do clone. Se a Unity pedir para atualizar a versão do projeto, **recuse** e instale a `6000.5.10f1`.

**Jogar.**

| Quero | Faça |
|---|---|
| O caminho do jogo | Abra `Assets/Scenes/Jogo/Bootstrap.unity` e dê Play. Ela cria jogador, câmera, UI e sistemas e leva à `OmegaMenu` (título e criador de personagem, em UI Toolkit). Para cair direto no mapa, dê Play na `DevScene` ou numa cena de Kanto (linha abaixo) |
| Testar uma cena de mapa | Abra a cena e dê Play: um gancho de Editor carrega o Bootstrap por cima (aditivo). Detalhe: [`../Mundo/MONTAGEM_DE_MAPAS.md`](../Mundo/MONTAGEM_DE_MAPAS.md) R1 |
| Pular o progresso | O componente `ModoDesenvolvedor` (Omega) enche a mochila com 99 de cada item do catálogo e libera a Pokédex inteira; desmarque `ligado` para jogar normal |
| Testar o mapa de Kanto | `Assets/Scenes/Mundo/KantoMulticena/` (abra a `KantoBase` **e** uma área, aditivo). **Não abra `Kanto3D` nem `KantoPecas` sem necessidade**: 89 e 74 MB, derrubam o Editor com 16 GB |

**Controles** (`Assets/Scripts/Inputs.inputactions`): mover **WASD** ou analógico esquerdo · interagir **E** · pular **Espaço** · correr **Ctrl** · espreitar **Shift** · próximo interagível **Tab** · dançar **G** · câmera: mouse, zoom na rodinha · menu: **Esc** (ou Start). Menus: setas/WASD, **Enter** confirma.

**Testar.** `Window → General → Test Runner`, abas **EditMode** e **PlayMode**, *Run All*. Detalhes e a rodada por linha de comando: §6.

**Achar as coisas.** Dois menus de ferramentas no Editor: **`Pokemon/…`** (validadores, auditor, geradores de conteúdo do Showdown, catálogo, IA) e **`Omega/…`** (mapa, arte, importadores, UI do outro lado do grupo). Catálogo de todos: [`FERRAMENTAS_DO_EDITOR.md`](FERRAMENTAS_DO_EDITOR.md). **Três delas conferem o seu trabalho** e valem rodar antes de commitar conteúdo: `Pokemon → Validar assets de dados`, `Pokemon → Validar roteiros`, `Pokemon → Auditar alcancabilidade do conteudo`.

**Ler a documentação.** [`../README.md`](../README.md) diz qual documento responde cada pergunta.

---

## 1. Quem cuida de quê

| Frente | Dono | Onde mora |
|---|---|---|
| Batalha, IA, regras, save, dados de Pokémon | Higor (esta máquina) | `Assets/Scripts/Systems`, `Data`, `MesaDeRegras`, `SistemaSave` |
| Mapa, visual, texturas, modelagem | klz + grupo | `Assets/Omega/Arte`, `Omega/Visual`, cenas `Kanto*` |
| UI (uGUI × UI Toolkit, telas) | outro lado do grupo | `Assets/Scripts/UI`, `Omega/Game/Runtime/UI` |
| Conteúdo de dados (golpes, espécies, treinadores) | quem precisar, pelas ferramentas | `Assets/Data` + geradores em `Omega/Editor` |

Mexer na frente de outro: um commit separado, com teste, e a pergunta escrita na mensagem.

A UI inteira que o projeto precisa (telas, contratos com os sistemas, o que falta e em que ordem) está em
[`UI_DO_PROJETO.md`](../Jogo/UI_DO_PROJETO.md).

## 2. Antes de escrever código: que sistema já decide isso?

O projeto tem **porta única por assunto**. Código novo pergunta a ela, nunca reimplementa:

| Assunto | Porta |
|---|---|
| Regra de batalha (dano, precisão, prioridade, status, clima, terreno) | `MesaDeRegras` |
| Estado do jogo (explorando, diálogo, batalha, menu) | `GerenciadorEstados` |
| Input | Action Map própria + `EscutaDeInput` (nunca guardar `entrada.Mapa`) |
| Achar asset por id / save | `CatalogoJogo` |
| Pokémon ↔ save/rede | `TradutorDePokemon` |
| Dado de fora do Runtime no save | `IParticipanteDoSave` |
| Sorteio | `IAleatorio` da batalha; o global só na criação, declarado em `AcasoGlobalTests` |

(O atalho de Pokédex na tecla E reaproveitou `Interagir` e não consultou o `GerenciadorEstados`:
abria a Pokédex no meio da batalha. Há 4 pares de sistemas duplicados — UI, movimento, pipeline de
dados, validador — todos por não conferir o que já existia.)

## 3. Git

- **Um tema por commit**, mensagem com o porquê. `.meta` sempre junto do arquivo.
- **Não commitar `Kanto3D.unity` nem `KantoPecas.unity`** até decidir: dividir o mapa em cenas por
  área (recomendado) ou LFS. As duas já estão no `origin` sem LFS (89 e 74 MB em 09/10; o GitHub recusa
  acima de 100 MB, e cada commit delas soma ~80 MB ao histórico para sempre).
- **LFS é para binário** (`.glb`, `.png`, `.fbx`, áudio — já configurado). Cena é texto: no LFS
  perde diff e merge.
- **Unity 6000.5.10f1** em todas as máquinas (a `main` ainda está em 9f1).
- Terreno binário (`TerrainData`) é `-text` no `.gitattributes`. (o eol corrompeu o `New Terrain.asset`
  e derrubou 58 testes)
- Unity regrava material/asset ao abrir cena (render type, campos novos de esquema): **não
  commitar o que você não mudou de propósito**; confira o `git diff` antes.
- Antes de afirmar algo sobre o remoto, `git log @{u}..HEAD`. (o backlog dizia "sem push" de algo
  que já estava no `origin`)

## 4. Conteúdo como dado

- **Comportamento é peça tipada** (`RegraDeBatalha` consultada, `EfeitoDeBatalha` disparado,
  condições `[SerializeReference]`); o asset só combina peças. Mecânica nova = peça nova, não `if`.
- **Criar asset por script**: criar vazio, salvar, recarregar pelo caminho, preencher em C#.
  Nunca crescer lista por `arraySize++` (nasce tudo zerado: 4 gatilhos com `hpNoMaximo: 0` nunca
  dispararam por três semanas).
- Script de conteúdo **idempotente**: logo após compilar, o CLI dá timeout mas o comando roda.
  A verdade se confere **no arquivo em disco**.
- Todo asset com **id explícito**, no `CatalogoJogo`, e com caminho até o jogador (senão entra no
  `AuditorDeAlcancabilidade` com motivo escrito).
- Dado externo por espécie: casar pelo prefixo `pNNNN_` do arquivo. **Os ids 10xxx do projeto não
  são os da PokeAPI.** Conferir por amostra, nome a nome. (256 espécies pegaram razão de gênero errada)
- **Nada que o catálogo alcança aponta direto para textura grande.** O `CatalogoJogo` vive no
  bootstrap, e a Unity carrega junto tudo que um asset referencia: imagem de espécie mora em
  `Resources` e a espécie guarda o caminho (`SpritesDoCatalogo`). (as imagens de espécie ocupavam
  985 MB de textura antes da tela de título; depois, 77 MB — medido numa build)
- Antes de dar comportamento a algo, procurar quem já dá. (o bônus de Água na chuva já vivia na
  tabela do `SistemaMundo`; um `ClimaData` igual dobraria o dano)


### 4.1 Geradores congelados (desde 07/10/2026)

Os **geradores de dados** (menus `Omega/Dados/*`, `Omega/Importar historia FireRed`, `Omega/Historia Pallet`) criaram espécies,
golpes, habilidades, itens e a história a partir do APK, da PokeAPI e do FireRed. Depois disso o conteúdo foi **corrigido à mão**:
226 espécies, 255 itens, 19 habilidades e 3 golpes de fã ou lixo foram **removidos**, e ~100 itens e vários golpes e habilidades
ganharam comportamento. Rodar um gerador de novo pode desfazer isso. Por isso:

- **Todo menu gerador começa com `GeradoresCongelados.Permitir(...)`** e, congelado, recusa e explica. No batch, recusar encerra a
  Unity com erro (o lote não fica pendurado).
- **Para rodar mesmo assim, de propósito:** `Pokemon → Geradores → Descongelar nesta sessão` (vale até fechar a Unity) ou, no batch,
  `NELORD_DESCONGELAR_GERADORES=1`. Depois, conferir o `git diff` asset por asset.
- **O conteúdo removido está listado** em `Assets/Omega/Dados/conteudo_removido.txt`. O teste `GeradoresEFerramentasTests` falha
  se qualquer um voltar a existir em `Assets/Data` (por gerador ou por qualquer outro caminho) e se aparecer menu gerador sem a trava.
  Trazer um de volta de propósito = tirar a linha da lista no mesmo commit, dizendo por quê.
- **O gerador original das espécies** (o extrator do APK, `tools/omega-extract`) **não está neste repositório nem nesta máquina**.
  Se reaparecer, ele precisa ler a lista de removidos antes de criar qualquer asset.
- **Conteúdo novo entra pelo caminho de hoje:** `ConteudoDoShowdown` (preenche comportamento, não cria espécie) e asset feito à mão.
- **Ferramentas que escrevem campo pelo nome em texto** (`Set(obj, "campo")`, `Get<T>(obj, "campo")`, `FindProperty("campo")`):
  o mesmo teste confere cada nome contra os campos que existem nas classes do jogo. **Renomeou um campo serializado? Rode o
  EditMode**: o teste aponta a ferramenta e a linha antes que ela quebre no meio, com metade dos assets gravados.

### 4.2 Fluxo de arte

A arte **não é gerada**: é importada e **ligada** a assets que já existem. Os menus de arte continuam livres (só ligam, não criam conteúdo).

| Arte | De onde vem | Como liga | Situação (07/10/2026) |
|---|---|---|---|
| Sprite de batalha da espécie | Folhas em `Resources/Batalha` | `EspecieData.folhaEmResources` + `quadroDaFolha` (menus `Omega/Sprites - *` e `Pokemon → Soltar sprites de Resources do catalogo`) | Todas as 1.326 espécies têm sprite ligado. **As formas criadas em 07/10** (Castform Sol/Chuva/Neve, Cramorant Com Presa/Com Raio, Mimikyu Descoberto) **usam a folha da base**: falta arte própria |
| Sprite da PokeAPI (alternativo) | `Assets/Omega/Editor/sprites_pokemon.json` (mapa asset → arquivo) | Menu `Omega/Especies - importar e ligar sprites da PokeAPI` | O mapa ainda cita 224 espécies removidas; é inofensivo (só liga a asset que existe) |
| Modelo 3D do Pokémon | 968 `.glb` em `Assets/Omega/Arte/Modelos/Pokemon/Gen1..9` | `CatalogoModelos3D` (número da Pokédex → modelo), menu `Omega/Modelos - montar catalogo 3D` | **O catálogo liga só 11 dos 968 modelos.** Rodar o menu e conferir; formas usam o modelo do número da Pokédex |
| Ícone de item | Atlas + `Assets/Omega/Editor/icones_item.json` | Menu `Omega/Itens - fatiar e ligar icones` → `ItemData.icone` | **2.106 dos 2.434 itens sem ícone** |
| NPC / personagem | `Assets/Omega/Parts`, `MontadorDeNpc` | Menu `Omega/Personagens NPC/Gerar todos` | — |

**Regra para arte nova:** (1) o arquivo entra na pasta da tabela acima com o **nome no padrão** dela (`NNNN_Nome` para Pokémon); (2) roda o
menu de ligação; (3) confere no asset (o campo preenchido) e no jogo; (4) nada de referência direta do `CatalogoJogo` a textura grande
(ver acima). **Forma sem arte própria usa a da base** — é aceitável como provisório, nunca como final: liste-a na `PENDENCIAS`.

## 5. Save

- Mudou o formato do `SaveJogo`/`SavePokemon`? **Sobe `VersaoDoFormato`** e escreve o degrau em
  `MigrarSeNecessario`. Build antiga recusa em vez de apagar.
- Campo novo no `Pokemon`? **Pergunte se ele atravessa o save.** (amizade, críticos e "usou X vezes"
  nunca foram gravados, com o teste de ida e volta verde)

## 6. Testes

### Como rodar

Há três caminhos. **Escolha um e confira o resultado no XML**, não no código de saída.

| Caminho | Quando | Como |
|---|---|---|
| **Test Runner** (janela do Editor) | O dia a dia | `Window → General → Test Runner`; abas *EditMode* e *PlayMode*; *Run All* ou o teste isolado |
| **Linha de comando, Editor fechado** | Rodada completa, CI, depois de mexer em batalha | `Unity.exe -batchmode -projectPath . -runTests -testPlatform EditMode -testResults resultado.xml -logFile log.txt` (e `PlayMode`). O executável fica em `<pasta do Unity>\Editor\Unity.exe`. Exige que **nenhuma outra instância** tenha o projeto aberto (`Temp/UnityLockfile`) |
| **CLI oficial `unity` com o Editor aberto** | Quando o Editor já está aberto e você não quer fechá-lo | `unity command run_tests --mode EditMode\|PlayMode --async_tests`, depois `test_status` até `completed`. `--filter` aceita **um** nome (dois dão "0 testes"). O pacote `com.unity.pipeline` sobe o servidor dentro do Editor. É opcional: tudo o que ele faz se faz pelo Test Runner |

Duas pegadinhas da linha de comando: **não passe `-quit` junto de `-runTests`** (o framework fecha sozinho; `-quit` pode encerrar antes de os testes rodarem e sair "com sucesso" sem XML); e o código de saída do `Unity.exe` não é o do script que o chama — **sempre confira `<test-run result="..." failed="...">` no XML**. Sem XML, nunca diga "passou".

### Categoria `Mapa` (desde 07/10/2026)

Os testes que carregam cena de mapa pesada (`Kanto3D`, `KantoPecas`, a multicena) — `BotDeVarreduraTests`, `DesempenhoTests`, `HistoriaFireRedPlayModeTests`, `HistoriaPalletPlayModeTests`, `KantoMulticenaTests` — ficam **fora da rodada padrão**: faziam a suíte inteira de PlayMode passar de 45 minutos e não terminar (memória). Rodada padrão do PlayMode, com o Editor **fechado**:

```
Unity.exe -batchmode -projectPath . -runTests -testPlatform PlayMode -testCategory "!Mapa" -testResults resultado.xml
```

Só os de mapa, sob demanda: `-testCategory "Mapa"`. (O `unity test` do CLI oficial não tem filtro de categoria.)

### Regras de teste

- **Só se asserta o que é igual em qualquer máquina.** Tempo é relatado, nunca exigido.
- Algo sorteado ou "estável" no commit: **bateria de 6+ rodadas**. Duas rodadas não pegam um flake de 5%.
- Pokémon de teste só por `ArnesDoJogador.PokemonDeTeste` (habilidade fixa): o que é sorteado na criação vira ruído (causou um flake de 1 em 6).
- Espere o **turno avançar**, não o estado `EscolhaAcao` (ele volta no mesmo quadro).
- Teste de PlayMode que depende de sorte da IA: dê ao selvagem **um golpe só** e deixe o jogador claramente mais forte.
- **Nenhum teste existente muda de expectativa** quando você acrescenta API nova (a `MesaDeRegras` ganha *overload*).
- Cada peça de batalha nova sai com teste de **EditMode** (regra pura) e de **PlayMode** (batalha de verdade, em dupla quando couber).

### Como testar `SistemaBatalha`

É `MonoBehaviour` + corrotina: o EditMode não o alcança. O caminho é o **PlayMode** (`Assets/Scripts/Tests/PlayMode/`), que carrega a `DevScene` e dirige a API pública (`EscolherGolpe`/`EscolherTroca`/`EscolherItem`/`EscolherFuga`) numa instância real. Dois cuidados aprendidos na marra:

1. **Apague os `ObjetoPersistente` antes de carregar a cena**, senão o teste herda a batalha do anterior (recarregar a cena não basta: os sistemas são `DontDestroyOnLoad` e a cópia nova se autodestrói).
2. **Monte a equipe do jogador explicitamente** (`SubstituirEquipe`), em vez de herdar o que a cena ou o save trouxerem.

Regra de ouro dessa suíte: **nenhuma asserção sobre número de dano, acerto ou escolha da IA** (é sorteado). Testa-se o que a máquina de estados garante: quem ocupa cada posto, quando a batalha encerra, as regras do formato. Para conferir a narração frase a frase, espelhe-a no Console: `batalha.OnMensagem += m => Debug.Log("[NARRACAO] " + m);`.

### Teclado em teste

Um teste (`SaveViaJogadorTests.EscapeAbreOMenu_PeloInputDeVerdade`) se declara `Ignore` sem a Game View em foco. O ponto cego (alguém desfazer o mapeamento do Escape) está fechado por `BindingsDeInputTests` (EditMode, roda sempre). **Não** herde de `InputTestFixture`: resolve o foco mas quebra por outro motivo (pressupõe um teste que constrói o próprio mundo de input; o nosso dirige uma cena carregada com sistemas persistentes que criam os próprios `Inputs`). Ver PENDENCIAS D10.

### Desempenho e memória nos testes

- `DesempenhoTests`: não rode um teste de mapa **sozinho** chamando a linha de base (pendura o runner). A/B sempre na **mesma carga** da cena (o HD mecânico oscila 2,5× entre rodadas).
- **Memória:** a máquina tem 16 GB. Depois de uma suíte grande de PlayMode (carrega as cenas de mapa), **reinicie o Editor** antes da próxima (o Editor caiu por falta de memória em 26/09).
- **A Unity cai sozinha em batch de vez em quando (desde 08/10/2026).** O log tem `Crash!!!` e uma pilha nativa (`tlsf_free`, `mono_domain_free`) e, conforme o momento, não há XML de resultado ou há o XML completo com saída diferente de 0. Não é teste falhando: é o motor (no dia, logo depois de uma atualização do Windows que corrompeu o compilador até reiniciar). Conduta: **repita a rodada**; só conta como falha o que aparece no XML. Se passar a cair toda vez, reinicie a máquina.

## 7. Desempenho

- Máquina mínima de referência: Ryzen 5 4600G (gráfico integrado), 16 GB, HD + SSD. Toda meta
  numérica é contra ela.
- **Medir antes, A/B depois, na mesma rodada.** Otimização sem número não entra.
- Runtime: sem LINQ, sem `Find` por quadro, sem alocar por quadro, sem `catch {}` vazio.

## 8. Onde registrar

| O quê | Onde |
|---|---|
| Resolveu algo do backlog | Apague a linha de [`PENDENCIAS.md`](PENDENCIAS.md) e explique em [`../Historico/PENDENCIAS_FECHADAS.md`](../Historico/PENDENCIAS_FECHADAS.md), **no mesmo commit** do código |
| Decidiu algo | [`DECISOES.md`](DECISOES.md): mova a linha de "esperam resposta" para "tomadas" |
| Mudou como um sistema funciona | O documento do sistema: [`../Jogo/REFERENCIA_DE_SISTEMAS.md`](../Jogo/REFERENCIA_DE_SISTEMAS.md) ou [`../Jogo/MODULOS_DO_JOGO.md`](../Jogo/MODULOS_DO_JOGO.md) (a descrição é do que **é**, nunca do que se pretende) |
| Mudou um número do estado do projeto | Refaça a §2 inteira de [`ESTADO_DO_PROJETO.md`](ESTADO_DO_PROJETO.md) (ela diz como contar) |
| Lição que vale para decisões futuras | Uma linha em [`../Historico/HISTORICO_DE_SESSOES.md`](../Historico/HISTORICO_DE_SESSOES.md) (não é changelog) |
| Documento novo | Entra no índice [`../README.md`](../README.md) |
| Número | Sempre com recorte (cena, máquina, data). "Não medi" quando não mediu |

## 9. Merge com a `main` (conferido em 09/10/2026)

`mapa/blocos` está **197 commits à frente** da `main`; a `main`, **7 à frente** da branch (era 116 em 26/09, 159 em 30/09). O que a `main` mudou desde o ancestral comum: `Assets/message.md` e `Assets/Docs/CATALOGO_DE_FEATURES.md` (novos), edições em `Assets/Scripts/COMPARATIVOS_EXTERNOS.md`, `DevScene.unity`, `Player.prefab`, duas cenas temporárias `InitTestScene*.unity` e `predio.mat` removido. **Desde 09/10 os documentos moram em `Docs/`**: `message.md` virou `Docs/Historico/GUIA_DE_IMPLEMENTACAO_2026-09.md` e o `COMPARATIVOS_EXTERNOS.md` virou `Docs/Historico/COMPARATIVOS_EXTERNOS.md`. Passos:

1. Decidir as cenas grandes **antes** (PENDENCIAS C3, DECISOES P2) — senão o merge leva `Kanto3D` e `KantoPecas` para a `main`.
2. Unificar a versão do Unity (`6000.5.10f1`).
3. Commitar ou descartar o que está solto na árvore (ESTADO §1, risco 4) e rodar as duas suítes inteiras **antes** do merge.
4. `git merge main` num clone ou branch de teste primeiro. Esperado: o git acompanha o renomeio de `COMPARATIVOS_EXTERNOS.md` (conferir se a edição da `main` entrou no arquivo novo); `Assets/message.md` e `Assets/Docs/CATALOGO_DE_FEATURES.md` chegam como arquivos novos em `Assets/` — **mova os dois para `Docs/Historico/`** (e apague os `.meta`), porque documento dentro de `Assets/` vira asset da Unity. O `message.md` da `main` tem o "aviso de alinhamento" (o atalho de Pokédex na tecla E, `AtalhoPokedex`, que ignora o `GerenciadorEstados`): o mesmo texto está no começo do guia arquivado.
5. **Abrir no Unity** `DevScene` e `Player.prefab` (mesclaram sem conflito de texto na simulação de 30/09, mas YAML de cena mesclado precisa ser conferido) e apagar as cenas temporárias `InitTestScene*`.
6. Rodar as duas suítes inteiras; só então push.
