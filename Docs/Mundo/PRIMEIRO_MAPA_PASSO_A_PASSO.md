# Seu primeiro mapa, passo a passo (para quem sabe só o básico de Unity)

| | |
|---|---|
| **Para quem** | Quem abre o Unity, sabe criar um objeto e mexer no Inspector, mas nunca montou uma cena deste jogo |
| **O que você faz** | Uma cena nova com chão, ponto de nascer, uma porta de ida e volta, um NPC que fala e uma zona de mato |
| **Depois disto** | [`MONTAGEM_DE_MAPAS.md`](MONTAGEM_DE_MAPAS.md) tem as receitas completas (treinador, loja, roteiro, MOs) |
| **Conferido em** | 13/10/2026, **contra o código** (nomes de componentes, campos e menus existem). Os passos **não foram executados no Editor** nesta data: se um passo não bater com a tela, anote em [`MONTAGEM_DE_MAPAS.md`](MONTAGEM_DE_MAPAS.md) §9 |

## Antes de começar: 4 ideias

1. **O jogo roda a partir da cena `Bootstrap`.** Ela cria o jogador, a câmera e a interface. Sua cena de mapa **não** leva jogador nem câmera. Se você der Play direto na sua cena, o Editor carrega o Bootstrap por cima sozinho.
2. **Para o jogador interagir com algo, o objeto precisa de um colisor com Is Trigger ligado** e de um componente do jogo (`Npc`, `PontoTransicao`, `ItemColetavel`…). Sem o gatilho, nada acontece e **não há erro**. É o engano mais comum.
3. **"Componente"** é um script que você pluga no objeto: selecione o objeto, clique **Add Component** no Inspector e digite o nome (ex.: `PontoTransicao`).
4. **Norte do mundo é −Z.** Não importa para este tutorial, só para não estranhar.

## Passo 1. A cena e o chão

1. **File → New Scene** (modelo *Basic*) e salve em `Assets/Scenes/` como `MeuMapa.unity`.
2. Apague a **Main Camera** (o Bootstrap traz a câmera). Pode manter a luz.
3. **GameObject → 3D Object → Cube**. Nome: `Chao`. Em **Transform**: Position (0, −0.5, 0), Scale (30, 1, 30). O cubo já vem com **Box Collider**: é ele que segura o jogador.
4. **Adicione a cena ao Build Settings**: **File → Build Profiles** (ou *Build Settings*) → botão **Add Open Scenes**. Sem isto a porta do passo 3 dá erro de cena não encontrada.

## Passo 2. Onde o jogador nasce

1. **GameObject → Create Empty**. Nome: `Spawn`. Posição (0, 1, 0).
2. **Add Component → `SpawnDaCena`**.
3. Dê **Play**. O Bootstrap entra e o jogador aparece no `Spawn`.

**Se o jogador cai no vazio:** o `Chao` não tem colisor, ou o `Spawn` está fora dele.

## Passo 3. Uma porta de ida e volta

Vamos usar duas áreas da mesma cena, para você não precisar criar uma segunda cena agora. (Com cena diferente, o campo `cenaDestino` recebe o nome da outra cena; vazio = a mesma.)

1. Crie um Cube chamado `PortaIda` em (5, 1, 0), Scale (1, 2, 1).
2. No **Box Collider** dele, marque **Is Trigger**.
3. **Add Component → `PontoTransicao`** e preencha:
   - `cenaDestino`: **deixe vazio** (mesma cena)
   - `posicaoDestino`: (−5, 1, 5)  → onde o jogador aparece
   - `aoEncostar`: ligado (atravessa ao pisar)
4. Crie `PortaVolta` em (−5, 1, 8), Scale (1, 2, 1), também **Is Trigger**, com `PontoTransicao`: `posicaoDestino` = (5, 1, −3) (**ao lado** da `PortaIda`, nunca em cima).
5. Selecione cada porta: uma **esfera laranja** (Gizmo) mostra onde o jogador vai aparecer. Ela deve estar sobre o chão e **fora** do trigger de qualquer porta.

