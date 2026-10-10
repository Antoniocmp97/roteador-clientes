# Plano — o servidor (Cloudflare Workers + D1)

> **2ª versão, 06/10/2026.** A 1ª (28/09) era o caminho feliz. Esta foi pedida
> depois de o usuário ler a primeira: *"precisamos pensar mais referente ao que
> pode dar errado"*, *"ainda é desconhecido todos os riscos, pros e contras que
> atribuir um servidor possa criar"*.
>
> **Status: nada construído. Nenhuma conta criada. O app não foi tocado.**
>
> ⚠️ Ler o `PLANO-METRICAS.md` antes deste: lá está o *porquê* da fase.

---

## 0. O que mudou da 1ª para a 2ª versão

Quatro coisas, e três delas são correções de defeito meu:

1. **Apareceu um PASSO ZERO obrigatório** (seção 3). O leitor de link do app
   **recusa** qualquer versão de formato que ele não conheça, e a v11 seria
   recusada. Isso cria uma janela real de link quebrado na rua. A correção é de
   quatro linhas e tem de ser publicada **dias antes** de qualquer outra coisa.
2. **A gravação da execução tinha um bug de perda de dados** (seção 5.3): eu
   havia escrito `DELETE` + `INSERT`, e um aparelho sincronizando com lista
   vazia **apagaria** o que o outro já tinha mandado.
3. **O link curto saiu do plano** e virou decisão à parte (seção 11). Ele é
   tentador — 77 caracteres contra 1208 — mas transforma o servidor de *cópia*
   em *único caminho*, e isso quebra o eixo do projeto.
4. **O número do link agora é medido, não estimado** (seção 2), e a
   sincronização passou a ser um **interruptor** (seção 5.5).

---

## 1. O que está em jogo — a promessa que não pode quebrar

Hoje o roteiro viaja **inteiro dentro do link** e o progresso é salvo no
aparelho (ADR-01). Consequência prática: **não existe servidor para cair.** Se
a internet do escritório morrer depois de o link ser enviado, o dia acontece
igual.

Até a v7.9.2 a tela do campo dizia isso com letras: *"Roteiro recebido por
link · nada é enviado para servidor"*. A frase saiu da tela, mas a arquitetura
continua sendo essa — e, para quem vender este app a outros clientes, ela é
**argumento de venda**, não detalhe técnico.

**A regra desta fase inteira:** o servidor é uma **cópia**. Tudo continua
funcionando sem ele. Qualquer passo que viole isso sai do plano ou vira decisão
separada e explícita.

---

## 2. O link de 10 paradas — medido

Replicação fiel do codificador do `index.html` (`escSep`, `codificarLinha`,
`simplificarLinha` a 30 m, `montarRoteiro`, `codificarRoteiro`,
`comprimirParaLink`), com **rota real do OSRM** e base fictícia de 20 paradas
na região (Criciúma · Içara · Araranguá · Tubarão).

Números = **caracteres do link inteiro**, já com
`https://antoniocmp97.github.io/roteador-clientes/` e o prefixo.

| paradas | rota | **v10 hoje** `#z=` | v10 `#r=` | **v11 c/ token** `#z=` | v11 `#r=` | v11 **sem trajeto** `#z=` | sem traj. `#r=` |
|---|---|---|---|---|---|---|---|
| 5  | 31 km  | **776**  | 955  | 794  | 978  | 452 | 564 |
| 10 | 52 km  | **1208** | 1644 | **1227** | 1667 | **598** | 883 |
| 15 | 135 km | 2127 | 2999 | 2146 | 3022 | 724 | 1147 |
| 20 | 373 km | 3308 | **4692** | 3328 | 4715 | 820 | 1368 |

Link **curto** pelo servidor (`#s=rid.token`): **77 caracteres**, em qualquer
número de paradas.

**A conferência que valida o modelo:** a v8.16.0 mediu, numa rota **real** de 10
paradas e 52,8 km, 139 pontos simplificados e um link de **1186 / 1553**. A
simulação, com 51,9 km, deu **141 pontos** e **1208 / 1644** — 2% de diferença
no comprimido. O modelo é fiel; a diferença no compatível é que meus nomes
fictícios são mais longos que os reais.

### O que esses números dizem

**O token custa 19 caracteres** no link comprimido de 10 paradas (23 no
compatível). É 1,6% — irrelevante.

