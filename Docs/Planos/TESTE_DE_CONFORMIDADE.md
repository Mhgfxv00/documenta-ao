# Teste de conformidade com o Showdown (D13) e batalha sem tela (D14)

| | |
|---|---|
| **O que este documento responde** | Como medir, em número, o quanto a nossa batalha bate com a do Showdown; o que precisa existir antes; em que ordem fazer |
| **Quando abrir** | Antes de começar D13 ou D14 ([`../Projeto/PENDENCIAS.md`](../Projeto/PENDENCIAS.md)), ou antes de uma leva de conteúdo de batalha que queira provar fidelidade |
| **Estado** | **Camadas A e B feitas, as duas a 100% (11/10/2026); D14 feito.** Node.js 24 LTS instalado em 10/10. Resultado da A na §6 e da B na §7 |
| **Fonte de verdade** | Showdown no commit `3065d24d` (o mesmo da auditoria e das levas) |

---

## 1. O que se mede

Duas camadas. A primeira é barata e cobre muito; a segunda pega o que a primeira não vê.

| Camada | Compara | Ferramenta do lado do Showdown | Por que é determinística |
|---|---|---|---|
| **A. Dano** | Os 16 valores possíveis de dano (as rolagens de 85% a 100%) de um golpe, para atacante, defensor, item, habilidade, clima, terreno, estágios e crítico dados | `@smogon/calc` (a calculadora oficial da comunidade, mesma fórmula do simulador) | A calculadora devolve **todas** as rolagens; não há sorteio |
| **B. Fluxo da batalha** | A sequência de acontecimentos de cenários curtos: ordem de ação, dano em HP, status aplicado, estágios, efeito de habilidade e item, desmaio, fim de turno | `pokemon-showdown` (o simulador, `BattleStream`) | Cenários **sem acaso** (golpes de 100% de precisão, efeitos garantidos), ou comparação por **faixa** quando o acaso é inevitável |

**Por que não comparar batalhas inteiras com a mesma semente:** o gerador de números do Showdown não é o nosso. A mesma semente dá rolagens diferentes, e copiar o gerador amarraria o motor ao Showdown sem ganho de regra. A camada A tira o acaso do dano; a camada B escolhe cenários onde o acaso não decide.

---

## 2. As peças

```
tools/conformidade/               (fora de Assets: não entra no build)
  package.json                    pokemon-showdown e @smogon/calc em versões fixas
  gerar-dano.js                   lê cenarios/dano/*.json, escreve esperado/dano/*.json
  gerar-fluxo.js                  lê cenarios/fluxo/*.json, roda o BattleStream, normaliza o log, escreve esperado/fluxo/*.json
  cenarios/                       os casos (versionados)
  esperado/                       o resultado do Showdown (versionado: o git diff mostra quando o upstream mudou)

Assets/Scripts/Editor/Tests/Conformidade/
  ConformidadeDeDanoTests.cs      para cada cenário: monta os dois Pokémon, roda CalculoDano com cada rolagem, compara os 16 valores
  ConformidadeDeFluxoTests.cs     para cada cenário: roda a batalha sem tela (D14), normaliza o RegistroDaBatalha, compara
  TradutorDeIds.cs                id do projeto ↔ id do Showdown (o mesmo nome normalizado da auditoria)
```

### 2.1 Formato de cenário

```json
{
  "nome": "earthquake-contra-levitate",
  "formato": "simples",
  "lados": [
    { "time": [ { "especie": "garchomp", "nivel": 50, "natureza": "jolly", "evs": [0,252,0,0,4,252], "ivs": [31,31,31,31,31,31],
                  "habilidade": "roughskin", "item": "choiceband", "golpes": ["earthquake"] } ] },
    { "time": [ { "especie": "rotomwash", "nivel": 50, "habilidade": "levitate", "golpes": ["protect"] } ] }
  ],
  "turnos": [ ["move 1", "move 1"] ],
  "comparar": ["dano", "status", "estagios", "ordem", "desmaio"]
}
```