**Teste:** Play, ande até a `PortaIda`, apareça perto da `PortaVolta`, entre nela, volte. Se ficar preso indo e voltando, o ponto de chegada está dentro de um trigger de porta.

## Passo 4. Um NPC que fala

1. Arraste um corpo de `Assets/Prefabs/Personagens/` (por exemplo `Brock`) para a cena, em (0, 0, 5). A raiz do prefab tem o colisor sólido.
2. Crie um **objeto filho** dele chamado `Interacao`: botão direito no NPC → **Create Empty**.
3. No filho: **Add Component → Box Collider**, marque **Is Trigger** e aumente o **Size** (por exemplo 3, 2, 3) para o jogador poder chegar perto.
4. No filho: **Add Component → `Npc`**. Em `falas`, adicione linhas de texto. Opcional: `falasRepetidas` (o que ele diz depois da primeira conversa) e `flagDeProgresso` (um nome qualquer, ex.: `falou_com_brock`).

**Teste:** Play, chegue perto e aperte o botão de interagir. O texto aparece na caixa de diálogo.

**Se nada acontece:** (a) o colisor não está com **Is Trigger**; (b) o colisor é pequeno demais.

## Passo 5. Uma zona de mato

1. Crie um Cube `Mato` em (10, 0.5, 10), Scale (8, 2, 8). Troque o **Box Collider** para **Is Trigger** (e, se quiser, desligue o **Mesh Renderer** para ele ficar invisível, ou deixe como marcação).
2. **Add Component → `AreaDeEncontro`**. Em `tabelaPadrao`, arraste um asset de `Assets/Data/Mundo/Encontros` (qualquer tabela existente serve para testar).
3. Deixe `encontroAoAndar` ligado. Referências `encontros` e `mundo` podem ficar vazias: o componente as procura sozinho.

**Teste:** Play, ande dentro do cubo. A cada 1,5 m andados ele sorteia um encontro, com a `chance` da tabela (padrão 0,2). Pode demorar alguns passos; se nunca acontece, confira a `chance` da tabela e se ela tem espécies.

**Mato sem `AreaDeEncontro` é só enfeite.**

## Passo 6. Antes de dizer que terminou

Com o Editor aberto, nesta ordem (menus no topo da janela):

1. **Pokemon → Validar assets de dados**
2. **Pokemon → Validar roteiros** (só se você mexeu em roteiros)
3. Olhe o **`git diff`** antes de commitar: o Unity às vezes regrava materiais e cenas sem querer. **Nunca commite `Kanto3D.unity` nem `KantoPecas.unity`.**

Se você criou assets novos (tabela, loja, treinador, item), eles precisam estar no **`CatalogoJogo`** (`Assets/Data/Sistema/CatalogoJogo.asset`): selecione-o e use **Pokemon → Popular catalogo selecionado com todos os assets**. Asset fora do catálogo **some do save em silêncio**.

## Quando algo não funciona: 5 perguntas

1. O objeto tem **Is Trigger** ligado?
2. O objeto tem o **componente** certo (`Npc`, `AreaDeEncontro`…)?
3. A cena está no **Build Settings**?
4. O ponto de chegada da porta está **no chão** e **fora** de outro trigger?
5. O Console (**Window → General → Console**) mostra algum erro vermelho?

## Glossário mínimo

| Palavra | Significa |
|---|---|
| **Cena** | Um arquivo `.unity`: um mapa ou parte dele |
| **Prefab** | Objeto pronto, reaproveitável (os corpos dos NPCs) |
| **Trigger** | Colisor que não bloqueia: só avisa que o jogador entrou |
| **Tag** | Etiqueta do objeto. O jogo não usa tags para interagir: o que vale é o componente (`Npc`, `AreaDeEncontro`…) |
| **Flag** | Texto guardado no save ("já falou com o Brock") |
| **Roteiro** | Asset com uma sequência de passos (fala, batalha, dar item…): a história |
| **Gizmo** | Desenho que só aparece no Editor, como a esfera laranja da porta |
