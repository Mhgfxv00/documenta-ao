# Mapa de Kanto — revisão técnica de 26/09 e a divisão em cenas por área

| | |
|---|---|
| **O que este documento responde** | Por que o mapa de Kanto é do jeito que é (colisores, texturas, pacote visual), o que a revisão de 26/09 achou de errado, e como e por que o mapa foi dividido em cenas por área |
| **Quando abrir** | Antes de mexer na estrutura do mapa, de colisor, de textura ou de cena; antes de decidir a adoção da multicena |
| **O que ele NÃO é** | Receita de montagem ([`MONTAGEM_DE_MAPAS.md`](MONTAGEM_DE_MAPAS.md)) nem medição de desempenho ([`MAPAS_E_DESEMPENHO.md`](MAPAS_E_DESEMPENHO.md)) |
| **Origem** | Seções 12.1 e 17 de [`../Projeto/PENDENCIAS.md`](../Projeto/PENDENCIAS.md), movidas **sem edição** em 09/10/2026. O que continua aberto está resumido em PENDENCIAS §8 |
| **Atenção** | Datas de 26/09 a 30/09. A `KantoMulticena` existe e funciona; **a adoção dela (e a saída de `Kanto3D`/`KantoPecas` do git) não foi decidida** |

---

# Parte 1. Revisão de 26/09

Consolida a revisão do que foi feito com o mapa desde que entrou no projeto, e a resposta sobre
o pipeline do Blender (antes em `Assets/mapa pelo blender.txt`). **Três afirmações daquela
resposta estavam erradas e foram corrigidas aqui pela medição**: os colisores não vêm do glTFast;
as peças repetidas não viraram geometria única; e "o export perdeu o link do atlas" contradiz o
diagnóstico já documentado pelo klz (item C).

### Linha do tempo

| Quando | Quem | O quê |
|---|---|---|
| 15/09 `79149fb6` | klz | Pacote visual (`Assets/Omega/Visual`: céu, água, vegetação, toon, grama instanciada, cachoeira, `OmegaVisualMontar`); `CorrecoesDoMundo` (colisores em runtime, pedras, caixas de água); `OmegaKanto3D` (monta a cena a partir do glb). A cena **não** entrou nesse commit |
| 16/09 `9039f602` | klz | `kanto.glb` novo (25 MB) e Kanto3D "remontada", primeiro commit da cena — já com **53.978 MeshColliders salvos**. O próprio commit registra: carga de 17,5 s → 22,7 s, "otimização a decidir". Entram também `GramaPontos.bytes` (28 MB), a história importada, `MarSemLimbo` e `CullingDeHistoriaPorCelula` |
| 24/09 | nós | Occlusion culling feito e revertido; M17: 306 materiais clonados com o atlas religado (`MateriaisCorrigidos/`) |
| 26/09 | nós | Occlusion culling reaplicado |

### O que o Blender preservou

O equivalente de "prefab" no Blender é o **Duplicado Vinculado** (Alt+D) e a **Instância de
Coleção**. Isso **sobreviveu** ao export: são 74.098 objetos com malha, mas só **5.787 malhas
diferentes**; 2.933 delas são compartilhadas por 71.244 objetos (`tufo_real` ×6.414, `cubo_Cerca`
×2.675, `cubo_RochaCaverna` ×2.167, `pedra_real` ×1.409, `cubo_LabParede` ×1.264, `cubo_Degrau`
×1.101, `arbusto_real` ×1.097, `arvore_v2` ×611). O que se perde é a identidade de prefab: o
glTFast cria objetos soltos que só dividem a malha, então colisor, LOD e material não se definem
"uma vez por tipo". E a hierarquia é plana: **23.694 filhos diretos** na raiz `Kanto`.

### Redundâncias e incoerências (com evidência)

