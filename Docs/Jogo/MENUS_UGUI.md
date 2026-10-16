# Sistema de Menus

| | |
|---|---|
| **O que este documento responde** | Como montar e ligar as telas de menu no Editor |
| **Quando abrir** | Ao criar um painel novo, ou quando um botão não faz nada |
| **O que ele NÃO é** | Catálogo das telas do jogo (isso é o [`UI_DO_PROJETO.md`](UI_DO_PROJETO.md)) nem referência de sistemas. Como se monta uma cena está em [`../Mundo/COMO_MONTAR_A_CENA.md`](../Mundo/COMO_MONTAR_A_CENA.md) |
| **Atualizado em** | 05/09/2026 (movido de `Assets/Scripts/UI/` em 09/10). **Na revisão de 02/09 cobria 4 das 14 telas uGUI** (Mochila, Equipe, Configurações, Cartão de treinador); as demais são descritas em [`UI_DO_PROJETO.md`](UI_DO_PROJETO.md). Várias telas uGUI citadas aqui estão **desativadas** como placeholders desde 07/10 (a Toolkit as substituiu): leia [`UI_DO_PROJETO.md`](UI_DO_PROJETO.md) §2.1 antes |

Painéis inspirados na arquitetura do YAPU, mas **sem nenhuma dependência externa**:
compilam só com Unity + TextMeshPro.

## Por que não usar o YAPU direto

O repositório do YAPU no GitHub contém apenas o código-fonte. Ele depende de
bibliotecas que não vêm junto e que não são de graça:

| Dependência | Usos no YAPU | Situação |
|---|---:|---|
| `Sirenix` (Odin Inspector) | 583 | Pago (~US$ 55) |
| `WhateverDevs` (Core, Localization, SceneManagement, TwoDAudio) | 1106 | Libs do próprio autor |
| `DG` (DOTween) | 197 | Grátis |
| `Zenject` / `ModestTree` | 165 | Grátis |
| `AssetIcons`, `Proxima`, `log4net` | 9 | Pagos / avulsos |

O próprio README diz: *"Given the need for **paid plugins** and copyrighted
assets it would be very difficult to have a repo that people can contribute to."*

Por isso o clone foi movido para `_YAPU_Reference/` na raiz do projeto — fora de
`Assets/`, então o Unity não tenta compilar. Continua lá para consulta.

---

## Arquivos

**Infraestrutura**

| Arquivo | Papel |
|---|---|
| `PainelMenu.cs` | Classe base de todo painel. Animação de slide + fade, abrir/fechar. |
| `Menu.cs` | Controlador principal. Liga os botões aos painéis e manda no `PlayerMovement`. |
| `UtilUI.cs` | O que várias telas montam igual: sigla de status colorida, estágios de atributo, ficha de golpe. |

**Painéis do menu do jogador** (todos abertos por um `Menu.Abrir*`)

| Arquivo | Papel |
|---|---|
| `MochilaMenu.cs` | Lista de itens + detalhes + usar item (pelo `SistemaInventario`). |
| `PartyMenu.cs` | Lista da equipe + status; é também o seletor de alvo de item. |
| `PokedexMenu.cs` | Espécies vistas: categoria, altura, peso, habitat, descrição. |
| `TrainerCardMenu.cs` | Nome, retrato, tempo, badges, dex, dinheiro. |
| `ConfiguracoesMenu.cs` | Volume (mestre/música/efeitos), tela cheia, qualidade e **dificuldade**. `PlayerPrefs`. |
| `SlotsDeSaveMenu.cs` | Os três slots: resumo, salvar, carregar, apagar — e a frase de erro do `SistemaSave`. |
| `CaixaMenu.cs` | Armazenamento: mover entre equipe e caixa. |

**Telas que não são do menu do jogador**

| Arquivo | Papel |
|---|---|
| `TelaDeTitulo.cs` | Porta de entrada: continuar, novo jogo, sair. Fica na `Bootstrap`. |
| `BatalhaUI.cs` | A tela de batalha inteira. Só escuta os eventos do `SistemaBatalha`. |
| `EscolhaDeBatalhaMenu.cs` | Escolha de golpe/alvo dentro da batalha. |
| `LojaMenu.cs` | Compra e venda, aberto pelo `Vendedor`. |
| `DialogoUI.cs` · `InteracaoUI.cs` | Caixa de fala e o aviso de "aperte para interagir". |
| `AmbienteUI.cs` | HUD de exploração: período, clima e quanto falta para virar. |

> `DadosJogador.cs` já foi listado aqui, mas mora em `Data/` — ele é **dado de partida**,
> não UI, e a distinção é a regra que o projeto não quebra: UI pede, sistema altera.

---

## Montagem na cena

### 1. No jogador