**O trajeto desenhado é 51% do link.** A 10 paradas: 1227 com ele, **598** sem.
Ele existe desde a v8.16.0 para o técnico ver o desenho do dia; recalculá-lo no
celular (o caminho de reserva já existe, porque link v5–v9 abre sem trajeto)
cortaria o link **pela metade** ao custo de uma chamada ao OSRM no celular.

⚠️ **O caso que preocupa é o `#r=` com muitas paradas.** A 15 paradas ele passa
de **2999** caracteres; a 20, de **4692**. E o `#r=` é justamente o formato que
vai para o **celular antigo** — o aparelho com menos capacidade recebe o link
mais longo.
⚠️ **Não afirmo que ele quebra num número exato.** O trecho depois do `#` nunca
vai ao servidor, então os limites de cabeçalho HTTP não valem, e os navegadores
atuais aguentam muito mais que isso. O que dá para afirmar é que um link de 4,7
mil caracteres é **passivo operacional**: ocupa a tela do WhatsApp, é difícil de
conferir, e qualquer aplicativo no caminho que quebre linha o corrompe.
⚠️ **E as linhas de 15 e 20 paradas não são o dia de vocês**: minhas paradas
fictícias chegam a Araranguá e Tubarão, então a rota infla. Elas servem para
mostrar **como o tamanho escala**, não para prever a operação.

---

## 3. PASSO ZERO — a correção que vem antes de tudo

⚠️ **Este é o achado mais importante desta revisão.** Sem ele, a entrada da v11
cria link quebrado na rua.

Em `lerRoteiroCompacto()` (~linha 6782):

```js
if (!['5','6','7','8','9','10'].includes(ver)) return null; // formato desconhecido
```

Um link **v11** aberto pelo app de hoje devolve `null` — a tela do campo mostra
link inválido. **Não degrada: não abre.**

E não basta tirar a lista. Logo abaixo há três outras comparações presas a
versões literais, e com `ver = '11'` todas falham:

| linha | o que faz | com v11 | o que se perde |
|---|---|---|---|
| `v8ouMais = ['8','9','10'].includes(ver)` | rid, origem, retorno | `false` | **o `rid`** → o progresso do técnico zera |
| `['9','10'].includes(ver)` | nome do técnico | `false` | a pílula do nome |
| `ver === '10'` | trajeto | `false` | o desenho no mapa |

**A correção — quatro comparações viram desigualdade numérica:**

```js
const n = Number(ver);
if (!(n >= 5)) return null;          // qualquer versão daqui para frente abre
const v8ouMais = n >= 8;
// ... n >= 9 para o técnico, n >= 10 para o trajeto
```

Com isso **todo formato futuro passa a ser aditivo de verdade**: um link v11
aberto por um app v8.23.2 mostra o roteiro inteiro, com rid, origem, retorno,
nome e trajeto — só ignora o grupo que não conhece. Que é exatamente o que se
quer, porque **a tela do campo não precisa do token**: quem usa o token é a
sincronização, e um app antigo simplesmente não sincroniza.

### Por que publicar isto ANTES, e esperar

Medido agora no site no ar: `Cache-Control: max-age=600`. O celular guarda o
app por **10 minutos**.