A ficha completa (natureza, IV e EV por atributo, habilidade escolhida) é exigência do teste: sem ela o dano não é comparável. Hoje a `TreinadorData` não carrega isso (PENDENCIAS A12, "ficha rasa"); o teste monta o `Pokemon` direto, não pelo treinador.

### 2.2 Normalização

O log do Showdown é texto de protocolo (`|-damage|p2a: Rotom|120/157`, `|-status|...`, `|-boost|...`, `|faint|...`). O `gerar-fluxo.js` reduz cada linha a um acontecimento neutro: `{turno, tipo, lado, posto, valor}`. Do nosso lado, o `RegistroDaBatalha` já tem os acontecimentos tipados; o teste os reduz ao mesmo formato. Diferença de **narração** (texto) nunca é divergência; diferença de **valor** é.

### 2.3 Saída

- Cada cenário vira um caso de teste NUnit (passa ou falha com o primeiro acontecimento divergente).
- Um relatório em `Temp/conformidade.txt`: % de cenários iguais por família (dano, status, habilidade, item, golpe), e a lista de divergências com o valor dos dois lados.

---

## 3. O que precisa existir antes

| Pré-requisito | Por quê | Esforço (estimativa) |
|---|---|---|
| **Node.js LTS na máquina de quem gera os esperados** | O Showdown e a calculadora são JavaScript. Só para **gerar** os esperados; os testes do Unity leem o JSON e não precisam de Node | instalação |
| **D14: batalha sem tela em EditMode** | Hoje a batalha começa por `StartCoroutine(RotinaBatalha())`, que não roda em EditMode. Com o `RitmoInstantaneo` a rotina **não espera tempo real** (conferido 10/10); falta uma porta que devolva o `IEnumerator` para o teste avançar à mão (como o `MotorDeRoteiro`) e um jeito de responder as escolhas pelo `Registrar` | 2 a 3 dias |
| **Tradução de ids** | O nome normalizado da auditoria já casa espécie, golpe, habilidade e item; faltam natureza e as formas | 0,5 dia |
| **`CalculoDano` chamável sem `SistemaBatalha` na cena** | A camada A chama o cálculo direto; conferir se ele lê `SistemaBatalha.Instancia` (PENDENCIAS D3 lista 5 lugares) | 0,5 a 1 dia |

---

## 4. Ordem sugerida

1. **Camada A (dano), sem esperar D14.** 200 a 500 cenários gerados por script (cada golpe de dano contra 3 defensores, com e sem STAB, item, habilidade e clima). É o maior ganho por esforço: o dano é onde mais conteúdo passa. *2 dias depois do Node.*
2. **D14.** A porta da batalha sem tela. *2 a 3 dias.*
3. **Camada B (fluxo)** com cenários escritos à mão por família: prioridade e ordem, status e residual, clima e terreno, troca forçada, golpe de vários turnos, Mega/Z/Dynamax/Tera. *3 a 5 dias.*
4. **Rotina:** rodar a geração quando o commit do Showdown mudar; o `git diff` em `esperado/` mostra o que o upstream mudou e os testes mostram o que o projeto precisa acompanhar.

---

## 5. Limites conhecidos

- **Arredondamento:** o projeto arredonda dano e cura em fração do HP onde o oficial trunca (PENDENCIAS D1). O teste vai acusar isso de cara; é o comportamento esperado do teste, não um defeito dele.
- **Mecânicas desligadas por padrão** (Z, Dynamax, Tera): os cenários ligam pelo `mecanicasPermitidas` do teste.
- **Conteúdo não oficial** (habilidades e itens de fã) fica fora: não há esperado para ele.
- **Dupla:** o Showdown suporta; os cenários de dupla entram depois dos de simples.

---

## 6. Resultado (10/10/2026)

**Feito:**