Adicione **`DadosJogador`** no mesmo GameObject que tem `PlayerMovement`.
Preencha nome, retrato, dinheiro, badges e as listas de equipe/mochila no Inspector.

### 2. Hierarquia da UI

```
Canvas
├── StartMenu              ← painel com os botões do menu
│   ├── Botao_Mochila
│   ├── Botao_Party
│   ├── Botao_Pokedex
│   ├── Botao_Configuracoes
│   ├── Botao_TrainerCard
│   ├── Botao_Save         → Menu.AbrirSlotsDeSave
│   └── Botao_Sair
├── MochilaMenu            ← + script MochilaMenu
├── PartyMenu              ← + script PartyMenu
├── PokedexMenu            ← + script PokedexMenu
├── ConfiguracoesMenu      ← + script ConfiguracoesMenu
├── TrainerCardMenu        ← + script TrainerCardMenu
├── SlotsDeSaveMenu        ← + script SlotsDeSaveMenu
├── CaixaMenu              ← + script CaixaMenu      (aberto pelo PC, no mundo)
└── LojaMenu               ← + script LojaMenu       (aberto pelo Vendedor)

GameManager
└── Menu                   ← + script Menu
```

Na prática isso tudo já está montado na `Bootstrap.unity` — a hierarquia acima é para
entender o que está lá, ou para remontar do zero.

### 3. Posições de animação

Cada painel precisa de dois `Transform` vazios: `posicaoAberta` (onde ele fica
visível) e `posicaoFechada` (fora da tela). O painel desliza de um para o outro.

Se você deixar os dois campos vazios, o painel só faz fade — sem slide.

### 4. Ligando os botões

No `OnClick` de cada botão do StartMenu, arraste o GameObject do `Menu` e escolha:

| Botão | Método |
|---|---|
| Mochila | `Menu.AbrirMochila` |
| Party | `Menu.AbrirParty` |
| Pokédex | `Menu.AbrirPokedex` |
| Configurações | `Menu.AbrirConfiguracoes` |
| Cartão | `Menu.AbrirTrainerCard` |
| Salvar / Carregar | `Menu.AbrirSlotsDeSave` |
| Caixa | `Menu.AbrirCaixa` |
| Sair | `Menu.Sair` |

`Menu.Salvar` **não existe mais**: salvar deixou de ser um botão que grava na hora e
virou uma tela de três slots (`SlotsDeSaveMenu`), porque escolher qual slot sobrescrever
é decisão do jogador. Há ainda duas portas abertas *pelo mundo*, não pelo menu:
`AbrirCaixaDoMundo` (o PC) e `AbrirLojaDoMundo(sortimento)` (o vendedor).

No botão **"voltar"** dentro de cada painel, use `Menu.VoltarAoMenu`.
Para a tecla que abre o menu, chame `Menu.AlternarMenu()`.

### 5. Prefabs de lista

`MochilaMenu` e `PartyMenu` instanciam um prefab por item. O prefab precisa de:

- um `Button` na raiz,
- um `TMP_Text` em qualquer lugar dentro dele (recebe o nome),
- opcionalmente um `Image` (recebe o ícone/sprite).

---

## Comportamento

**`AlternarMenu()`** funciona em camadas:

1. Submenu aberto → fecha o submenu, volta pro menu principal.
2. Menu principal aberto → fecha tudo, jogador volta a andar.
3. Tudo fechado → abre o menu principal, jogador para de andar.

O `PlayerMovement.menuFechado` é controlado automaticamente — o jogador só anda
quando **tudo** está fechado.

As animações usam `Time.unscaledDeltaTime`, então continuam funcionando se você
pausar o jogo com `Time.timeScale = 0`.

---

## Como acrescentar um painel

1. Script novo herdando `PainelMenu`, no mesmo diretório.
2. GameObject filho do Canvas, com o script e os dois `Transform` de posição.
3. Campo `[SerializeField]` no `Menu` e um `public void AbrirX() => AbrirPainel(x);`
   ao lado dos outros.
4. Botão no StartMenu apontando para esse método, e o "voltar" para `Menu.VoltarAoMenu`.

Nada mais: o `Menu` não precisa saber o que o painel faz, e o painel não precisa saber
que o `Menu` existe.

> **As três "extensões" que este documento listava foram todas feitas** — e ficam aqui
> como registro de que a doc envelheceu sem ninguém notar, que é o risco desta página:
> `Menu.Salvar()` virou `SistemaSave` com três slots, versão de formato e gravação
> atômica; `MochilaMenu.UsarItemSelecionado()` deixou de ter `Debug.Log` no lugar do
> efeito e passa pelo `SistemaInventario`; e `PartyMenu` ganhou seleção de alvo, status
> colorido e estágios de atributo.