O cenário ruim, que é um fluxo real e documentado (*"meia hora depois surge
mais uma parada"*):

```
09:00  escritório publica a v11
09:02  técnico abre o roteiro  → guarda o app v8.23.2 em cache
09:05  escritório acrescenta uma parada e manda o link v11
09:05  técnico abre          → app de 09:02, do cache → LINK INVÁLIDO
```

Janela de 10 minutos, batendo justamente na hora em que o escritório regera o
link. A correção do passo zero fecha isso **para sempre** — mas só se ela já
estiver no cache de todo mundo **antes** do primeiro link v11.

**Então:** publicar o passo zero como versão própria, deixar **pelo menos uma
semana** (ou até os três técnicos terem aberto um roteiro), e só depois começar
o resto. É uma alteração de quatro linhas, sem efeito visível, e **não precisa
de servidor nenhum** — pode ir hoje.

---

## 4. O desenho

```
  ESCRITÓRIO                                  CELULAR
  ──────────                                  ───────
  planeja · traça · gera o link ───────────►  abre · marca · anota km
       │                                           │
       │ POST /plano                               │ POST /execucao
       │ (senha)                                   │ (rid + token do link)
       ▼                                           ▼
  ┌──────────────────────────────────────────────────────┐
  │  WORKER  roteador.CONTA.workers.dev   ·  D1 (SQLite) │
  └───────────────────────────┬──────────────────────────┘
                              │  GET /relatorio (senha)
  escritório  ◄───────────────┘
```

Rotas: `GET /saude` · `POST /plano` · `POST /execucao` · `GET /relatorio`.

### 4.1 O esquema

```sql
CREATE TABLE IF NOT EXISTS roteiro (
  rid          TEXT PRIMARY KEY,
  tok          TEXT NOT NULL,
  criado_em    INTEGER NOT NULL,       -- epoch ms, do servidor
  data_do_dia  TEXT    NOT NULL,       -- 'AAAA-MM-DD', calculada NO CLIENTE
  tecnico      TEXT    NOT NULL DEFAULT '',
  origem_lat   REAL, origem_lng REAL, origem_label TEXT,
  voltar       INTEGER NOT NULL DEFAULT 0,
  km_previsto  REAL, min_previsto REAL,
  n_paradas    INTEGER NOT NULL DEFAULT 0
);
CREATE INDEX IF NOT EXISTS idx_roteiro_dia ON roteiro(data_do_dia);

CREATE TABLE IF NOT EXISTS parada (
  rid TEXT NOT NULL, ordem INTEGER NOT NULL,
  nome TEXT DEFAULT '', cliente TEXT DEFAULT '',
  lat REAL NOT NULL, lng REAL NOT NULL,
  tipo TEXT DEFAULT '', prioritaria INTEGER NOT NULL DEFAULT 0,
  PRIMARY KEY (rid, ordem)
);

CREATE TABLE IF NOT EXISTS execucao (
  rid TEXT PRIMARY KEY,
  km_saida REAL, km_chegada REAL,
  atualizado_em INTEGER NOT NULL
);

CREATE TABLE IF NOT EXISTS conclusao (
  rid TEXT NOT NULL,
  chave TEXT NOT NULL,              -- a coordenada, ou 'retorno' — a MESMA
                                    -- chave do progresso local
  concluido_em INTEGER NOT NULL,    -- hora do CELULAR
  recebido_em  INTEGER NOT NULL,    -- hora do SERVIDOR  (ver R4)
  PRIMARY KEY (rid, chave)
);
```

⚠️ **Sem chave estrangeira**, de propósito: no SQLite elas exigem
`PRAGMA foreign_keys=ON` e atrapalhariam a ordem dentro de um `batch`.

⚠️ **`data_do_dia` é calculada no cliente.** Criciúma é UTC−3: "hoje" em UTC
não é hoje aqui, e um roteiro gerado às 22h viraria o dia seguinte.

⚠️ **`recebido_em` existe por causa do relógio do celular** (R4). Guardar as
duas horas custa 8 bytes e é a única forma de desconfiar de uma delas depois.

---

## 5. As correções em cima da 1ª versão

### 5.1 O token nunca muda
O escritório regera o link o dia inteiro. `POST /plano` lê o `tok` já gravado e
o devolve, em vez de sobrescrever — senão o celular que está na rua com o link
antigo para de ser aceito.

### 5.2 Paradas: apagar e regravar está certo
A lista muda ao longo do dia; reconciliar não traria nada. ⚠️ Uma parada
removida deixa uma `conclusao` órfã (a chave é a coordenada). **Fica**: é
histórico, e apagá-la esconderia que a visita aconteceu.

### 5.3 ⚠️ Conclusões: MESCLAR, nunca apagar — correção de bug do plano anterior
A 1ª versão fazia `DELETE FROM conclusao WHERE rid=?` e reinseria o conjunto
que o celular mandasse, com o argumento de que "o celular é a fonte". **O
argumento tem um buraco:** se o celular sincroniza com a lista **vazia** — porque
o técnico limpou o navegador, trocou de aparelho, ou abriu o link num segundo
celular — aquele `DELETE` **apaga o dia inteiro** que já estava gravado.

A gravação passa a ser:

```js
// ACRESCENTA ou atualiza; nunca apaga por omissão
INSERT INTO conclusao (rid,chave,concluido_em,recebido_em) VALUES (?,?,?,?)
  ON CONFLICT(rid,chave) DO UPDATE SET
    concluido_em = MIN(conclusao.concluido_em, excluded.concluido_em)
```

E reabrir uma parada passa a ser **explícito**, numa lista própria da carga:

```js
// p.reabertas = ['-28.67750,-49.36970', ...]
DELETE FROM conclusao WHERE rid=? AND chave=?
```

Com isso uma carga vazia não faz nada, e `MIN` guarda a **primeira** hora de
conclusão — que é a verdadeira, mesmo se a parada for reaberta e refeita.

### 5.4 Validação do km, no servidor e na tela
O erro de digitação mais provável é a casa decimal: `11234` onde o certo era
`112340`. Barato de pegar:

- `km_chegada` tem de ser **≥** `km_saida` (odômetro não anda para trás);
- a diferença dividida pelo `km_previsto` tem de cair numa faixa larga
  (sugiro **0,5 a 3,0**);
- fora da faixa, **grava igual** e marca como suspeito. ⚠️ **Não recusar**:
  recusar o número é perder o dado, e quem está na rua não vai voltar para
  corrigir.

### 5.5 ⚠️ A sincronização é um INTERRUPTOR
Não é zelo — resolve quatro problemas de uma vez:

- **R9**, a promessa de privacidade: desligada, a afirmação "nada sai da
  máquina" volta a ser **verdade literal**, e continua sendo um modo suportado
  para um cliente que exija isso;
- **o retorno atrás** fica trivial: deu errado, desliga;
- o **beta** nasce com ela ligada e a produção com ela desligada, até a decisão
  estar tomada;
- e ela fala o **idioma que o app já tem** — é o mesmo padrão do modo leve, do
  vidro e das janelas (`hg_sync`, aplicado no `<head>`).

⚠️ **Desligada, nem a fila é criada.** Não é "grava e não envia": é o app de
hoje, sem um `fetch` a mais.

---

## 6. O ambiente BETA

Pedido do usuário: *"inclua no plano uma base teste, fora do site já
hospedado"*. Está certo, e é mais necessário do que parece.

### ⚠️ A armadilha que inviabiliza o caminho óbvio

Um beta em `antoniocmp97.github.io/roteador-clientes/beta/` — ou **num segundo
repositório** — **compartilha o `localStorage` com a produção.** O
armazenamento do navegador é por **origem** (esquema + host + porta), e **não**
por caminho. Todo projeto no GitHub Pages de uma conta vive em
`antoniocmp97.github.io`, então os dois seriam a **mesma origem**.

Consequência concreta: testar o beta mexeria nas **21 chaves** reais —
`hg_base_clientes`, `hg_arranjo_modulos`, `hg_dia_planejado`, `hg_origem_padrao`.
Um teste que apaga a base guardada apaga **a base de verdade**.

Dava para contornar prefixando as chaves, mas são **26 ocorrências literais** no
arquivo e esquecer uma é justamente o caso que corrompe o dado real.

### A recomendação: Cloudflare Pages

| | |
|---|---|
| **Onde** | `roteador-beta.pages.dev` — **origem diferente**, então `localStorage` isolado por construção, sem tocar numa linha |
| **Mesma conta** | do Worker. Grátis, e é a peça que já vai existir |
| **Banco próprio** | `roteador_beta`, num Worker próprio. Dado de teste nunca encosta no de verdade |
| **Produção** | segue **exatamente** onde está: GitHub Pages, branch `main`. Nada muda |
| **Branch** | `beta`. Publicação automática a cada push |

### Sem build, sem arquivo diferente

O ambiente se descobre em tempo de execução, então **o `index.html` é o mesmo
nos dois** e o merge `beta → main` não tem conflito artificial:

```js
const EH_BETA = location.hostname.endsWith('.pages.dev')
             || location.hostname === 'localhost'
             || location.hostname === '127.0.0.1';
const SERVIDOR = EH_BETA ? 'https://roteador-beta.CONTA.workers.dev'
                         : 'https://roteador.CONTA.workers.dev';
```

⚠️ **A marcação visual vai nas DUAS telas**, inclusive na do campo: o risco
real é um link de beta chegar ao WhatsApp de um técnico e o dia ser registrado
no banco de teste sem ninguém perceber. Uma faixa que diga TESTE resolve, e no
celular ela tem de estar **acima** do primeiro cartão.

⚠️ **O `localhost` entra na conta de propósito**: o servidor local do
`.claude/launch.json` passa a apontar para o beta sozinho, então nenhum teste
feito aqui escreve no banco de verdade.

---

## 7. Registro de riscos

Ordenado por quanto dói, não por probabilidade.

| # | risco | tamanho | o que o plano faz |
|---|---|---|---|
| **R1** | **O servidor virar caminho único para o campo** | fatal | **Não acontece nesta fase.** O link continua carregando o roteiro inteiro. É o link curto que criaria isso — e ele saiu para a seção 11 |
| **R2** | Link v11 recusado por app em cache | alto, **medido** | Passo zero (seção 3), publicado com antecedência |
| **R3** | Carga vazia apagar o dia já gravado | alto | Mesclar em vez de apagar (5.3) |
| **R4** | Relógio do celular errado → hora errada | médio | Guardar `concluido_em` **e** `recebido_em` |
| **R5** | **Registrar hora de chegada e saída de empregado** | **alto, jurídico** | Ver seção 9. **Não é risco de código** |
| **R6** | O km virar dinheiro com 15% de erro | alto | Ver seção 9. Odômetro = fato; app = previsão |
| **R7** | Virar produto para vários clientes | médio | Isolamento por conta **desde o esquema**, mesmo com um cliente só |
| **R8** | Cloudflare mudar regra / suspender conta | médio | D1 é SQLite: `wrangler d1 export` dá um `.sql`. Exportação semanal + o histórico local da Fase A, que é cópia independente |
| **R9** | Perder "nada sai da máquina" como argumento | médio | O interruptor (5.5) mantém o modo antigo suportado |
| **R10** | Beta corromper dado real pelo `localStorage` | alto | Origem diferente (seção 6) |
| **R11** | Link de beta chegar a um técnico | médio | Faixa TESTE nas duas telas + banco separado |
| **R12** | Senha do escritório vazar | médio | Senha longa e aleatória, em gerenciador; troca = um `wrangler secret put`. **Nunca** em arquivo |
| **R13** | Segredo no repositório público | alto | `.gitignore` com `.dev.vars`, `node_modules/`, `.wrangler/` **antes** do primeiro commit da pasta |
| **R14** | CORS mal configurado → falha muda | baixo | `OPTIONS` na primeira linha |
| **R15** | Fila entupir com erro permanente | baixo | 4xx descarta, 5xx repete |
| **R16** | A fase parar no meio | baixo | Todo ponto de parada é estável (seção 10) |
| **R17** | O teste aqui não pegar o que falha na rua | médio | Beta + piloto só com o usuário antes dos técnicos |

### O que o servidor traz de bom, e que também é risco não ter

Para ser justo com o outro lado:

- **O progresso deixa de morrer com o navegador do técnico.** Hoje, limpar o
  navegador perde o dia. Com o servidor, dá para restaurar.
- **O escritório passa a saber o que já foi feito** sem telefonar.
- **O planejamento aprende** (Fase C): sem dado, a otimização continua
  puramente geográfica para sempre.
- **O dia morre todo dia.** Sem isto, não existe série, e sem série não existe
  nem resposta para "os 15% são sistemáticos?".

---

## 8. Cenários — o que acontece quando

| cenário | hoje | com o servidor |
|---|---|---|
| Servidor cai no meio do dia | — | nada muda na rua; a fila guarda e manda depois |
| Servidor cai às 7h, antes de sair | — | o link já tem tudo: o dia acontece igual |
| Celular sem sinal o dia inteiro | tudo salvo no aparelho | idem; sobe quando pegar sinal |
| Técnico limpa o navegador | **perde o dia** | perde na tela, mas o servidor tem; ⚠️ e **não apaga** o que já subiu (R3) |
| Mesmo link em dois aparelhos | progressos separados | mesclados; a **primeira** hora vence |
| Escritório regera o link com o técnico já na rua | progresso sobrevive (por `rid`) | idem, e o token não muda (5.1) |
| Técnico reabre parada marcada por engano | sai da tela dele | lista `reabertas` apaga no servidor |
| Esquece o km | — | o dia entra sem km. **Nunca bloquear** |
| Digita o km com casa errada | — | grava e marca suspeito (5.4) |
| Relógio do celular errado | — | `recebido_em` permite desconfiar |
| Dia atravessa a meia-noite | — | `data_do_dia` é a do planejamento, não a da conclusão |
| Não termina o roteiro | — | dia parcial; o relatório **diz** quantas de quantas |
| Rota refeita no meio do dia | rota desfeita (v4.1) | `km_previsto` é atualizado no `/plano` |
| Planeja num computador, abre noutro | nada é compartilhado | o servidor passa a ser a ponte |
| Mesmo cliente para dois técnicos | permitido, só avisa | dois `rid` distintos, sem cruzar (v8.9.0) |
| Link de beta mandado por engano | — | escreve no banco de teste; a faixa avisa |
| Conta suspensa | — | exportação semanal + histórico local |
| Cliente pede os dados, ou a exclusão | nada a entregar | `rid` é a chave; dá para extrair e apagar |
| Técnico sai da empresa | — | o histórico dele continua. ⚠️ **Decidir por quanto tempo** |

---

## 9. Para a conversa com o cliente

Isto não é código, e é a parte mais importante desta revisão.

### 9.1 A moldura que eu recomendaria: previsão × fato

| | de onde vem | o que é |
|---|---|---|
| **km do app** | OSRM, antes de sair | **previsão** — porta a porta, sem trânsito, sem manobra |
| **km do odômetro** | o carro | **fato** — já é coletado hoje, no papel |
| **a diferença** | a conta | **informação** — manobra, desvio, aderência ao roteiro |

A primeira medição de campo (28/09) deu **27 previstos contra 31 rodados,
+15%**, e o próprio usuário apontou a causa: *"precisei fazer pequenos
quadrados para estacionar"*.

⚠️ **O que eu não prometeria ao cliente:** um número de km exato. A previsão
tem 15% de erro medido **num ponto só**, e três fontes somadas (manobra,
caminho real diferente do calculado, e o próprio odômetro, que em carro de
série lê 2 a 5% alto).

⚠️ **O que dá para prometer com segurança:** comparação entre dias e entre
técnicos pelo **mesmo** critério, taxa de paradas concluídas, horário de início
e de fim, e tempo por visita. Comparação aguenta erro sistemático; valor
absoluto não.

⚠️ **Se o número for virar reembolso ou combustível**, pagar sobre o
**odômetro** (que é o fato e já existe) e usar o app para planejar. Pagar sobre
a previsão é pagar 15% errado para algum lado.

### 9.2 ⚠️ O que precisa ser verificado ANTES de prometer relatório com horários

**Não sou advogado e isto não é orientação jurídica.** Mas é um risco grande
demais para ficar fora do plano:

Um sistema que registra **hora de chegada e de saída** de empregado não é só
uma métrica de rota — ele se aproxima de **controle de jornada**, que no Brasil
tem regra própria (CLT art. 74 e a portaria de registro eletrônico de ponto).
Há diferença entre *"a visita levou 40 minutos"* e *"o técnico começou às 8h12
e parou às 17h30"*, e o segundo é o que o relatório vai mostrar sem querer.

Há ainda o lado da **LGPD**: dado de cliente é, em boa parte, dado de empresa
(nome e endereço comercial), mas **horário e deslocamento de pessoa
identificada são dados pessoais do empregado**.

**O que eu faria antes da conversa:** levantar com quem cuida de pessoal e
contabilidade (1) se o registro de horários cria obrigação de ponto, (2) quem é
o controlador do dado, e (3) o que precisa ser comunicado aos técnicos.

### 9.3 A transparência é desenho, não só ética
Se o técnico descobrir sozinho que está sendo medido, o dado piora — alguém vai
parar de marcar parada, ou vai marcar tudo no fim do dia de uma vez, e aí a
série não vale nada. **O número é insumo de planejamento ou é cobrança?** Os
dois funcionam, mas as telas são escritas diferente, e essa decisão vem antes
do código.

---

## 10. A ordem, com os pontos de parada

```
PASSO ZERO   leitor de link tolerante a versão nova      ← pode ir HOJE
             4 linhas · sem servidor · sem efeito visível
             ⏳ esperar ~1 semana de propagação
─────────────────────────────────────────────────────────────────────
A1  histórico do dia + km/tempo previstos      ┐ Fase A do PLANO-METRICAS.
A2  hora em cada conclusão (2 formatos!)       ├ Local. Nenhum dado sai.
A3  km de saída e chegada, sem bloquear        │ Útil sozinha.
A4  exportar                                   ┘
─────────────────────────────────────────────────────────────────────
BETA  Cloudflare Pages + Worker e banco de teste
B0    conta e banco                            ← você, ~20 min
B1    esquema      ┐ nenhum dado real sai da máquina.
B2    Worker       ├ Totalmente reversível.
B3    testar sem o app ┘ (os 3 testes de recusa são os que importam)
──────────── 🚦 PORTÃO: aceitar a revisão do ADR-01 ────────────────
B4    interruptor de sincronização + escritório manda  (link v11)
B5    celular manda, com fila
B6    ver os dias
──────────── piloto: 1 semana só você → 1 técnico → os três ────────
C1    relatório previsto × real
C2    a duração real volta para o planejamento
```

**Todo ponto de parada é estável**, e isso é de propósito:

- **depois do passo zero**: o app ficou mais robusto, nada mais mudou
- **depois da Fase A**: histórico local + exportação. Já serve, sem servidor
- **depois do B3**: existe um servidor que não faz nada. Nenhum dado saiu
- **depois do B5 com o interruptor desligado**: nada mudou para ninguém

---

## 11. O link curto — decisão à parte, NÃO nesta fase

77 caracteres contra 1208. É muito tentador, e o Worker já estaria lá.

⚠️ **E é exatamente o que viola a seção 1.** Com `#s=rid.token`, o roteiro
**não está mais no link**: o celular tem de buscá-lo. Servidor fora do ar às 7h
da manhã = **ninguém começa o dia**. O servidor deixa de ser cópia e passa a
ser o caminho único.

Mitigações possíveis, se um dia valer a pena:

1. **O celular guarda na primeira abertura.** A janela de risco fica só na
   primeira abertura do dia — mas ela é justamente às 7h.
2. **Os dois links.** O curto para mandar, o longo guardado como reserva. Aí o
   problema é humano: quem vai achar o longo quando precisar?
3. **Encurtar sem servidor:** tirar o trajeto do link e recalculá-lo no celular
   leva 1227 → **598** caracteres, **metade**, sem acoplar nada. ⚠️ Custo: uma
   chamada ao OSRM no celular, que pode falhar sem sinal — e aí o mapa do dia
   aparece sem a linha, que é o comportamento que link v5–v9 já tem.

**Minha recomendação:** a opção 3, **depois** da Fase B estar estável, e como
versão própria. Ela resolve metade do incômodo sem criar dependência nenhuma.

---

## 12. O que fica de fora, de propósito

- **Chave do geocodificador** (rua + número). O Worker é o intermediário que a
  esconde. **Depois** — e é a terceira razão de ter escolhido a Cloudflare.
- **Login por pessoa.** Com três técnicos, senha do escritório + token por
  roteiro resolve. Rever se a equipe crescer ou se o cliente quiser entrar.
- **Relatório para o cliente ver sozinho.** Depende de decisão de acesso.
- **Observação livre por parada** (adiada em 29/08/2026). ⚠️ Vale lembrar que,
  com servidor, ela fica barata — e é o campo que explica os 15%: *"não tinha
  vaga, dei três voltas"*.

---

## 13. O que ainda não sei

Perguntado ao usuário em 06/10/2026; as respostas mudam o plano nos pontos
marcados.

| pergunta | muda o quê |
|---|---|
| **Para que o número vai servir** — planejar, pagar, avaliar pessoas? | 9.1, 9.3, e como as telas são escritas |
| **De quem é o dado** — Hagamorfis, ou o cliente é outra empresa? | R5, R7, LGPD, isolamento por conta |
| **Onde o km de saída/chegada vai hoje** — papel, WhatsApp, planilha? | A3: alimentar o destino que já existe em vez de criar outro |
| **Quem testa o beta, e por quanto tempo** | seção 10, o piloto |
| Os técnicos já sabem, ou vão saber? | 9.3 |
| Orçamento zero é requisito duro? | R8, durabilidade da escolha |
| Quantos meses de histórico o cliente quer? | rotação, R8, e "técnico que saiu" |
| O cliente vai querer entrar e ver, ou você entrega o relatório? | seção 12 |