- **Gerador** em `tools/conformidade/` (`npm install && npm run dano`): `@smogon/calc` 0.12.0, geração 9, nível 50. São 12.420 casos: a grade inteira (20 atacantes × 8 defensores × 36 golpes) no caso base, e 11 variações num recorte (crítico, Life Orb, Choice, Expert Belt, Adaptability, Thick Fat, sol, chuva, queimado, Ataque +2, Defesa +1). O esperado fica em `esperado/dano.json` (~5 MB), versionado.
- **Teste** `ConformidadeDeDanoTests` (EditMode): monta os dois Pokémon com natureza, IV, EV, habilidade e item; calcula pelo próprio `CalculoDano.Resolver` com um acaso controlado; compara mínimo e máximo. O clima vem de um `SistemaMundo` montado fora de cena com a tabela do prefab `Sistemas`. Relatório em `Logs/conformidade_dano.txt`. **Catraca:** o teste só falha se a % cair abaixo da linha de base, hoje 100%.
- **D14:** `SistemaBatalha.RodarRotina` deixa o teste avançar a rotina da batalha à mão; `BatalhaSemTelaTests` roda uma batalha selvagem inteira em EditMode, sem cena e sem tempo real (a classe `BatalhaSemTela` é reaproveitável pela camada B).

**Medido:**

| | Antes | Depois |
|---|---:|---:|
| Atributos (HP, Ataque…) iguais ao oficial | 100% | 100% |
| Dano (mínimo e máximo) igual ao oficial | **55,5%** | **100%** |

**O que o teste achou e foi corrigido:**

1. A fórmula era em ponto flutuante com um piso só no fim, e a rolagem era um real entre 0,85 e 1. Passou a ser em inteiros, na ordem do oficial (`CalculoDano.DanoNaOrdemDoCartucho`, ver [`../Jogo/REFERENCIA_DE_SISTEMAS.md`](../Jogo/REFERENCIA_DE_SISTEMAS.md)). Só isso levou de 55,5% para 95,4%.
2. Life Orb era ×1,3; o oficial é 5324/4096.
3. A queimadura cortava o **atributo** Ataque; o oficial corta o **dano** físico depois da efetividade. Efeito colateral corrigido: Guts com queimadura dava ×0,75 em vez de ×1,5, e Facade não ignorava o corte.
4. Thick Fat, Water Bubble, Purifying Salt e Heatproof cortavam o **dano**; o oficial corta o **atributo de ataque de quem bate**.
5. Defeito do cenário, não do motor: a calculadora "queima" atacante do tipo Fogo; o jogo não deixa. O gerador pula esse caso.

**Por fazer:** a camada B (fluxo da batalha contra o simulador), usando a `BatalhaSemTela`. Os pontos a conferir primeiro estão em [`../Jogo/CALLBACKS_DO_SHOWDOWN.md`](../Jogo/CALLBACKS_DO_SHOWDOWN.md) §7. A camada A também pode crescer: formas, golpes de poder variável e de vários acertos, mais itens e habilidades.

---

## 7. Camada B, o fluxo (11/10/2026)

**Como roda.**
- `npm run simulador` (uma vez) baixa o Showdown no commit `3065d24d` em `tools/conformidade/.showdown/` (fora do git) e o compila.
- `npm run fluxo` roda os cenários de `cenarios-fluxo.js`, `cenarios-fluxo-lote2.js` e `cenarios-fluxo-lote3.js` e escreve `esperado/fluxo.json`, versionado.
- O teste `ConformidadeDeFluxoTests` monta os mesmos times com os assets do projeto, roda a batalha sem tela com as mesmas escolhas dos dois lados e compara **o estado depois de cada turno**. Relatório em `Logs/conformidade_fluxo.txt`.

**O que se compara em cada turno:**
- quem agiu e em que ordem;
- quem está em campo;
- HP, status, estágios e itens do time;
- clima, terreno, Trick Room e armadilhas de cada lado;
- o tipo Tera;
- o vencedor.