- **A. Colisor implementado quatro vezes, com regras diferentes.** *(26/09: a lista de exclusão
  virou uma só, `ColisaoDoMapa`, usada pelos quatro. Continua em aberto onde o colisor nasce —
  montagem ou runtime — e os 53.978 salvos na cena.)*
  - Onde: `OmegaKanto3D` (editor), `CorrecoesDoMundo` (runtime), `OmegaHistoriaPallet` e
    `OmegaImportarHistoriaFireRed` (os dois últimos como colisores "temporários").
  - A lista de exclusão do `OmegaKanto3D` é mais curta: não exclui `lamp`, `rug`, `pillow`,
    `portafolha`, `portavidro`, `macaneta`, `capacho`, `doormat`, e usa `plant_` onde os outros
    usam `plant`.
  - A intenção escrita no código é colisor em runtime, com a cena limpa. A cena foi salva com
    os 53.978 da montagem de editor.
  - Consequências: é a causa dos 87 MB e da carga lenta (A3); tapete, luminária e maçaneta
    colidem, embora a regra de runtime os excluísse; o cozimento paralelo do `CorrecoesDoMundo`
    encontra 1 colisor pra fazer.
  - Milhares de `cubo_*` têm MeshCollider onde BoxCollider basta (só os 4 maiores grupos somam
    ~7.200 objetos).
- **B. O pacote visual nunca foi aplicado na cena versionada.** Toon (63 materiais), água, céu,
  vegetação, cachoeira e grama instanciada têm **zero referências** na Kanto3D desde o primeiro
  commit dela. Só rodam os scripts que se instalam sozinhos (`PaisagemDistante`,
  `CorrecoesDoMundo`, `MarSemLimbo`). Os 28 MB de `GramaPontos.bytes` só são lidos por um
  `GramaDados.asset` que nenhuma cena usa.
- **C. Dois diagnósticos de textura que se contradizem — um deles é o nosso M17.**
  - O klz documentou em `OmegaVisualToon.cs` que a cor, no Blender, vinha de nós de shader que
    o glTF não exporta. O atlas aplicado pela UV (em metros) dá "listras coloridas". Por isso o
    toon descarta o atlas e usa a paleta de Kanto (`kanto_paleta.json`).
  - O M17 fez o oposto: religou o `kanto_atlas` em 306 clones (shader glTF PBR).
  - Nenhum dos dois caminhos está aplicado de forma coerente na cena, e o M17 pode ter ido na
    direção errada. **O grupo que vai trabalhar as texturas precisa decidir entre os dois antes
    de mexer.**
- ~~**D. Correções refeitas a cada carga em vez de na fonte.**~~ **Resolvido em 26/09** (M7): pedras e
  paisagem distante salvas nas duas cenas; `CorrecoesDoMundo` apagado.
- ~~**E. Dois rastreadores de "último ponto seguro" no mesmo jogador.**~~ **Resolvido em 26/09**: o
  resgate de queda do `CorrecoesDoMundo` foi removido a pedido; fica só o `MarSemLimbo` (piso
  invisível sob o mar e bloqueio do mar aberto sem Surf).
- **F. Resíduos.**
  - `mapa.isStatic = true` no `OmegaKanto3D` só marca a raiz, não os filhos.
  - O `message.md` desta branch aponta para `Assets/Docs/CATALOGO_DE_FEATURES.md`, que só
    existe na `main`.

### Feito em 26/09: `KantoPecas` (mapa em prefabs)

`Omega > Kanto - extrair pecas e montar cena nova` (`OmegaKantoPecas.cs`) gerou
`Assets/Scenes/Legado/KantoPecas.unity`: 23.694 instâncias de 199 prefabs, sem dependência do glb,
posições idênticas à original. Materiais neutros com o nome do original (sem cor nem textura, pro
grupo decidir), colisor pela malha (33.920 caixas + 18.615 de malha, contra 53.979 de malha),
occlusion assado. Carga ~2× mais rápida e quadro ~30% mais leve que a Kanto3D (números em
`message.md` §10). Resolve na cena nova os itens A (em parte), F e 2 da lista abaixo.

