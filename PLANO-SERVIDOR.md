# Plano — o servidor (Cloudflare Workers + D1)

> Escrito em 04/10/2026. É a **Fase B** do `PLANO-METRICAS.md`, detalhada a
> ponto de ser implementada direto.
>
> **Status: nada construído. Nenhuma conta criada. O app não foi tocado.**
>
> ⚠️ **Leia o `PLANO-METRICAS.md` antes deste.** Lá está o *porquê* da fase e a
> escolha do backend. Aqui está só o *como*.

---

## 0. O que isto é — e o que não é

**É:** uma função pequena na Cloudflare, com um banco SQLite ao lado, que
recebe o que foi planejado (do escritório) e o que foi executado (do celular),
e devolve isso junto para virar relatório.

**NÃO é** uma reescrita do app. O `index.html` continua sendo o app, continua
abrindo sozinho, e **continua funcionando inteiro sem o servidor**. A
sincronização é um acréscimo: se a Cloudflare sumir, o técnico não percebe.

⚠️ **Essa promessa é o eixo do projeto inteiro e não pode ser quebrada em
nenhum passo.** O roteiro viaja no link (ADR-01) e o progresso é salvo no
aparelho. O servidor é uma **cópia**, nunca a fonte.

---

## 1. O desenho, numa página

```
  ESCRITÓRIO (navegador)                   CELULAR (navegador)
  ───────────────────────                  ───────────────────
  planeja o dia                            abre o link
  traça a rota                             marca as paradas
  gera o link  ───────────────────────────► anota km de saída/chegada
       │                                         │
       │ POST /plano                             │ POST /execucao
       │ (senha do escritório)                   │ (rid + token do link)
       ▼                                         ▼
  ┌──────────────────────────────────────────────────┐
  │   WORKER  (roteador.CONTA.workers.dev)            │
  │   ~160 linhas de JavaScript                       │
  └───────────────────────┬──────────────────────────┘
                          ▼
                 ┌─────────────────┐
                 │   D1 (SQLite)   │  roteiro · parada
                 │                 │  execucao · conclusao
                 └─────────────────┘
                          │
  escritório  ◄───────────┘  GET /relatorio  (senha do escritório)
```

Quatro rotas, só isso:

| rota | quem chama | o que faz |
|---|---|---|
| `GET /saude` | qualquer um | responde "ok". Serve para testar sem nada configurado |
| `POST /plano` | escritório | grava o roteiro planejado e suas paradas |
| `POST /execucao` | celular | grava km de saída/chegada e as horas de conclusão |
| `GET /relatorio` | escritório | devolve os dias, previsto e real lado a lado |

---

## 2. B0 — O que VOCÊ faz, uma vez (~20 minutos)

⚠️ **Nada aqui toca no app nem envia dado nenhum.** É só preparar a conta.

### Passo 1 — Conta na Cloudflare
`dash.cloudflare.com/sign-up` — grátis, **não pede cartão** para Workers e D1.

### Passo 2 — Node.js
Se `node --version` não responder no terminal, instalar de `nodejs.org`
(versão LTS). É só a ferramenta de linha de comando; o app continua sem build.

### Passo 3 — Entrar pela linha de comando
```bash
npx wrangler login
```
Abre o navegador e pede autorização. Uma vez só, nesta máquina.

### Passo 4 — Criar o banco
```bash
npx wrangler d1 create roteador
```
Ele imprime um bloco com um **`database_id`**. **Guarde esse texto** — o Sonnet
vai precisar dele no `wrangler.toml`.

### Passo 5 — Escolher a senha do escritório
Pense numa senha (qualquer coisa longa). Ela **não vai para o repositório** —
será guardada como segredo na Cloudflare, no passo B3.

**Quando terminar, diga ao Sonnet:** "conta pronta, database_id é tal".

---

## 3. B1 — O banco (Sonnet)

Criar a pasta e o esquema. ⚠️ **Primeira coisa no projeto que não é o
`index.html`** — isto derruba a decisão 5 da seção 4 do `CLAUDE.md`, e isso
precisa ser escrito lá quando a fase fechar.

