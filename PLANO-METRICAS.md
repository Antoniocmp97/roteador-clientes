# Plano — Métricas e Relatórios

> Escrito em 28/09/2026, a partir das conversas de 27 e 28/09.
> **Status: plano aprovado em estrutura, nenhuma linha de código escrita ainda.**
>
> Este documento existe porque a sessão de chat seria compactada e o plano
> precisa sobreviver a isso, à troca de máquina e à próxima sessão. Ele é a
> continuação natural do `CLAUDE.md` — o que está aqui ainda **não** está lá,
> porque ainda não aconteceu.

---

## 0. De onde isto veio

O usuário usou o app num dia real de trabalho (28/09/2026) e trouxe três
relatos. Dois viraram a v8.23.1/v8.23.2. O terceiro abriu esta fase:

> *"Consegui atender todas as paradas, sei que não será a rotina de todos.
> Agora a parte do banco de dados se torna importante, para maiores métricas e
> relatórios."*

E logo depois, a informação que mudou o desenho inteiro:

> *"Outros dois técnicos vão utilizar, e sei que não vão ter a mesma dedicação.
> Eles preenchem a quilometragem de saída e de chegada."*

---

## 1. O marco — o ponto de retorno

Antes de qualquer coisa desta fase, o estado atual foi marcado:

- **Tag `v8.23.2`** no commit `e789131`, anotada
- **Ponto de restauração completo** em
  `backups/MARCO_v8.23.2_2026-09-28_antes-das-metricas/` — os **10 arquivos
  versionados**, e não só o `index.html`, porque esta fase pode acrescentar
  arquivos novos e um backup do index sozinho não bastaria para voltar

⚠️ Conferir se a tag foi **empurrada** para o GitHub. Tag local não sobrevive a
troca de máquina, e a pasta `backups/` também não é versionada.

---

## 2. O que já foi medido — não precisa remedir

Estes números vieram do app rodando e sustentam o plano inteiro.

### O plano do dia já é gravado, e apagado todo dia

`salvarDia()` escreve em `hg_dia_planejado` um retrato **completo**:
data, técnico ativo, e para cada técnico: nome, cor, retorno ligado, origem e
a lista de paradas com `id`, `name`, `lat`, `lng`, `cliente`, `tipo` e `prio`.

⚠️ **Chave única, sobrescrita no dia seguinte.** Todo dia o app escreve o
relatório de ontem por cima.

⚠️ **Falta ali o km e o tempo previstos.** O `ultimoResumo` (dois números) não
entra. A decisão de não guardar a rota traçada foi por causa da *geometria*,
que é grande e envelhece — o argumento não vale para o resumo.

**Tamanho medido:** um dia com 8 paradas ocupa **1171 bytes**. A 250 dias úteis
dá **~286 KB/ano**, contra um teto típico de **5 MB** do navegador. Cabe por
anos, mas o histórico deve nascer com regra de rotação mesmo assim.

### A execução é cega

`salvarProgresso()` grava `JSON.stringify([...feitos])` em
`hg_prog_r_<rid>` — **só quais** paradas foram concluídas, por coordenada.

⚠️ **Sem hora.** E a hora de cada conclusão é o campo mais valioso para
relatório. Gravar é praticamente uma linha.

⚠️ **Armadilha conhecida na mudança:** o formato atual é um array. Virando um
mapa `{chave: timestamp}`, a leitura precisa entender **as duas formas**, senão
quem estiver com roteiro aberto perde o que já marcou. O projeto já fez
exatamente essa conversão uma vez, na v6.8.0 (índices → coordenadas).

### A primeira medição de campo do km

28/09/2026: o app previu **27 km**, o usuário rodou **31 km** — **+15%**.
Causas somadas: manobras para estacionar (apontado por ele), caminho real
diferindo do calculado (medido em 19/09 entre Waze e OSRM) e o próprio
odômetro, que em carro de série lê 2 a 5% alto.

⚠️ **Um ponto não faz média.** Se repetir perto de 15% por semanas, aí é fator
sistemático e cabe ajuste configurável — e só se o número for usado para
combustível ou reembolso.

### O link e o trajeto (assunto vizinho, mesma peça de servidor)