Não vieram: história, NPCs e professores (só na Kanto3D). A paisagem distante, que só se instalava
na cena "Kanto3D", agora está salva nas duas. O colisor do jogador virou **cápsula** (raio 0,375,
altura 2): a caixa de 0,75 × 1 m na diagonal ficava mais larga que o vão de porta. **Piloto de colisor por prédio feito em 26/09** no `PredioCeladon` (12
instâncias, dois andares, com interior): 318 → 94 caixas. Ficam paredes, cantoneiras, pisos, lajes,
forro, base, degraus, muretas e guarda-corpo; móvel grande vira uma caixa com **1,2 m de altura
mínima** (baixo demais, o jogador sobe nele como degrau — aconteceu com a mesa); saem janelas,
frisos, rodapé, platibanda, verga, molduras, TV/tela sobre móvel e placas. **O primeiro teste não
valia:** o `CorrecoesDoMundo` punha de volta, a cada carga, `MeshCollider` em toda peça sem colisor
(2.688 no PredioCeladon). Esse trecho e o resgate de queda foram removidos em 26/09. Refeito sem
eles: a porta passa (reto e a 45°), e do centro o jogador bate em algo nas quatro direções; câmera
dentro (o forro segura). O spawn da `KantoPecas` ficou na porta de um PredioCeladon, pra teste. Medição de todas as estruturas: 144, 98
com interior, 26.782 colisores → ~6.640 estimados. As outras seguem por peça
(ver a conversa sobre casa (ver a conversa sobre casa
de transição × casa em que se entra). Armadilha registrada: **Batching Static quebra o
`CorrecoesDoMundo`** (malha vira "Combined Mesh"; os tufos de mato ganharam colisor gigante e o
jogador ficou preso) — os prefabs não usam, e a regra de colisor recusa malha combinada.

### O que fazer, em ordem

1. **Fonte única de colisor.** Uma regra só, compartilhada pelos quatro lugares, rodando num
   lugar só (na montagem/importação, não em runtime). `cubo_*` usam BoxCollider; vegetação e
   decoração ficam sem colisor; o resto usa MeshCollider. Tirar os 53.978 salvos na cena.
2. **Peças repetidas viram prefab no Unity.** As 2.933 malhas compartilhadas viram prefabs, cada
   um com colisor, LOD e material definidos uma vez.
3. **Decidir cor/textura (item C)** antes de qualquer outra mudança visual.
4. **Aplicar ou descartar de vez o pacote visual (item B).** Se descartar, remover os 28 MB de grama.
5. **Levar para a fonte as correções de runtime (D, M7)** e unificar o resgate de queda (E).
6. **Agrupar a hierarquia por área** (cidade, rota, interior): é a base para dividir em cenas e
   ter occlusion por área.
7. **Dividir em cenas por área.** O `SistemaTransicao` já foi desenhado para isso; resolve junto
   o C3 (arquivo acima do limite do GitHub).

**Para conteúdo novo de mapa:** modelar e exportar peça por peça (parede, casa, prop — não a
cidade inteira), montar no Unity com prefab, e conferir o material como asset do Unity depois
de importar, em vez de assumir que está certo só porque abriu sem erro. Não é preciso refazer
Kanto de uma vez: é decisão de escopo que envolve quem modela.

---

# Parte 2. Arquitetura de conteúdo e cenas por área (análise de 27/09, primeira versão feita)

Registro de uma análise, não trabalho feito. Estimativas em dias de uma pessoa, com testes e medição em build.

**Para que serve o `CatalogoJogo`:** transformar os ids de texto do save e da rede de volta em assets, e ser
a lista oficial do que está no jogo (o validador e o auditor leem dela). O problema nunca foi ter catálogo; é
ele guardar **referência direta**, o que na Unity significa "carregar junto". A O0 tirou as imagens (985 → 77
MB); o que ele ainda arrasta são dados leves (espécies, golpes, itens) — pouco, mas cresce com o conteúdo.

**Como outros projetos fazem:**

| Abordagem | Onde se usa | Custo |
|---|---|---|
| Catálogo com referências diretas (o nosso) | Projeto pequeno e médio, a maioria dos fangames em Unity | Tudo na memória; funciona enquanto cabe |
| `Resources.Load` por caminho | O que a O0 fez com as imagens | Tudo em `Resources` entra sempre na build e pesa na inicialização; a Unity desencoraja em projeto grande |
| **Addressables** | Padrão atual da Unity, jogos comerciais | Carregamento assíncrono com contagem de uso; muda o código que hoje pega o asset na hora |
| Dados fora de asset (JSON/CSV/SQLite, estilo Showdown/PokeAPI) | RPGs e card games com muito conteúdo tabelado | Perde o Inspector como editor, a menos que se escreva ferramenta |

### Estimativas

**A. Imagens, modelos e áudio para Addressables — 4 a 7 dias.** Pacote não instalado; 9 arquivos de runtime
usam `Resources.Load`.

| Passo | Dias |
|---|---|
| Instalar e criar grupos (folhas de batalha, retratos, ícones, modelos 3D, áudio) | 0,5 |
| Espécie/item guardam endereço; a ferramenta da O0 (`SpritesDoCatalogo`) converte | 1 |
| Carregamento assíncrono nos pontos de uso (sprite de batalha, equipe, Pokédex, PC/mochila, visor 3D), com liberação | 2 a 3 |
| Trocar os 9 `Resources.Load` e esvaziar `Resources` | 0,5 |
| Testes, build, medição, build de conteúdo no pipeline | 1 a 2 |

Ganho: liberação de memória de verdade, sem o índice do `Resources` na partida, caminho para mobile/DLC.
Risco: toca telas de UI, que são do outro lado do grupo — combinar.

**B. Dados fora de asset — 3 a 5 semanas. Recomendação: não fazer.** Exportar e carregar tabelas: 3–4 dias. O
grosso é o comportamento (regras/efeitos/condições em `[SerializeReference]` de ~925 golpes e ~500
habilidades) virar uma linguagem em JSON com serializador e validador: 1,5–2 semanas. Ferramenta de edição: 1
semana. Refazer validador, auditor, geradores e ~40 classes de teste: 1 semana. Ganho: poucos MB e diffs
mais legíveis. Só vale se o conteúdo passar a vir de fora (modding, servidor definindo balanceamento) ou se
entrar gente de design editando tabela.

**C. Dividir o mapa em cenas por área — 5 a 8 dias.** Resolve junto: cenas de 87/73 MB no git, memória do
Editor (caiu em 26/09), limite de 100 MB do GitHub, merge com a `main` (C3/O1).

Como fica:
- Uma cena por área jogável (Pallet, Rota 1, Viridian…), ~20–30 cenas — não uma por quadrado de 200 m.
- Cena persistente leve com jogador, câmera, céu, paisagem distante e sistemas (o bootstrap já é isso).
- **Carregador por proximidade (código novo):** a área atual e as vizinhas ficam carregadas; as distantes
  descarregam. É o culling por área de hoje descarregando de verdade. O `SistemaTransicao` atual troca de cena
  inteira com fade (`LoadScene`) — serve para interior e caverna, não para andar no mundo aberto.
- Interiores continuam cenas próprias com transição, como a `Casa`.
- Occlusion culling por cena (arquivos pequenos).

| Passo | Dias |
|---|---|
| Definir fronteiras com o grupo do mapa (rotas e cidades) | 0,5 |
| Ferramenta que corta a `KantoPecas` em cenas (dos prefabs que ela já tem) e move NPCs/gatilhos/encontros da história para a área certa | 2 a 3 |
| Carregador por proximidade: pré-carga assíncrona, descarga, folga de distância | 1,5 a 2 |
| Ajustar quem assume "o mapa é uma cena só": spawn, posição no save, atores por id, a raiz `Kanto`, `CullingDoMapaPorArea`, paisagem distante | 1 a 2 |
| Occlusion por cena, bot atravessando fronteira, medição | 1 |

Riscos: emenda visível na fronteira se a pré-carga for curta; atores da história FireRed amarrados por id
(conferir quando mudarem de cena); combinar com quem está mexendo na `KantoPecas`.

### Ordem proposta (a decidir)

1. **C — cenas por área** (mais urgente: destrava git, merge e memória). Primeiro passo: propor as fronteiras
   a partir dos 69 quadrados e da hierarquia da história, para o grupo do mapa aprovar.
2. **A — Addressables**, já incluindo as cenas novas.
3. **B — não fazer**; registrar como decisão fechada se aprovado.

### §17 C — primeira versão feita (27/09): `Assets/Scenes/Mundo/KantoMulticena`

- Ferramenta `OmegaKantoMulticena` (menu Omega > Kanto multicena): 1. relatório das áreas, 2. separar.
  Parte da cópia `KantoPecas2.unity`; a `KantoPecas` original não foi tocada. Refazer = rodar de novo.
- **`KantoBase.unity` (47 KB):** luz, spawn, paisagem distante, as 9 peças com mais de 400 m (chão e mar,
  raiz `KantoFixo`) e o `CarregadorDeAreas` — sempre carregada, para nunca faltar chão.
- **14 cenas em `Areas/`** (maior 15 MB; antes uma só de 76 MB): Pallet, Viridian, Pewter, Liga, Celadon,
  CeruleanSaffron, Lavender, Vermilion, Fuchsia, Cinnabar, Sevii123, Sevii45, Sevii67, Interiores. Cada
  uma com a raiz "Kanto" e os quadrados `Area_x_y` dela: o culling por área continua valendo por cena.
  **O agrupamento é proposta** (`OmegaKantoMulticena.Regiao`), tirada dos nomes de estrutura em cada
  quadrado — ajustar com o grupo do mapa e rodar de novo.
- **`CarregadorDeAreas`:** carrega aditivamente as áreas a até 250 m da borda e descarrega acima de 400 m
  (folga contra piscar na divisa); área ainda chegando não é esquecida.
- Base e áreas entraram no Build Settings (carregar por caminho exige).
- `KantoMulticenaTests` (2): só as áreas perto carregam, o jogador fica no chão; teleportar para as
  Sevii traz a área nova e descarrega as do continente.

**Falta:** occlusion culling por cena (a origem tinha 22 MB de dado assado; multicena precisa assar por
cena); medir carga, memória e quadro contra a `KantoPecas` numa build; pré-carga mais longa se aparecer
emenda na fronteira; decidir se `Kanto3D`/`KantoPecas` saem do Build Settings e do git depois que a
multicena for adotada (O1, C3).

**§17 C — NPCs na multicena (30/09).** Ferramenta `Omega > Kanto multicena - 4. tutores, vendedor e treinador de teste`
(`OmegaKantoMulticenaNpcs`), reaplicável: cada NPC vai para a cena da **área** que cobre o ponto dele (carrega e descarrega junto
com o chão), num grupo `NPCsKantoMulticena`, nunca na base. **14 tutores** (`Professor_*`, posições lidas da Kanto3D — o mundo é o
mesmo), mais **`Vendedor_Teste`** e **`Treinador_Teste`** 4 m ao lado do spawn (Celadon). A loja `LojaDeTeste_TodosOsItens` vende os
**479 itens que têm preço**; os **2.202 de preço 0** ficaram de fora, porque a loja nunca vende preço 0 (item de história, molde da
cápsula) — dar preço a eles é decisão de conteúdo. O treinador (`TreinadorDeTeste`, 3 Pokémon de nível 12, revanche ligada) é para
testar a luta. **Conferir no jogo:** 3 tutores (Espiritos, Fundamentos, Sono) ficaram no `y` da Kanto3D porque o raio não achou o chão
da multicena ali (podem flutuar ou afundar), e o Menu/SistemaDialogo que os NPCs procuram na cena precisam existir onde a multicena roda.
Teste: `KantoMulticenaNpcsTests` (lê os arquivos de cena, sem abrir nenhum).

**§17 C — a história na multicena, fatia de validação (11/10).** Ferramenta `OmegaHistoriaMulticena` (menu `Omega > Kanto multicena - 5. historia de Pallet`), reaplicável: põe na cena da área de Pallet (`Kanto_Pallet.unity`), num objeto `Historia`, a mãe, a irmã do rival, o Carvalho, o rival, dois moradores, o Carvalho da rua, o ponto de chegada ao laboratório e o gatilho ao norte. Usa os mesmos roteiros (`Assets/Data/Mundo/Roteiros/Pallet`) e as mesmas coordenadas da `Kanto3D` (mãe em 28,5 / 2,35 / 109 nas duas); a montagem é a mesma de `OmegaHistoriaPallet.MontarPalletEm`, extraída para servir aos dois. Não abre a `Kanto3D` e não roda o gerador de roteiros (congelado). A `Kanto3D` segue com a cópia dela até a P2. Testes: `HistoriaMulticenaTests` (lê o arquivo de cena) e `HistoriaMulticenaPlayModeTests` (os atores carregam com a área, ficam no chão e a mãe conversa pelo executor de roteiro). **Falta:** o resto da história (Viridian em diante) e as placas e gatilhos de encontro.