```
PROJETO APP LOGISTICA/
├── index.html
└── servidor/              ← novo
    ├── src/index.js
    ├── schema.sql
    ├── wrangler.toml
    └── LEIA-ME.md
```

**`servidor/schema.sql`:**

```sql
-- O roteiro PLANEJADO pelo escritório.
CREATE TABLE IF NOT EXISTS roteiro (
  rid          TEXT PRIMARY KEY,          -- o id que o app já gera
  tok          TEXT NOT NULL,             -- senha deste roteiro (ver seção 5)
  criado_em    INTEGER NOT NULL,          -- epoch ms
  data_do_dia  TEXT    NOT NULL,          -- 'AAAA-MM-DD' no fuso DE QUEM PLANEJOU
  tecnico      TEXT    NOT NULL DEFAULT '',
  origem_lat   REAL, origem_lng REAL, origem_label TEXT,
  voltar       INTEGER NOT NULL DEFAULT 0,
  km_previsto  REAL, min_previsto REAL,   -- do OSRM, o que o app já calcula
  n_paradas    INTEGER NOT NULL DEFAULT 0
);
CREATE INDEX IF NOT EXISTS idx_roteiro_dia ON roteiro(data_do_dia);

CREATE TABLE IF NOT EXISTS parada (
  rid TEXT NOT NULL, ordem INTEGER NOT NULL,   -- 1..n, na ordem traçada
  nome TEXT DEFAULT '', cliente TEXT DEFAULT '',
  lat REAL NOT NULL, lng REAL NOT NULL,
  tipo TEXT DEFAULT '', prioritaria INTEGER NOT NULL DEFAULT 0,
  PRIMARY KEY (rid, ordem)
);

-- O que ACONTECEU, mandado pelo celular.
CREATE TABLE IF NOT EXISTS execucao (
  rid TEXT PRIMARY KEY,
  km_saida REAL, km_chegada REAL,
  atualizado_em INTEGER NOT NULL
);

CREATE TABLE IF NOT EXISTS conclusao (
  rid TEXT NOT NULL,
  chave TEXT NOT NULL,              -- a MESMA chave do progresso local: a
                                    -- coordenada, ou 'retorno' para a volta
  concluido_em INTEGER NOT NULL,    -- epoch ms
  PRIMARY KEY (rid, chave)
);
```

⚠️ **Sem chave estrangeira, de propósito**: no SQLite elas só valem com
`PRAGMA foreign_keys=ON` e atrapalhariam a ordem dentro de um `batch`. O
`rid` amarra tudo e é suficiente.

⚠️ **`concluido_em` é epoch ms, não texto de data.** Criciúma é UTC−3 e
"hoje" em UTC não é "hoje" aqui. Quem sabe o fuso é o navegador, então
`data_do_dia` é calculada **no cliente** e viaja pronta.

Aplicar:
```bash
npx wrangler d1 execute roteador --file=schema.sql --remote
```

---

## 4. B2 — O Worker (Sonnet)

**`servidor/wrangler.toml`:**
```toml
name = "roteador"
main = "src/index.js"
compatibility_date = "2026-01-01"

[[d1_databases]]
binding = "DB"
database_name = "roteador"
database_id = "COLAR-O-ID-DO-PASSO-4"
```

**`servidor/src/index.js`** — o esqueleto completo. Serve como está; o Sonnet
confere os nomes dos campos contra o que o `index.html` realmente manda.