Roteiro de 8 paradas, 106,7 km: link de **1533 caracteres**, dos quais o
**trajeto desenhado é 68%** (979 de 1439 do texto cru). Sem ele o link cai para
**495 caracteres**, e a linha pode ser **recalculada** no celular a partir das
coordenadas — o caminho de fallback já existe e já é usado por link v5–v9.

---

## 3. As três fases

### FASE A — Parar a perda, sem servidor

Nenhum destes depende de decisão nenhuma. Nenhum mexe no fluxo atual. Todos são
reversíveis.

| | o que faz | onde |
|---|---|---|
| **A1** | O plano do dia vira **histórico** em vez de ser sobrescrito, e passa a guardar **km e tempo previstos** | escritório, `salvarDia()` |
| **A2** | Cada parada concluída grava a **hora** | celular, `salvarProgresso()` |
| **A3** | **Km de saída e de chegada** na tela do campo, sem bloquear nada | celular |
| **A4** | **Exportar** o histórico em CSV/JSON | escritório |

**Onde pedir os dois números do A3:** km de saída ao abrir o roteiro, antes da
primeira parada; km de chegada no momento em que o técnico marca a **última**
parada — o instante de maior atenção do dia, quando a tela já mostra o 🎉.

⚠️ **Nunca bloquear.** Se ele pular, o dia é registrado sem o km real. Registro
pela metade vale infinitamente mais que nenhum.

⚠️ **A Fase A não entrega relatório.** Ao fim dela o escritório acumula os
planos e exporta; a execução dos dois técnicos continua presa nos celulares
deles até a Fase B. O que a Fase A entrega é outra coisa:
- **para a sangria** — cada dia sem gravar é um dia que nunca vira relatório
- **constrói a fila de sincronização** que a Fase B precisaria de qualquer jeito
- quando o servidor chegar, **semanas de histórico sobem sozinhas** dos
  celulares, porque já estarão gravadas lá

### FASE B — Juntar as duas metades

| | o que faz |
|---|---|
| **B1** | Escolher o backend — **decidido: Cloudflare Workers + D1** (seção 4) |
| **B2** | Escrever a função: recebe a execução, guarda, devolve |
| **B3** | O celular **sincroniza sozinho**, com fila e repetição — rua sem sinal existe |
| **B4** | O escritório lê o que os técnicos executaram |

⚠️ **É aqui que o ADR-01 é revisado** — dado de cliente passa a sair da
máquina. É decisão fundadora; tem de ser tomada e escrita, não acontecer de
lado.

⚠️ **E a decisão 5 do `CLAUDE.md` cai junto**: o repositório deixa de ser um
arquivo só.

### FASE C — O que o usuário quer de verdade

| | o que faz |
|---|---|
| **C1** | Relatório: previsto × real por dia e **por técnico**, taxa de conclusão, tempos |
| **C2** | **O laço fecha**: a duração real das visitas volta para o planejamento, e o app passa a dizer *"este dia não cabe"* antes de o técnico sair |

O **C2** é o prêmio. Hoje a otimização é **puramente geográfica** — está nas
limitações do `CLAUDE.md` que ela não trata duração de visita. Com o tempo real
acumulado, o planejamento aprende com a execução. É o ataque direto ao
*"não será a rotina de todos"*.

---

## 4. A escolha do backend — Cloudflare Workers + D1

> Registrado a pedido do usuário em 28/09/2026, relendo a conversa:
> *"Estava relendo e acho que faz muito sentido."*

**O que eu escolheria: Cloudflare Workers + D1** — e por um motivo que não
aparece numa tabela de comparação: **é a mesma peça que resolve outras duas
coisas que já foram levantadas.**

1. **O link curto** com o roteiro cifrado — precisa de um guarda-volumes
   chave-valor
2. **A chave paga do geocodificador** (rua + número) — precisa de um
   intermediário que a esconda, porque o repositório é público
3. **O banco de métricas** — esta fase

São três necessidades diferentes, **uma peça só**. Se o caminho fosse Supabase,
o Worker seria necessário mesmo assim para o geocodificador.

**A favor, além disso:**
- **não hiberna** (o plano grátis do Supabase pausa projeto parado por alguns
  dias — morde depois de uma semana de férias)