**O acaso é travado nos dois lados.**
- Do lado do Showdown: precisão sempre acerta; toda chance abaixo de 100% falha (`randomChance` e o `random(100)` dos efeitos secundários); a variação de dano fica no mínimo ou no máximo.
- Do lado do projeto:
  - `SistemaBatalha.SorteioForcadoParaTeste` fixa o acaso;
  - `SistemaBatalha.PrecisaoSempreAcerta` desliga a rolagem de precisão;
  - `SistemaBatalha.EscolhaRoteirizadaDoAdversario` faz o adversário seguir o roteiro em vez da IA;
  - `SistemaBatalha.CatalogoParaTeste` dá o catálogo (sem ele, as regras de clima Areia e Neve não valem).
- Cada cenário roda duas vezes (rolagem mínima e máxima).

**Escopo.**
- 169 cenários, em 15 famílias: ordem, status, itens, estágios, clima e terreno, armadilhas, golpes, habilidades, poder, lados, formas (Mega e Tera), Struggle, Dynamax, Z-move e dupla.
- **Dynamax e Z-move** rodam em outra geração do Showdown (gen 8 e gen 7): cada cenário pode pedir o formato (`opcoes.formato`), e o ajuste de precisão vai no dex da própria batalha, porque gen 7 e 8 são mods. Escolhas: `move N dynamax` e `move N zmove`. O cristal Z do projeto se chama `...Held`; o teste procura os dois nomes e compara o item sem o sufixo.
- O teste libera todas as mecânicas de formato para os dois lados (`mecanicasDoTreinador`).
- **Dupla** (`opcoes.dupla`, formato gen 9 doubles): 13 cenários, 169 no total. A escolha de cada lado traz uma parte por posto, separadas por vírgula (`move 1 2, move 1`; o número depois do golpe é o alvo: 1 e 2 são os postos do outro lado, -1 e -2 os do próprio). O estado fotografado ganha o segundo ativo de cada lado e a ordem diz o posto (`p1a`, `p1b`). Nenhum cenário de dupla derruba ninguém ainda.
- **Defeitos que a dupla achou:** o golpe em área não acertava o aliado, o fator 0,75 entrava no fim do cálculo e o Intimidate baixava um adversário só. Corrigidos (ver PENDENCIAS_FECHADAS, 12/10).
- Ficam fora o que depende de outro sorteio (duração do sono, multi-hit comum, desempate de velocidade) e o que pede resposta no meio do turno (U-turn, troca forçada depois de nocaute, dupla).

**Quadro final.** No turno em que a batalha acaba, o projeto desfaz o que é só da luta (clima, armadilhas, estágios, Mega) e o Showdown mantém. Nesse turno se compara só HP, item e status.

**Achado do lote 3.** O golpe de status de quem dinamaxiza vira Max Guard (prioridade +4), mas a fila ordenava pela prioridade do golpe de antes. Corrigido em `PrioridadeDe`. Cenário em que o golpe Max derruba o alvo não serve para comparar o HP do Dynamax: no fim da luta o projeto desfaz o Dynamax e o Showdown não.

**Resultado.** A primeira rodada deu **73,5%** (a maior parte era defeito do próprio teste). Depois de acertar o harness e de corrigir o que era do motor e dos dados, chegou a **100%**. Os achados estão em [`../Jogo/BATALHA_ATE_O_OFICIAL.md`](../Jogo/BATALHA_ATE_O_OFICIAL.md) §15.

**Catraca.** `LinhaDeBase = 100`. Quem mexer em batalha e baixar a %, falha. Cenário novo que ache divergência é o ganho: corrija e deixe a linha em 100.

**Para crescer:**
- troca forçada depois de nocaute em dupla, terrenos e climas em dupla, habilidades de aliado (Plus/Minus, Friend Guard);
- mais habilidades de entrada e de saída;
- golpes de vários turnos com sorteio de duração, com a duração injetada.

---

## 8. Camada por golpe (12/10/2026)