```js
// Worker do Roteador de Clientes.
// ⚠️ O app funciona inteiro SEM isto. Aqui só se GUARDA uma cópia.

const CORS = {
  'Access-Control-Allow-Origin': '*',          // o app vive em github.io
  'Access-Control-Allow-Methods': 'GET,POST,OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type,X-Senha',
  'Access-Control-Max-Age': '86400'
};

const json = (dados, status = 200) => new Response(JSON.stringify(dados), {
  status, headers: { 'Content-Type': 'application/json; charset=utf-8', ...CORS }
});

const num = v => (v === '' || v === null || v === undefined || isNaN(Number(v)))
  ? null : Number(v);

const doEscritorio = (req, env) =>
  !!env.SENHA_ESCRITORIO && req.headers.get('X-Senha') === env.SENHA_ESCRITORIO;

export default {
  async fetch(req, env) {
    // ⚠️ O preflight vem ANTES de tudo. Sem isto o navegador nem tenta o POST.
    if (req.method === 'OPTIONS') return new Response(null, { status: 204, headers: CORS });

    const url  = new URL(req.url);
    const rota = url.pathname.replace(/\/+$/, '') || '/';
    try {
      if (rota === '/saude')                             return json({ ok: true, versao: 1 });
      if (rota === '/plano'     && req.method === 'POST') return await gravarPlano(req, env);
      if (rota === '/execucao'  && req.method === 'POST') return await gravarExecucao(req, env);
      if (rota === '/relatorio' && req.method === 'GET')  return await lerRelatorio(req, url, env);
      return json({ erro: 'rota desconhecida' }, 404);
    } catch (e) {
      return json({ erro: String((e && e.message) || e) }, 500);
    }
  }
};

async function gravarPlano(req, env) {
  if (!doEscritorio(req, env)) return json({ erro: 'senha' }, 401);
  const p = await req.json();
  if (!p || !p.rid || !p.tok) return json({ erro: 'faltam rid/tok' }, 400);

  // ⚠️ O token de um roteiro NUNCA muda depois de criado. O escritório regera
  //    o link o dia inteiro (v4.1: mudar a lista desfaz a rota), e trocar o
  //    token deixaria de fora o celular que já está na rua com o link antigo.
  const ja  = await env.DB.prepare('SELECT tok FROM roteiro WHERE rid=?').bind(p.rid).first();
  const tok = ja ? ja.tok : String(p.tok);

  const cmds = [
    env.DB.prepare(`INSERT INTO roteiro
        (rid,tok,criado_em,data_do_dia,tecnico,origem_lat,origem_lng,origem_label,
         voltar,km_previsto,min_previsto,n_paradas)
      VALUES (?,?,?,?,?,?,?,?,?,?,?,?)
      ON CONFLICT(rid) DO UPDATE SET
        data_do_dia=excluded.data_do_dia, tecnico=excluded.tecnico,
        origem_lat=excluded.origem_lat,   origem_lng=excluded.origem_lng,
        origem_label=excluded.origem_label, voltar=excluded.voltar,
        km_previsto=excluded.km_previsto, min_previsto=excluded.min_previsto,
        n_paradas=excluded.n_paradas`)
      .bind(p.rid, tok, Date.now(), String(p.data_do_dia || ''), String(p.tecnico || ''),
            num(p.origem_lat), num(p.origem_lng), String(p.origem_label || ''),
            p.voltar ? 1 : 0, num(p.km_previsto), num(p.min_previsto),
            (p.paradas || []).length),
    // A lista muda ao longo do dia: apaga e regrava, em vez de reconciliar.
    env.DB.prepare('DELETE FROM parada WHERE rid=?').bind(p.rid)
  ];

  (p.paradas || []).forEach((s, i) => cmds.push(
    env.DB.prepare(`INSERT INTO parada (rid,ordem,nome,cliente,lat,lng,tipo,prioritaria)
                    VALUES (?,?,?,?,?,?,?,?)`)
      .bind(p.rid, i + 1, String(s.nome || ''), String(s.cliente || ''),
            Number(s.lat), Number(s.lng), String(s.tipo || ''), s.prio ? 1 : 0)));

  await env.DB.batch(cmds);          // batch é transação: ou tudo, ou nada
  return json({ ok: true, rid: p.rid, tok });
}

async function gravarExecucao(req, env) {
  const p = await req.json();
  if (!p || !p.rid || !p.tok) return json({ erro: 'faltam rid/tok' }, 400);

  const r = await env.DB.prepare('SELECT tok FROM roteiro WHERE rid=?').bind(p.rid).first();
  if (!r)              return json({ erro: 'roteiro desconhecido' }, 404);
  if (r.tok !== p.tok) return json({ erro: 'token' }, 401);

  const cmds = [
    env.DB.prepare(`INSERT INTO execucao (rid,km_saida,km_chegada,atualizado_em)
        VALUES (?,?,?,?)
        ON CONFLICT(rid) DO UPDATE SET
          km_saida      = COALESCE(excluded.km_saida,   execucao.km_saida),
          km_chegada    = COALESCE(excluded.km_chegada, execucao.km_chegada),
          atualizado_em = excluded.atualizado_em`)
      .bind(p.rid, num(p.km_saida), num(p.km_chegada), Date.now()),
    // ⚠️ O CELULAR É A FONTE: ele manda o conjunto INTEIRO, com as horas que
    //    ele guardou. Apagar e regravar é o que faz "reabri uma parada por
    //    engano" chegar aqui. E não se perde hora nenhuma, porque quem guarda
    //    o original é o aparelho.
    env.DB.prepare('DELETE FROM conclusao WHERE rid=?').bind(p.rid)
  ];

  for (const [chave, quando] of Object.entries(p.conclusoes || {}))
    cmds.push(env.DB.prepare(
      `INSERT INTO conclusao (rid,chave,concluido_em) VALUES (?,?,?)`)
      .bind(p.rid, String(chave), Number(quando) || Date.now()));

  await env.DB.batch(cmds);
  return json({ ok: true });
}

async function lerRelatorio(req, url, env) {
  if (!doEscritorio(req, env)) return json({ erro: 'senha' }, 401);
  const de  = url.searchParams.get('de')  || '0000-01-01';
  const ate = url.searchParams.get('ate') || '9999-12-31';

  const r = await env.DB.prepare(`
    SELECT r.rid, r.data_do_dia, r.tecnico, r.km_previsto, r.min_previsto, r.n_paradas,
           e.km_saida, e.km_chegada,
           (SELECT COUNT(*)          FROM conclusao c WHERE c.rid = r.rid) AS concluidas,
           (SELECT MIN(concluido_em) FROM conclusao c WHERE c.rid = r.rid) AS primeira,
           (SELECT MAX(concluido_em) FROM conclusao c WHERE c.rid = r.rid) AS ultima
      FROM roteiro r LEFT JOIN execucao e ON e.rid = r.rid
     WHERE r.data_do_dia BETWEEN ? AND ?
     ORDER BY r.data_do_dia DESC, r.tecnico`).bind(de, ate).all();

  return json({ ok: true, dias: r.results });
}
```

