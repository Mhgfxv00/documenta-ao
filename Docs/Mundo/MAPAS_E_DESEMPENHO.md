# Mapas 3D: leves, amplos e populados

Registro da sessão de 29–30/09/2026. Reúne o método de medir, o que foi medido na KantoMulticena, o que
mudou no projeto e as diretrizes para os próximos mapas. Complementa o [`COMO_TRABALHAR.md`](../Projeto/COMO_TRABALHAR.md) (regras de
trabalho) e o [`PENDENCIAS.md`](../Projeto/PENDENCIAS.md) (itens O1 a O11 falam de desempenho).

**Todo número de tempo aqui é do Editor (Unity 6000.5.10f1), nesta máquina.** Nenhuma build foi medida
ainda (PENDENCIAS O11). Contadores (SetPass, triângulos, projetores de sombra) são confiáveis entre
rodadas; milissegundos oscilam de 1 a 2 ms dentro da mesma configuração e, entre rodadas, chegaram a
derivar 4 ms para cima em todas as linhas.

---

## 1. Como medir

### 1.1 Profiler (Window > Analysis > Profiler)

Responde: **onde o tempo do quadro vai e se o limite é CPU ou GPU.**

1. Abra o Profiler, escolha **Play Mode** e clique em gravar (círculo vermelho).
2. Módulos que importam para mapa: **CPU Usage** (barra azul = renderização, verde = scripts, amarelo =
   física), **GPU Usage** (precisa estar habilitado; no Editor pode vir vazio) e **Rendering** (SetPass Calls,
   Batches, Triangles, Shadow Casters).
3. Clique num quadro do gráfico. Na parte de baixo, troque a vista para **Timeline** (ver o que roda em
   paralelo) ou **Hierarchy** (ordenar por *Time ms* ou *GC Alloc*).
4. Como ler o limite: se a linha `WaitForTargetFPS`/`Gfx.WaitForPresentOnGfxThread` domina, a CPU está
   esperando a GPU (limite = GPU). Se a thread principal está cheia e a GPU tem folga, o limite é CPU.
5. Para caçar picos de GC: ordene a Hierarchy por *GC Alloc* e procure o quadro alto.
6. Feche a **Scene view** ao medir: ela renderiza a cena de novo e infla todos os contadores.

### 1.2 Frame Debugger (Window > Analysis > Frame Debugger)

Responde: **o que exatamente é desenhado, em que ordem, e por que dois desenhos não viraram um só.**

1. Com o jogo rodando, abra a janela e clique em **Enable**. O jogo congela no quadro atual.
2. A lista da esquerda são os eventos de desenho. Procure os grupos **RenderLoop.Draw** e as passadas de
   **shadow** (*Main Light Shadowmap*, uma linha por cascata). Clique num evento para ver, no Game view,
   só o que foi desenhado até ali.
