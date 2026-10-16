# Animações 3D — organização preliminar

| | |
|---|---|
| **O que este documento responde** | Onde a animação vai morar, e as duas regras que evitam retrabalho quando ela entrar |
| **Quando abrir** | Antes de importar o primeiro clipe, ou de encostar em `Animator` |
| **Estado** | **Só a locomoção do jogador existe** (`AnimadorDoJogador`, clipes do Mixamo em `Assets/Omega/Anims`, controller `PlayerHumanoide`; menus `Omega → Animacoes` e `Mixamo - exportar personagem`). **Zero** para Pokémon, NPC e batalha. As pastas de `Assets/Art`, `Assets/Animations` e `Assets/Prefabs` abaixo seguem vazias (só um `LEIAME.md` em cada) |
| **Atualizado em** | 09/10/2026 (cabeçalho e contagem de eventos); o corpo é de 03 a 05/09 |

Estrutura criada como **preparação**, antes de existir qualquer animação: é o lugar
combinado e as duas regras que já valem.

**A recomendação principal inverteu de sinal** — ver o item 3 no fim. O texto antigo
mandava adiar a animação até separar simulação de apresentação; essa separação está
feita, e o caminho agora é ligar a animação ao `ReprodutorDeBatalha`.

## Onde cada coisa mora

```
Assets/
├── Art/
│   ├── Creatures/     modelos e texturas das criaturas
│   ├── Characters/    jogador e NPCs
│   ├── Environment/   cenário
│   └── VFX/           partículas e efeitos visuais
│
├── Animations/
│   ├── Creatures/     clipes + Animator Controllers das criaturas
│   ├── Characters/    jogador e NPCs
│   └── UI/            transições de tela, feedbacks de menu
│
└── Prefabs/
    ├── Creatures/     prefab por criatura (modelo + Animator + colisor)
    ├── Characters/
    ├── Gameplay/      coletável, ponto de transição, área de encontro
    └── UI/
```

O pacote `AurynSky` continua onde está: é arte de terceiro e, pelo item 41 do Módulo,
deve ficar identificável como dependência, não misturada com a arte própria.

## Duas regras que valem desde já

### 1. Animação não decide regra de jogo (Módulo, item 24)

O dano, o acerto, o status e o fim do turno são resolvidos por `SistemaBatalha` e
`CalculoDano`. A animação **reage** ao que já foi decidido — nunca o contrário. Na
prática: nada de `AnimationEvent` aplicando dano, e nada de esperar a animação terminar
para saber quem ganhou.

O projeto já tem por onde reagir, sem acoplar nada: os eventos públicos do `SistemaBatalha` e o enum
`EventoBatalha` com **27** momentos (entrou em campo, usou golpe, causou dano, recebeu dano,
status aplicado, desmaiou...). Animação nova = mais um ouvinte, não uma mudança em
`SistemaBatalha`.

### 2. Ligar a animação a evento tipado, nunca ao texto narrado

`SistemaBatalha.OnMensagem` carrega uma `string` de narração. Pendurar animação nela
faria "mudar uma fala" quebrar a animação. Se faltar um momento específico (impacto,
crítico, entrada em campo), o certo é **adicionar um evento tipado**, não interpretar a
frase. Vale o mesmo que já foi registrado para áudio.

## O que precisa ser decidido antes de aprofundar

1. **Modelo por espécie**: hoje `EspecieData` só tem `Sprite`. Animação 3D vai precisar
   de uma referência a prefab/modelo. É um campo novo no dado — pequeno, mas é decisão de
   conteúdo (uma referência direta? por `Addressables`, quando houver volume?).
2. **Prefab por criatura**: criatura animada praticamente exige prefab (modelo +
   Animator + colisor no mesmo lugar). O projeto tem prefabs próprios do jogador (`Assets/Prefabs/Bootstrap/`) e dos NPCs (`Assets/Prefabs/Personagens/`, 44 corpos), mas ainda não há nenhum de criatura — e é aqui
   que a dívida de prefabs começa a doer de verdade.
3. ~~**Ritmo da batalha**~~ — **resolvido em 03/09, e a recomendação inverteu.**

   O texto anterior dizia que a luta avançava com `WaitForSeconds` dentro das próprias
   regras e mandava resolver a "etapa 2" **antes** de investir em animação. Esse
   bloqueio não existe mais: era **um** `WaitForSeconds`, em `SistemaBatalha.Fala()`, e
   ele virou `IRitmoDeBatalha` injetável — tempo real no jogo, instantâneo em servidor,
   replay ou teste.

   Mais que isso: `RegistroDaBatalha` grava o que aconteceu como lista de
   acontecimentos tipados, e `ReprodutorDeBatalha` reproduz essa lista **no ritmo de
   quem assiste**. Ou seja, a peça que a animação precisava — mandar no tempo sem mexer
   na regra — já está pronta e coberta por teste.

   O caminho recomendado agora é o oposto do que estava escrito aqui: ligar a animação
   ao `ReprodutorDeBatalha`, não ao `SistemaBatalha`. Quem desenha consome
   acontecimento; quem decide não espera ninguém.

Ver [AUDITORIA_ORGANIZACAO.md](../Historico/AUDITORIA_ORGANIZACAO.md) — seções 2 (etapas de
multiplayer) e 4 (itens 8, 24 e 25 do Módulo).