---

## 5. A autenticação — a parte que a tabela deixou "por sua conta"

São **dois papéis**, e nenhum deles é "uma chave no `index.html`" — isso não
existe num repositório público.

### Papel 1 · o escritório: uma senha
Guardada como **segredo na Cloudflare** (nunca no repositório) e digitada uma
vez no navegador do escritório, ficando em `localStorage` (`hg_senha_servidor`).
Vai no cabeçalho `X-Senha`. Dá direito a **escrever planos e ler tudo**.

### Papel 2 · o celular: um token por roteiro
Dá direito a escrever **a execução daquele roteiro só**. Nada mais.

⚠️ **ACHADO DO CÓDIGO (04/10/2026) — o `rid` NÃO serve como token.**
Ele nasce de `Date.now().toString(36) + Math.random().toString(36).slice(2,6)`
(linha ~6468): 8 caracteres de relógio, que são adivinháveis, e **apenas 4
aleatórios** — 1,68 milhão de combinações, varrível. ⚠️ **Isto não é falha no
app de hoje**: ali o `rid` é só uma chave de `localStorage` e não protege
nada. Vira falha no minuto em que for usado como senha. **Por isso existe o
`tok`.**

```js
function novoToken(){                       // 12 bytes = 96 bits
  const b = new Uint8Array(12);
  crypto.getRandomValues(b);
  return btoa(String.fromCharCode(...b))
         .replace(/\+/g,'-').replace(/\//g,'_').replace(/=+$/,'');   // 16 chars
}
```