- a função é **onde a chave secreta mora**, então o repositório público fica
  limpo
- D1 é SQLite: SQL de verdade, que é o que relatório quer
- camada grátis folgada para o volume real (2–3 técnicos, ~10 mil linhas/ano)

**Contra, honestamente:**
- exige escrever a função (JavaScript, pequena)
- autenticação é por conta do projeto — não vem pronta
- **é a primeira coisa no projeto que não é o `index.html`**, e isso muda a
  natureza do repositório

⚠️ Camadas gratuitas mudam. Conferir os limites atuais antes de fechar.

**Alternativas consideradas e por que não:**

| | por que não |
|---|---|
| Supabase | hiberna no plano grátis; e precisaria do Worker mesmo assim para a chave do geocodificador |
| Firebase/Firestore | NoSQL deixa relatório mais trabalhoso |
| Turso / Neon | não dá para acessar do navegador — exigem função na frente de qualquer jeito |
| PocketBase em VPS grátis | manutenção, backup e uptime passam a ser do projeto |
| Planilha do Google | grátis e o relatório já nasce em planilha, mas lento, frágil e envelhece mal |
| Nada (local + exportar) | é a Fase A; vive num navegador só |

---

## 5. As decisões que travam

1. **Aceitar que dado de cliente saia da máquina** — trava toda a Fase B.
   Revisa o **ADR-01**.
2. **Autenticação** — quem pode escrever? O mínimo é o id do roteiro mais um
   segredo no link. Precisa ser pensado antes do B2.
3. **LGPD** — são dados de clientes de terceiros num servidor de terceiro.
4. **O número é insumo ou cobrança?** Não trava nada, mas **muda como as telas
   são escritas**, e é melhor decidir antes de escrever.
   ⚠️ No momento em que o sistema mede pessoas, ele muda o comportamento delas.
   Como **insumo de planejamento** ("este dia não cabe"), a equipe colabora.
   Como cobrança, o dado piora.
5. **Quanto histórico guardar localmente** antes de rotacionar.

---

## 6. O princípio que organiza tudo

**Só pedir ao técnico o que não dá para deduzir.**

| dado | custo para o técnico |
|---|---|
| quais paradas foram feitas | **zero** — ele já toca em "concluída" |
| hora de cada conclusão | **zero** — vem junto do toque |
| início e fim do dia | **zero** — primeira e última hora |
| km previsto, ordem, cliente, tipo | **zero** — o escritório já tem |
| **km de saída e de chegada** | **dois toques no dia**, que ele **já faz no papel** |
| tempo de viagem × tempo dentro do cliente | **zero** — estimar o trajeto pelo OSRM e subtrair do intervalo entre conclusões |

⚠️ **O km não é trabalho novo: é trocar o papel.** Os dois técnicos já anotam
saída e chegada. Isso é o que torna o pedido aceitável.

⚠️ **E é a dedicação irregular que decide a arquitetura.** O "link de volta"
(sem servidor) exigiria que o técnico mandasse algo no fim do dia — disciplina
exatamente onde já se sabe que não vai haver. **Quem esquece o km vai esquecer
o link.** Com servidor, o celular manda sozinho e o técnico não faz nada além
do que já faria.

---

## 7. Ressalvas que não podem se perder

- **O relatório precisa aguentar buraco.** Se metade dos dias vier sem km, ele
  mostra o que tem e **diz quantos dias entraram na conta**. Relatório que só
  funciona com dado completo não sobrevive a equipe real.
- **Histórico local vive num navegador só.** Limpar o navegador perde; duas
  máquinas planejando dividem os dados. Por isso o A1 **nasce com exportar**
  (A4), e por isso o servidor vem no fim — mas o histórico local é o que
  garante que, quando ele chegar, já exista passado para mostrar.
- **A Fase A não é provisória.** É a primeira metade do desenho final, e é a
  única parte que **perde valor a cada dia que espera**.

---

## 8. Próximo passo combinado

Começar por **A1 e A2**, que não dependem de nenhuma decisão em aberto.

Quando isso estiver publicado e rodando, reavaliar: a essa altura já haverá
alguns dias de histórico, e a conversa sobre o servidor acontece com dado na
mesa em vez de hipótese.
