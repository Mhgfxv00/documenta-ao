# O que sincroniza em rede (e o que cada máquina reconstrói sozinha)

Fecha o item **R10** de [`REDE_E_MULTIPLAYER.md`](REDE_E_MULTIPLAYER.md). Escrito **antes** do primeiro `NetworkBehaviour`, para o impulso natural ("sincronizar o `transform` de tudo") não vencer. Vale para qualquer pacote de transporte (R1).

## A regra em uma frase

**Viaja a escolha e o estado que decide o jogo. Tudo que é só para os olhos cada máquina refaz a partir disso.**

## 1. Batalha (por turnos)

| Viaja pela rede | Por quê |
|---|---|
| A **ação escolhida** (`AcaoDeRede`: golpe/troca/item/fuga + modificadores Mega, Z, Dynamax, Tera + alvo) | O cliente manda escolha, nunca resultado (REDE §3.4) |
| O **veredicto** do árbitro (`Aceita`/`Recusada`/`Pendente`) | Quem decide é a autoridade (`IArbitroDeBatalha`) |
| A **semente** do sorteio e, para retomar do meio, os **consumos** | A semente reproduz as rolagens; sem os consumos só reproduz do início |
| A **foto** (`EstadoDeBatalhaSerializado`) na abertura da partida e na retomada | Equipes, postos, clima, terreno, campos, turno |
| O **registro** de acontecimentos (R5/R7), quando existir como delta | Para o cliente desenhar HP, status e mensagens sem receber a foto inteira a cada turno |

| Cada máquina reconstrói | Como |
|---|---|
| Barras de HP animadas, números de dano, textos da caixa de mensagem | A partir do registro |
| Animação de golpe, partículas, câmera, som, sprite | A partir do evento do golpe (`EfeitoDeBatalha` publica evento; R9 dará o gancho de apresentação) |
| O **cálculo** de dano, ordem de ação, resíduos | Só se for **simulação espelhada**. Hoje a decisão é: **só a autoridade calcula**; o cliente recebe o resultado pelo registro. Espelhar exige o teste de determinismo entre máquinas (REDE, nota após a tabela §2) |
| Previsões da UI (golpe Z/Max que sairia, motivo de botão indisponível, turnos de Dynamax) | `SistemaBatalha.MecanicasParaUI.cs`: leitura pura sobre o estado, sem mudar nada |

**Nunca viaja:** o resultado de um sorteio já resolvido pelo cliente (o cliente não sorteia), a IA do adversário local, qualquer `ScriptableObject` (viaja o **id**; o catálogo é o mesmo nas duas pontas, falha alta se o id não resolver).

## 2. Mundo (exploração)

| Viaja | Reconstrói-se localmente |
|---|---|
| Posição e rotação do **jogador** (e só do jogador; o resto a partir disso) | Animação de andar/correr (a partir da velocidade), sombra, poeira, passos |
| Criatura que **segue** o jogador: só qual é e (se mudar) o estado | Posição seguindo o dono, animação |
| Estado de **NPC que muda o jogo** (derrotado, item pego, flag de história) | Idle, olhar, balanço |
| Troca de cena/área (o `SistemaTransicao` é o dono único) | Fade, música, carregamento |
| Quem está na área (interesse por cena, R11) | Os modelos dos outros jogadores |

**Regra para o `transform`:** só sincroniza o que **um jogador controla**. Nada que seja consequência de outra coisa sincronizada (decoração, partícula, câmera, sombra) vira `NetworkTransform`.

## 3. Estado de criatura (o que é "estado que decide o jogo")

Está em `EstadoDeBatalhaDoPokemon` e `MemoriaDeBatalha`. **Desde 13/10 a foto leva tudo isto** (`SaveDeLuta`, R4), e a retomada devolve a *mesma* batalha (R3, provado). A lista, para quem for mexer:

- estágios de atributo, `TravaDeAcao` (golpe travado, carga), proteção em curso, HP do Substituto
- forma assumida (Mega, Transformar), tipo assumido, habilidade copiada, item desativado
- Dynamax (ativo + turnos restantes), Tera (tipo, estelar), quais mecânicas o lado já gastou
- memória de batalha: golpes revelados (a IA depende), último golpe, contadores por entrada em campo, usos de gatilho
- extras de status volátil (golpe vinculado, fator de dano)
- nível da batalha: recompensa, inventário do adversário, tentativas de fuga, perfil de IA, histórico

## 4. Teste de aceitação

`Retomada_*` em `ConformidadeDeFluxoTests.Retomada.cs`: fotografa no início da escolha de um turno, restaura numa batalha nova, continua com as mesmas ações e exige a **mesma** narração, a mesma foto a cada início de turno e o mesmo fim da batalha que a original, sobre os cenários do teste de conformidade. Quem acrescentar estado de luta novo e esquecer de pô-lo na foto derruba este teste (controle feito em 13/10: sem repor a luta dos aliados, 18% dos cenários passam).

## 5. Limites conhecidos da foto

- **Só vale no início da escolha do turno** (`NoInicioDaEscolha`): ali não existe fila do turno, proteção em curso, nem dano do turno. Foto de outro ponto: `Restaurar` recusa. Em dupla, vale o pedido do primeiro posto.
- **O perfil de IA não viaja** (é um asset sem catálogo): quem restaura o passa (`perfilDeIa`); vazio = o dos selvagens.
- **A equipe de `dados` é substituída** pela da foto; caixa e mochila ficam.
- **O acaso:** semente + consumos põem o sorteio no ponto exato (`AleatorioSemeado.AvancarAte`). Provado também com o acaso semeado de verdade (crítico, precisão e variação rolando: `Retomada_ComAcasoSemeado_…`, 281 cenários iguais) e com a foto do primeiro pedido de ação (338).
- **Terreno** viaja por id, e o `CatalogoJogo` precisa conter tudo que a foto cita (espécie e forma, golpe, item, status, efeito de campo, clima, terreno). Id que não resolve dá erro no Console e o campo fica vazio.