⚠️ **base64url não contém os separadores do link** (`~`, `|`, `*`, `%`) —
então o token entra **sem passar pelo `escSep`**, e não há risco de partir o
link ao meio (a armadilha da v8.16.0).

⚠️ **O `tok` vive junto do `rid`**: nas variáveis do técnico (`t.rid` nas
linhas ~2548/2557/2575 ganha um `t.tok` ao lado), em `zerarIdRoteiro()`
(~6472), e no plano do dia. Sem isso, trocar de aba troca o token.

⚠️ **Quem tem o link tem o token — e isso está certo.** Quem tem o link já tem
o roteiro inteiro. O token não protege o conteúdo; protege contra **estranho
escrever execução num roteiro que não é dele**.

---

## 6. B3 — Publicar e testar, AINDA SEM O APP (Sonnet)

```bash
npx wrangler secret put SENHA_ESCRITORIO
```
```bash
npx wrangler deploy
```

Ele imprime a URL (`https://roteador.CONTA.workers.dev`). Testar as rotas por
linha de comando, **antes de encostar no `index.html`**:

| teste | esperado |
|---|---|
| `GET /saude` | `{"ok":true,...}` |
| `POST /plano` **sem** `X-Senha` | **401** |
| `POST /plano` com a senha e um roteiro de mentira | `{"ok":true,"tok":...}` |
| `POST /execucao` com `tok` **errado** | **401** |
| `POST /execucao` com o `tok` certo | `{"ok":true}` |
| `GET /relatorio` com a senha | o dia de mentira, com previsto e real |
| `GET /relatorio` **sem** a senha | **401** |

⚠️ **Os três testes de 401 são os que importam.** Um Worker que aceita
qualquer um é pior que nenhum Worker.

⚠️ Usar dados **inventados**, nunca a base real — ainda é teste.

### 🚦 PORTÃO — a decisão do ADR-01

**Tudo até aqui é reversível e não enviou dado nenhum de cliente.** Dá para
construir, publicar, testar e até desistir sem consequência.

**O passo seguinte cruza a linha**: nome e coordenada de cliente passam a sair
da máquina e ficar num servidor de terceiro.

Antes do B4, é preciso **(1)** o aceite explícito do usuário e **(2)** o
`ADR-01` reescrito no documento de arquitetura, dizendo o que mudou e por quê.
Não deixar isso acontecer de lado.

---

## 7. B4 — O escritório manda o plano (Sonnet, no `index.html`)

Ponto de enganche: **`gerarLinkDoRoteiro()`**. É por onde todo roteiro passa
antes de ir para a rua.

1. garantir o `tok` do técnico ativo (criar se não houver);
2. montar o corpo do `POST /plano` a partir de `stops`, `ultimoResumo`,
   `originLatLng` e do nome da aba;
3. `data_do_dia` calculada **no cliente**, em fuso local;
4. **enfileirar** (seção 9) em vez de enviar direto — o escritório também pode
   estar sem rede;
5. pôr o `tok` no link: **formato v11**, 12º grupo.

⚠️ **O formato v11 acrescenta UM grupo no fim e não mexe nos 11 anteriores.**
O 1º grupo (hoje `'10'`, montado na linha ~6709) passa a `'11'`. Links v5–v10
continuam abrindo, como sempre — é a mesma regra desde a v2.6. **Testar um
link antigo de verdade.**

⚠️ **Custo no tamanho do link: ~17 caracteres.** Há folga de sobra: o
`PLANO-METRICAS.md` mediu que o trajeto desenhado é **68%** do link e que
recalculá-lo no celular levaria de 1533 para 495 caracteres.

---

## 8. B5 — O celular manda a execução (Sonnet, no `index.html`)

Dois pontos de enganche, os dois já existentes:

