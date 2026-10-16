# Nel Ord

Vertical slice de um RPG 3D de exploração e captura de criaturas, com batalha por turnos, feito em **Unity 6** (`6000.5.10f1`, URP) por um grupo. Terceira pessoa, mundo de Kanto jogável, batalha 1×1 e dupla com um motor de regras guiado por dados, IA com dificuldade, save em três slots. Alvo declarado: mundo grande em várias áreas e **2 a 18 jogadores em rede** (ainda não existe).

> **Onde o projeto está agora:** [`Docs/Projeto/ESTADO_DO_PROJETO.md`](Docs/Projeto/ESTADO_DO_PROJETO.md). **O que mais trava hoje é a UI** (duas pilhas, sem decisão): lá está o aviso e a ordem de ataque.

## Começar em cinco minutos


Detalhes, armadilhas e o que **não** fazer (não abrir as cenas de 89 e 74 MB, não commitar o que a Unity regravou): [`Docs/Projeto/COMO_TRABALHAR.md`](Docs/Projeto/COMO_TRABALHAR.md) §0.

## Onde ler o quê

Todo o material fica em [`Docs/`](Docs/README.md). O índice completo, "qual documento responde cada pergunta", é o [`Docs/README.md`](Docs/README.md). O essencial:

| Eu quero… | Leia |
|---|---|
| saber **como está o projeto** | [`Docs/Projeto/ESTADO_DO_PROJETO.md`](Docs/Projeto/ESTADO_DO_PROJETO.md) |
| escolher **o que fazer** | [`Docs/Projeto/PENDENCIAS.md`](Docs/Projeto/PENDENCIAS.md) (o que falta) e [`Docs/Projeto/DECISOES.md`](Docs/Projeto/DECISOES.md) (o que espera resposta) |
| **trabalhar** sem repetir tropeços (git, testes, conteúdo, save, merge) | [`Docs/Projeto/COMO_TRABALHAR.md`](Docs/Projeto/COMO_TRABALHAR.md) |
| entender **o jogo** | [`Docs/Jogo/GDD.md`](Docs/Jogo/GDD.md) |
| achar **onde mora** uma coisa no código | [`Docs/Jogo/REFERENCIA_DE_SISTEMAS.md`](Docs/Jogo/REFERENCIA_DE_SISTEMAS.md) |
| **montar um mapa, um NPC, um trecho de história** | [`Docs/Mundo/MONTAGEM_DE_MAPAS.md`](Docs/Mundo/MONTAGEM_DE_MAPAS.md) |
| saber o que **cada menu do Editor** faz | [`Docs/Projeto/FERRAMENTAS_DO_EDITOR.md`](Docs/Projeto/FERRAMENTAS_DO_EDITOR.md) |
| mexer na **UI** | [`Docs/Jogo/UI_DO_PROJETO.md`](Docs/Jogo/UI_DO_PROJETO.md) |

## As três regras que o projeto não quebra

Foram **medidas** por varredura no código, não afirmadas ([`Docs/Jogo/GDD.md`](Docs/Jogo/GDD.md) §10):

1. **UI pede, sistema decide.** Nenhum botão escreve direto em `DadosJogador` nem em `Pokemon`.
2. **`MesaDeRegras` é a porta única das regras de batalha.** Habilidade, item, golpe e efeito de campo são fontes de regra; o cálculo de dano nunca sabe que habilidade existe.
3. **`EntradaJogador` é o dono único do input.** Ninguém faz `new Inputs()` fora dele.

## O que há no repositório

```
Assets/Scripts/   o jogo (Data, Systems, World, UI, Roteiro, Interacao) + ferramentas de Editor e testes
Assets/Omega/     o outro lado do grupo: UI Toolkit, criador de personagem, visual e ferramentas de mapa
Assets/Data/      o conteúdo, em Conteudo/ (regras), Mundo/ (o que se monta) e Sistema/ (catálogo, perfis de IA)
Assets/Scenes/    Jogo/ (Bootstrap, a cena inicial), Mundo/ (o mapa), Interiores/, Desenvolvimento/, Legado/
Assets/ThirdParty/ pacotes de terceiros (fora do Git)
ArteOriginal/     fontes de arte que o Unity não importa
Docs/             toda a documentação (mapa de pastas: Docs/Projeto/ESTRUTURA_DO_PROJETO.md)
```

Documentos dentro de `Assets/` viram assets da Unity (com `.meta`): por isso a documentação mora em `Docs/`, fora de `Assets/`.