3. No painel da direita, cada desenho mostra o shader, o keyword set e, quando **não** juntou com o vizinho,
   o motivo (*Why this draw call can't be batched with the previous one*).
4. Uso típico aqui: contar quantas passadas de sombra existem e quantas peças entram em cada uma.

### 1.3 Development Build com o Profiler conectado

O Editor mente para mais (ver seção 5). O número que vale é o da build.

1. **File > Build Profile** (Build Settings), plataforma Windows, marque **Development Build** e
   **Autoconnect Profiler**. Marque também **Deep Profiling Support** só se precisar (deixa tudo lento).
2. Cenas: para a KantoMulticena, a `Bootstrap`, a `KantoBase` e as 14 cenas de `Areas/` precisam estar na lista.
3. **Build And Run**. Com *Autoconnect Profiler* o Profiler do Editor conecta sozinho.
4. Se não conectou: no Profiler, o menu **Playmode** (canto superior esquerdo) vira a lista de alvos; escolha o
   **player** da máquina. Para outra máquina na rede, use o IP no mesmo menu; no Android, *Autoconnect* funciona por USB.
5. Grave, ande pelo mapa, salve o resultado com **Save** (arquivo `.data`) e guarde junto do relatório.
6. Para um número sem o peso do Profiler, faça a mesma build com o Profiler desconectado e leia o
   `DesempenhoTests` (ele imprime o quadro-resumo no log do player).
7. `Frame Timing Stats` (Player Settings > Other Settings) precisa estar ligado para o tempo de GPU aparecer.

### 1.4 Medição repetível no projeto (`DesempenhoTests`)

`KantoMulticena_RenderPorConfiguracao_Relatado` carrega a `KantoBase`, mede em dois pontos (spawn e centro de
Celadon), troca uma configuração do URP por vez e escreve a tabela em `Temp/desempenho_multicena.md`.

```
unity command run_tests --mode PlayMode --async_tests --filter KantoMulticena_RenderPorConfiguracao_Relatado --filter_type testName
```

Regras: comparar só linhas da **mesma rodada**; repetir a rodada para saber o ruído; o teste restaura o asset
do URP no fim. Colunas: ms/quadro, CPU ms e GPU ms (`FrameTimingManager`), SetPass, triângulos, projetores de
sombra. As colunas `draws` e `batches` saem 0: esses contadores não existem no Editor nesta versão.
O que ainda **não** se registrava: SetPass, sombras e censo de material. Agora entra no teste.

---

## 2. O que foi medido na KantoMulticena (30/09/2026)

### 2.1 Censo (áreas carregadas ao redor de Celadon)

- **51.584 renderers**, todos ativos; **51.582 projetam sombra**; 51.576 marcados *static*; **nenhum** com `LODGroup`.
- **384 materiais únicos, 1 shader** (URP/Lit em 51.576 usos). O SRP Batcher já estava ligado
  (`m_UseSRPBatcher: 1`) e agrupa por shader, não por material: reduzir materiais **não** é a alavanca aqui.
- GPU Resident Drawer desligado (`m_GPUResidentDrawerMode: 0`), MSAA 4x no asset PC.

### 2.2 Configuração de render, antes → depois (mesmas condições, 2 rodadas)

Antes: sombra a 150 m, 4 cascatas, mapa 4096. Depois: 60 m, 2 cascatas, mapa 2048.

| Ponto | Configuração | ms/quadro (rodadas 1 / 2) | GPU ms | SetPass | Triângulos (mil) | Projetores |
|---|---|---|---|---|---|---|
| spawn | antes | 25,6 / 25,6 | 17,6 / 17,7 | 84 | 2.952 | 5.848 |
| spawn | depois | 22,5 / 21,9 | 14,9 / 14,6 | 58 | 2.235 | 3.771 |
| spawn | sem sombra (teto do ganho) | 20,9 / 19,8 | 12,7 / 12,1 | 36 | 1.494 | 0 |
| Celadon | antes | 21,5 / 19,5 | 14,7 / 13,3 | 81 / 77 | 2.879 / 2.799 | 4.129 / 3.747 |
| Celadon | depois | 17,4 / 15,9 | 11,0 / 10,1 | 45 | 2.089 / 2.037 | 1.168 / 1.182 |
| Celadon | sem sombra | 15,7 / 14,6 | 9,4 / 8,7 | 36 | 1.392 / 1.341 | 0 |

Alavancas isoladas no spawn (rodadas 1 / 2), contra "antes" = 25,6:

| Alavanca | ms/quadro | Efeito nos contadores |
|---|---|---|
| distância 60 m | 24,1 / 24,1 | projetores −22%, SetPass 84 → 80 |
| distância 100 m | 24,8 / 24,4 | quase nada |
| 2 cascatas | 23,8 / 23,6 | **SetPass 84 → 60, triângulos −22%** |
| mapa 2048 | 24,8 / 24,3 | contadores iguais (só custo de preencher o mapa; dentro do ruído) |
| sem MSAA | 25,9 / 25,0 | **sem ganho** |

Leitura:

- **Sombra é a maior alavanca controlável:** desligar tudo tira 5 a 6 ms nos dois pontos (de ~25 e ~21).
  A configuração nova recupera cerca de 3 a 4 ms, mais da metade disso.
- **A cascata pesa mais que a distância:** cada cascata repete a cena inteira dentro do seu volume; 4 → 2 corta
  ~640 mil triângulos e 24 SetPass no spawn.
- **Limite:** CPU e GPU andam juntas. A GPU ocupa ~70% do quadro e cai junto com as sombras; o Profiler numa
  build em CPU integrada (o alvo, 4600G) é que diz qual dos dois aperta primeiro.
- MSAA 4x não aparece como custo aqui; não vale mexer sem medir numa GPU integrada.

### 2.3 Experimento (não aplicado): peça pequena não projeta sombra

Numa terceira rodada, com a configuração nova. Números só valem dentro da rodada (ela toda ficou ~4 ms mais lenta).

| Ponto | Regra | ms/quadro | Projetores |
|---|---|---|---|
| spawn | nenhuma | 29,0 | 3.771 |
| spawn | maior lado da caixa < 1 m (15.130 peças) | 28,6 | 2.563 |
| spawn | maior lado < 3 m (32.774 peças) | **26,1** | 1.538 |
| Celadon | nenhuma | 24,5 | 1.168 |
| Celadon | < 1 m | 22,5 | 1.045 |
| Celadon | < 3 m | **21,1** | 510 |

A regra de 3 m parece render outros 2,5 a 3,5 ms, mas **custa a sombra de cerca, poste, banco, mato alto e
detalhes de fachada**. Decisão visual do grupo do mapa; se aceita, entra no gerador (`OmegaKantoPecas`) e
não em runtime.

### 2.4 O que NÃO foi feito

- **LOD e combinação de malhas.** Com 51 mil renderers estáticos sem `LODGroup`, é o próximo passo natural, mas:
  o commit `2080e1c9` já registra jogador preso no mato por malha combinada/Batching Static, e o commit
  `1d34e3c5` mostra que cortar vegetação por distância não deu ganho mensurável. Sem medir numa build, o risco
  é maior que o ganho provável. Ordem sugerida: (1) GPU Resident Drawer no URP e medir; (2) `LODGroup`/impostor
  só nas casas e árvores grandes; (3) combinar malha só de chão e fachada, nunca do que tem colisor fino.
- **Ligar o SRP Batcher:** já estava ligado.
- **Build medida:** ver PENDENCIAS O11.

---

## 3. O que mudou no projeto

| Arquivo | Mudança |
|---|---|
| `Assets/Settings/PC_RPAsset.asset` | `m_ShadowDistance` 150 → 60, `m_ShadowCascadeCount` 4 → 2, `m_MainLightShadowmapResolution` 4096 → 2048 |
| `Assets/Settings/Mobile_RPAsset.asset` | as mesmas três (parâmetros estritamente mais leves; **não medido** no perfil Mobile) |
| `Assets/Scripts/Tests/PlayMode/DesempenhoTests.cs` | novo teste `KantoMulticena_RenderPorConfiguracao_Relatado` |

Vale para **todas** as cenas (o asset é do projeto, não da KantoMulticena). Efeito visível: sombras somem
além de 60 m do jogador e a transição entre cascatas fica mais próxima. Conferir em Play Mode no Kanto e
na DevScene.

---

## 4. Diretrizes para mapas leves, amplos e populados

### 4.1 O tamanho em metros importa pouco

O custo vem do que está **carregado e visível ao mesmo tempo**. O que o tamanho real muda: memória e disco
(conteúdo único), precisão de float acima de ~5–10 km da origem, e tempo de produção. A 5 u/s, 1 km leva
200 s a pé.

### 4.2 Parecer grande sem pesar

- Marcos altos e distantes (ficam na base da cena, só geometria: commit `e63a2f1d`).
- Profundidade em camadas: primeiro plano detalhado, meio simples, fundo em cartão/silhueta, névoa.
- Rotas com curva e relevo; algo novo a cada 30–60 s andando; bloqueios naturais que dão identidade a cada área.
- Regra dos triângulos do *Breath of the Wild*: formas grandes servem de marco, médias bloqueiam a vista,
  pequenas dão ritmo.
- Interiores em cena separada; deslocamento rápido depois que o jogador conhece o lugar.

### 4.3 Arquitetura recomendada

1. Kit modular pequeno (30 a 60 peças), atlas de textura, poucos materiais.
2. Chão em Terrain ou malha contínua por chunk, com relevo (o chão em blocos do `MapaBlocos` perde isso).
3. Chunks de 100 a 200 m em cenas aditivas (já é o `CarregadorDeAreas`).
4. **Orçamento por chunk escrito e testado** (renderers, SetPass, triângulos, projetores de sombra, memória).
   O teste da seção 1.4 é o ponto de partida.
5. Repetido em instancing/GPU Resident Drawer; combinado só o estático sem colisão fina.
6. Conteúdo lógico (portas, encontros, NPCs) em dado, separado do visual.

### 4.4 Build é mais leve que o Editor

Sim, em geral bem mais: o Editor paga Scene view, Inspector, gizmos, Profiler, Mono com checagens extras,
shaders sob demanda e ausência de otimizações de build. Não há fator fixo; a régua é uma **Development Build**
(seção 1.3). Medir os dois uma vez dá a margem.

### 4.5 Geração procedural determinística

Dá para gerar **o preenchimento** e um mapa **novo**; **reproduzir o Kanto atual** por regras não compensa.

| Objetivo | Dificuldade | Motivo |
|---|---|---|
| Reproduzir o Kanto de hoje | Alta, sem retorno | Cidades, rotas e prédios são autorais (vieram do glb e de peças). Regras que dessem o mesmo resultado teriam que codificar cada decisão à mão. |
| Preencher um mapa: árvores, pedras, mato, cercas | Baixa a média | Já existem `VegetacaoDoMapa` e `MapaBlocos` (grade de texto). Semente por chunk + regras por bioma. |
| Chão a partir da grade de texto | Média | O `MapaBlocos` já prevê "segunda grade de alturas alimentando o heightmap do Terrain". |
| Cidade nova | Média a alta | Ruas, lotes e prédios modulares por regra funcionam; a legibilidade (marcos, entradas) pede mão. |

Caminho híbrido: grade de texto define zonas, rotas e pontos de interesse (legível no git); o gerador
preenche por semente (o mesmo resultado em todos os clientes, útil para o multiplayer em rede); pontos de
interesse ficam à mão. Um primeiro chunk vertical (grade → Terrain → vegetação por semente → orçamento
medido) é uma tarefa de dias, não de semanas.

### 4.6 Alternativas às práticas comuns

| Alternativa | Ganha | Custa |
|---|---|---|
| GPU Resident Drawer (Unity 6) | menos CPU por renderer | exige Forward+ e shaders compatíveis; ganho **não medido** aqui |
| DOTS / Entities e subcenas | escala de milhares de objetos | reescrever a integração com física, colisão e multiplayer |
| Addressables por chunk | controle de memória e download | mais infraestrutura que cenas aditivas |
| HLOD e impostores | horizonte denso sem custo | ferramenta própria ou pacote de terceiros |
| Geração procedural com semente | disco e rede | mundo genérico sem pontos autorais |

### 4.7 Referências

- Streaming e cidade grande: [Megacity 2019](https://github.com/Unity-Technologies/Megacity-2019),
  [Megacity Metro](https://github.com/unity-technologies/megacity-metro).
- Streaming leve com Addressables e LOD: [Oculus Asset Streaming](https://github.com/oculus-samples/Unity-AssetStreaming).
- Streaming por cenas: [OpenWorldFramework](https://github.com/tiredamage42/OpenWorldFramework) (código não avaliado).
- HLOD com impostores: [discussão no fórum da Unity](https://discussions.unity.com/t/hlod-streaming-octahedral-impostors-using-megacity-subscenes/742223).
- Design de mundo: [análise do BotW](https://www.gamedeveloper.com/design/breath-of-the-wild-open-world-analysis-gravity-to-go-forward),
  [5 lições do BotW](https://www.gamedeveloper.com/design/5-design-lessons-learned-from-i-the-legend-of-zelda-breath-of-the-wild-i-),
  [Design Secrets of BotW](https://80.lv/articles/the-design-secrets-of-breath-of-the-wild).
- Instancing e batching: [Unity Manual, GPU instancing](https://docs.unity3d.com/Manual/GPUInstancing.html).

---

## 5. Registrar desempenho a cada mudança

Regra nova para não repetir o buraco desta sessão (as medições anteriores registravam ms e alocação, mas não
SetPass, sombras nem a configuração testada):

1. Antes de mexer: rodar o teste da seção 1.4 e guardar `Temp/desempenho_multicena.md` (renomear com data).
2. Mexer em **uma** coisa. Rodar de novo. Comparar só o que saiu da mesma rodada.
3. Registrar aqui (ou na PENDENCIAS) a linha: o que mudou, os contadores antes/depois e se o ms saiu do ruído
   (~2 ms).
4. Ao fim de uma etapa, uma Development Build com o Profiler (seção 1.3) e as metas da PENDENCIAS O11.

## 6. Pendências e decisões

- Conferir visualmente as sombras novas (60 m, 2 cascatas) no Kanto e na DevScene.
- Decidir a regra "peça < 3 m sem sombra" (seção 2.3) com o grupo do mapa.
- Medir numa build na máquina mínima (O11) e no perfil Mobile.
- Testar GPU Resident Drawer numa área do Kanto.
- Escolher se o próximo mapa nasce pelo caminho híbrido da seção 4.5.