- **`salvarProgresso(codigo, rid, feitos)`** (~linha 7015) — hoje grava
  `JSON.stringify([...feitos])`, um array **sem hora**. Com o A2 do
  `PLANO-METRICAS.md` ele vira `{chave: epoch_ms}`.
  ⚠️ **`lerProgresso()` (~7004) precisa entender AS DUAS FORMAS** (array
  antigo e objeto novo), senão quem estiver com um roteiro aberto perde o que
  já marcou. O projeto já fez essa conversão uma vez, na v6.8.0.
- **o km de saída e de chegada** (A3), guardados junto.

Depois de cada um dos dois: `enfileirar({rota:'/execucao', corpo:{...}})` e
`tentarSincronizar()`.

⚠️ **Nunca bloquear e nunca avisar de erro.** Sem rede, o toque na parada
responde exatamente como hoje. No máximo um indicador discreto de "N para
enviar". Registro pela metade vale mais que registro nenhum.

---

## 9. A fila — o coração do B4 e do B5

```js
const CHAVE_FILA = 'hg_fila_sync';
const SERVIDOR   = 'https://roteador.CONTA.workers.dev';   // NÃO é segredo

function enfileirar(item){                 // {rota, corpo, senha?}
  try{
    const f = JSON.parse(localStorage.getItem(CHAVE_FILA) || '[]');
    f.push(item);
    localStorage.setItem(CHAVE_FILA, JSON.stringify(f.slice(-200)));
  }catch(e){}
}

async function tentarSincronizar(){
  let f;
  try{ f = JSON.parse(localStorage.getItem(CHAVE_FILA) || '[]'); }catch(e){ return; }
  if (!f.length || !navigator.onLine) return;
  const sobrou = [];
  for (const item of f){
    try{
      const r = await fetch(SERVIDOR + item.rota, {
        method:'POST',
        headers: Object.assign({'Content-Type':'application/json'},
                               item.senha ? {'X-Senha': item.senha} : {}),
        body: JSON.stringify(item.corpo)
      });
      // ⚠️ 4xx = o servidor ENTENDEU e recusou. Repetir não resolve: descarta.
      //    5xx e falha de rede = tentar de novo depois.
      if (!r.ok && r.status >= 500) sobrou.push(item);
    }catch(e){ sobrou.push(item); }
  }
  try{ localStorage.setItem(CHAVE_FILA, JSON.stringify(sobrou)); }catch(e){}
}
```

Quando tentar: **na carga**, **depois de cada gravação**, no evento `online`,
e um `setInterval` lento (60 s) como rede de segurança.

⚠️ **Nada disto pode usar `safeFetchJSON()`** (~linha 5965). A mensagem de erro
daquela função manda *"baixe o arquivo e abra-o diretamente no navegador"* —
certo para o OSRM, absurdo aqui. A sincronização é **muda**.

⚠️ **Regra 2 do projeto — modo leve.** Nada aqui depende de `transitionend`,
de animação ou de o mapa estar em tela cheia, então passa por construção. Mas
**testar nos dois estados assim mesmo**, que é a regra.

⚠️ O `setInterval` não para nunca, e isso é aceitável porque ele **não faz nada
com a fila vazia** (sai na primeira linha). Registrar isso, porque a revisão de
24/09 confere "temporizador com parada garantida".

---

## 10. B6 — Ver os dias (Sonnet)

O mínimo que fecha o laço: um painel no escritório que chama
`GET /relatorio?de=&ate=` e mostra uma linha por dia/técnico —
**previsto × real**, paradas concluídas de quantas, primeira e última hora.

O relatório de verdade (médias, tendência, o "este dia não cabe") é a **Fase C**
do `PLANO-METRICAS.md`. Aqui é só provar que o dado chegou inteiro.

⚠️ **Tem de aguentar buraco.** Se metade dos dias vier sem km, mostra o que tem
e **diz quantos dias entraram na conta**. Relatório que só funciona com dado
completo não sobrevive a equipe real.

---

## 11. Segurança — a conferência que não pode falhar

**O repositório é PÚBLICO.** Antes de qualquer commit desta fase:

- [ ] `.gitignore` bloqueia **`.dev.vars`**, **`node_modules/`** e
      **`.wrangler/`** (`.dev.vars` é onde o wrangler guarda segredo local)
- [ ] a senha do escritório **não** aparece em arquivo nenhum do repositório —
      ela vive em `wrangler secret` e no `localStorage` de quem usa
- [ ] nenhum `tok` de roteiro real foi commitado (eles nascem em tempo de
      execução e não devem ser escritos em lugar nenhum do repositório)
- [ ] busca por nome de cliente real em tudo que vai ser commitado
- [ ] o `database_id` **pode** ir no repositório: sozinho não dá acesso a nada

⚠️ **A URL do Worker no `index.html` não é segredo** e pode ser commitada. Quem
a tiver ainda precisa da senha ou de um token válido.

---

## 12. Armadilhas — a lista para não cair

| ⚠️ | o que é |
|---|---|
| **CORS** | sem tratar `OPTIONS` **antes de tudo**, o navegador nem chega a tentar o POST. O erro no console fala de CORS e parece outro problema |
| **`safeFetchJSON`** | não usar aqui. A mensagem dele é sobre o OSRM |
| **o `rid` como senha** | não serve: 4 caracteres aleatórios. Ver seção 5 |
| **fuso horário** | `data_do_dia` calculada no cliente. "Hoje" em UTC não é hoje em Criciúma |
| **progresso em dois formatos** | `lerProgresso()` tem de entender o array velho e o objeto novo |
| **o token não pode mudar** | o escritório regera o link o dia inteiro; o celular já está na rua com o antigo |
| **link v11** | 12º grupo no fim; os 11 anteriores não se movem. Testar link antigo |
| **a fila com 4xx** | repetir um 4xx para sempre entope a fila. Descartar |
| **separadores** | o token é base64url e não precisa de `escSep` — mas não trocar por outra codificação sem reconferir `~`, `|`, `*`, `%` |
| **modo leve** | regra 2 do projeto: testar ligado e desligado |
| **limites grátis** | folgados para 3 técnicos, mas **conferir os números atuais** antes de fechar: eles mudam |

---

## 13. O que fica de fora, de propósito

- **Link curto** (o roteiro guardado no servidor, link virando `#s=<id>`). É a
  segunda das três razões de ter escolhido a Cloudflare, e o mesmo Worker
  serve — mas é uma mudança bem mais funda no ADR-01. **Depois.**
- **Chave do geocodificador** (rua + número). Terceira razão. O Worker vira o
  intermediário que esconde a chave paga. **Depois.**
- **Login de verdade** por pessoa. Com 3 técnicos, senha do escritório mais
  token por roteiro resolve. Rever se a equipe crescer.
- **LGPD.** São dados de clientes de terceiros num servidor de terceiro.
  Precisa de decisão, não de código.

---

## 14. Ordem recomendada

```
B0  conta e banco          ← VOCÊ, ~20 min, não toca em nada
B1  esquema                ┐
B2  Worker                 ├ Sonnet. Nenhum dado real sai da máquina.
B3  publicar e testar      ┘ Totalmente reversível.
───────────── 🚦 PORTÃO: aceitar a revisão do ADR-01 ─────────────
A1  histórico do dia       ┐ do PLANO-METRICAS.md. Fazer ANTES do B4:
A2  hora na conclusão      ├ é o registro local que a fila envia.
A3  km de saída/chegada    ┘
B4  escritório manda       ┐
B5  celular manda          ├ Sonnet.
B6  ver os dias            ┘
```

⚠️ **O A1–A3 entra no meio por um motivo prático**: o que a fila manda é
exatamente o que eles gravam localmente. Desenhar o servidor **primeiro** (que
é o que este documento fez) deixa o formato local já na forma que o servidor
quer — então sincronizar vira envio direto, sem conversão. Esse é o ganho de
ter planejado o B antes de escrever o A.

⚠️ **E o A continua valendo sozinho.** Se o portão não for aberto, A1–A3
seguem úteis: gravam o histórico na máquina e exportam. Nada se perde.
