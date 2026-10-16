# Como montar a cena do zero

| | |
|---|---|
| **O que este documento responde** | Quais peças uma cena de jogo precisa e como se ligam no Inspector, passo a passo, **do zero** |
| **Quando abrir** | Para reconstruir a base de uma cena (sistemas, assets, jogador, grama, batalha, diálogo, treinador) ou entender o que cada peça exige |
| **O que ele NÃO é** | A receita do dia a dia. Para montar **conteúdo** num mapa que já existe (porta, mato, NPC, loja, roteiro), use [`MONTAGEM_DE_MAPAS.md`](MONTAGEM_DE_MAPAS.md). Desde a cena `Bootstrap` (03/09) os sistemas e o jogador nascem nela, e as cenas de jogo contêm só o conteúdo delas |
| **Conferido em** | Texto de 02 a 05/09, movido do antigo README em 09/10. **Os passos não foram reexecutados**; os nomes de menu foram conferidos contra o código em 09/10 |
| **Catálogo dos menus** | [`../Projeto/FERRAMENTAS_DO_EDITOR.md`](../Projeto/FERRAMENTAS_DO_EDITOR.md) |

> **Nota de 05/09:** desde a cena `Bootstrap` (03/09), montar os sistemas à mão só é
> necessário para uma cena nova de conteúdo — a `Bootstrap` já é dona de tudo e as cenas de
> jogo contêm só o conteúdo delas. Os passos abaixo continuam válidos como referência do que
> cada peça precisa, e o passo 1 continua servindo para reconstruir do zero.
>
> **Depois de criar assets novos, rode `Pokemon > Popular catalogo selecionado com todos os
> assets`** (com o `CatalogoJogo` selecionado): asset fora do catálogo grava no save e
> **some ao carregar**, porque o id não resolve de volta. O `ValidadorDeCatalogo` e o
> `AuditorDeAlcancabilidade` rodam como teste e acusam os dois erros mais comuns — asset
> fora do catálogo, e asset que existe mas nada no jogo aponta para ele.

1. **Menu `Pokemon > Criar objeto de Sistemas`** — cria o GameObject `Sistemas` com
   `GerenciadorEstados`, `SistemaInventario`, `SistemaBatalha`, `SistemaEncontros`,
   `SistemaDialogo` e `SistemaSave`. Eles se acham sozinhos; só faltam dois campos:
   - `SistemaBatalha.tabelaTipos` → seu asset de `TabelaTipos`
   - `SistemaSave.catalogo` → seu asset de `CatalogoJogo`

2. **Menu `Pokemon > Criar tipos e caracteristicas basicos`** — cria os assets dos sete tipos
   e das cinco características iniciais e preenche a `TabelaTipos` com as relações clássicas.
   Roda de novo sem duplicar nada.

3. **Assets** (menu `Create > Pokemon`): 3 `Especie`, alguns `Golpe`, `Habilidade` (opcional),
   1 `Pokebola`, 1 `ItemCura`, 1 `TabelaEncontros`, 1 `CatalogoJogo`. Para preencher as
   quatro listas do catálogo, selecione o asset e rode **`Pokemon > Popular catalogo
   selecionado com todos os assets`** em vez de arrastar um por um — reconstrói do zero
   com tudo que existir no projeto, sem duplicar nem deixar lixo ao rodar de novo.

4. **Jogador**: precisa de `EntradaJogador` no mesmo objeto (ou num pai) de `PlayerMovement`
   e `PlayerInteracao` — é dele que os três (mais `CameraRotate`) pegam o mapa de Input.
   No `DadosJogador`, monte a equipe inicial — basta escolher a espécie e o nível em cada
   elemento; XP, golpes e HP são preenchidos no `Awake`. Na mochila, adicione Pokébola e
   Poção.

5. **Grama**: objeto com Collider `Is Trigger`, componente `AreaDeEncontro` e ao menos um `ContextoDeEncontro`
   com `TabelaEncontros` (ou a `tabelaPadrao`, que existe como reserva).

6. **Batalha**: no objeto que tinha o `Luta2d` (agora `BatalhaUI`), ligue o painel, os textos
   de nome/nível/HP, as barras e os containers + prefabs de botão. Os botões do painel de
   ações chamam `BotaoLutar`, `BotaoMochila`, `BotaoEquipe`, `BotaoFugir` e `BotaoVoltar` —
   confira no Inspector que cada `Button.OnClick` aponta exatamente um desses métodos (sem
   duplicar nem sobrar chamada de outro script; um `onClick` com alvo nulo simplesmente não
   dispara nada, silenciosamente). `painelAprenderGolpe` é opcional como os outros: sem
   ninguém ouvindo `SistemaBatalha.OnPedirEscolhaDeGolpe`, um golpe novo com a ficha cheia
   esquece o mais antigo sozinho (sem travar a batalha esperando uma escolha que nunca
   chega) em vez de abrir a tela de escolha.

7. **Diálogo**: um `DialogoUI` com painel + texto, e um `Npc` no mundo.

8. **Treinador**: um `NpcTreinador` no mundo (mesmo Collider trigger do `Npc`), com a
   equipe montada no Inspector (espécie + nível de cada um, igual à equipe do jogador),
   nome, falas e recompensa em dinheiro. Preencha `inventarioDeBatalha` (itens
   `UsavelEmBatalha`) se quiser que ele use item na própria luta.