**O que é.** Um cenário para cada golpe do Showdown (647 na variante neutra, 616 na com estado), gerado por `tools/conformidade/gerar-golpes.js` com o mesmo acaso travado da camada B. O teste do Unity (`ConformidadeDeFluxoTests.Golpes_*`) monta os mesmos dois Pokémon com os assets do projeto e compara turno a turno: HP, status, estágios, efeitos de lado, clima, terreno, item e ordem. É a prova de que o golpe **faz o que o oficial faz**, e não só de que tem os números certos (o importador único já confere os números).

| Variante | Cenário | Pega o quê |
|---|---|---|
| Neutra (`golpes.json`) | Mew com o golpe contra um Snorlax que só espera, dois turnos | O efeito básico de cada golpe |
| Com estado (`golpes-estado.json`) | O Snorlax é queimado, usa Belly Drum (HP pela metade, Ataque +6) e segura Light Clay; o golpe vem nos turnos 2 e 3 | O que depende do alvo: Hex, Venoshock, Brine, Knock Off, Stored Power, Incinerate… |

**Resultado.** Primeira rodada: 91,6% (111 divergências, ~55 golpes). Depois da Leva 27 e das peças novas: **99,8% (1291/1294) e 99,5% (1226/1232)**. As catracas estão em `LinhaDeBaseDosGolpes` (99,7) e `LinhaDeBaseDosGolpesComEstado` (99,4) e só podem subir.

**O que a varredura achou** (corrigido): efeito "em si mesmo" que estava no alvo (Aqua Step, Armor Cannon, Make It Rain, Spin Out, Torch Song, Trailblaze); Tera Blast com quedas no alvo; Electro Shot e Meteor Beam sem o +1 do preparo; Victory Dance e Shelter com os estágios errados; recuo de Steel Beam, Mind Blown e Chloroblast; Fillet Away sem custo; Belly Drum e No Retreat que podiam repetir; Flail e Reversal com escala contínua em vez da tabela; Heat Crash e Heavy Slam com poder 60 no lugar de 40; Syrup Bomb que baixava a Velocidade duas vezes; Dream Eater e Snore sem a condição; Dark Void, Aura Wheel e Hyperspace Fury para qualquer espécie; Trick Room, Magic Room e Wonder Room que não desligavam; Gravity sem proibir os golpes da bandeira; Burning Jealousy e Alluring Voice sem olhar se o alvo subiu atributo; o desempate de Photon Geyser; Heal Pulse e Floral Healing arredondando para baixo; Incinerate queimando qualquer item; Disable e Encore em quem ainda não agiu; **Explosion e familia ganhando quando os dois lados caem juntos** (o oficial dá a vitória a quem foi atingido).

**O que fica de fora, e por quê** (`FORA` no gerador): golpes de sorteio (Metronome, Present, Magnitude, Psywave, Assist, Sleep Talk, Acupressure, golpes de 2 a 5 acertos); golpes de nocaute de um acerto (a precisão deles continua sorteada no simulador); golpes que dormem (o oficial acorda na vez seguinte do Pokémon, o projeto no fim do turno perdido: perde-se o mesmo número de turnos, só o estado no fim do turno difere); Healing Wish e Lunar Dance (quem usa desmaia, e o time do cenário é de um só); Transform (o registro nomeia diferente); Fake Out, First Impression e Last Resort (o projeto não deixa *escolher* o golpe que vai falhar; o oficial deixa e ele falha).

**O que ainda diverge, de propósito ou por falta de peça:** Instruct (só mira o aliado no projeto); Knock Off no turno em que a batalha acaba (o item derrubado volta ao fim da luta: regra do projeto); a ordem dos resíduos (PENDENCIAS D14), escondida pelo item inerte do alvo; e o que a varredura não alcança, que é golpe que pede preparo próprio. Lista em [`../Projeto/PENDENCIAS.md`](../Projeto/PENDENCIAS.md) S20.
