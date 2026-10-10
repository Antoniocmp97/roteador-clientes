# Plano — busca sem acento (v8.24.0) e ícone da aba (v8.24.1)

> Escrito em 08/10/2026 para ser executado em outra sessão, no modelo Sonnet.
> Tudo que precisava de decisão do usuário já foi decidido e está registrado
> aqui. **Não reabrir as decisões** — implementar.
>
> Base: `main`, commit `d12a990`, versão atual **8.23.2**.
> ⚠️ `PLANO-SERVIDOR.md` está modificado na árvore de trabalho e **não faz
> parte deste plano**. Não commitar junto, não mexer.

---

## 0. Por que estas duas coisas, e por que juntas

São dois pedidos independentes do mesmo relato do usuário (08/10/2026), feitos
depois de um dia de uso no trabalho:

1. *"Ao digitar a busca no trabalho hoje acabei percebendo uma necessidade
   básica, porém muito eficaz. O filtro é muito específico, não consegue
   localizar caso o nome tenha acento."*
2. *"Precisamos de uma logo para colocar na Aba, hoje está um globinho.
   Crie algo como HF."*

Não têm nada em comum no código — uma é uma função de comparação de texto, a
outra é uma linha no `<head>`. Vão juntas só porque foram pedidas juntas, e
por isso **viram duas versões separadas**, não uma.

---

## 1. Decisões já tomadas (não reabrir)

| Assunto | Decisão | Quem decidiu |
|---|---|---|
| Escopo da busca | **Só acento e cedilha.** Nada de "palavras em qualquer ordem", nada de ignorar hífen | usuário, 08/10 |
| Onde o ícone mora | **Dentro do `index.html`**, como data-URI no `<link rel="icon">`. Nenhum arquivo novo | usuário, 08/10 |
| Letras do ícone | ⚠️ **a pergunta morreu:** o desenho escolhido **não tem letra**. "HR" de duas letras não cabia em 16px (§3.3), e o usuário acabou preferindo um ícone sem sigla | estudo de 08/10 |
| **Desenho final** | ✅ **`rota` · "Da origem ao destino"** — a linha literal está em §3.2 | usuário, 09/10: *"Gostei do rota, Da origem ao destino"* |

⚠️ **O pedido andou: HF → HR → estudo de 14 propostas.** O usuário pediu
"HF", trocou para "HR" ao ver a primeira comparação (*"HR me pareceu bom,
continuamos com essa"*) e depois pediu um estudo de design (*"vista a camisa
de designer... talvez o H e R em forma de asfalto"*). Se em algum lugar
aparecer "HF", é resíduo da conversa.

⚠️ **A ideia do asfalto foi dele e foi levada a sério** — virou três
candidatos, um deles funcionando (`r-via`), e as duas formas que **não**
couberam em 16px ficaram na página com o motivo escrito em vez de
desaparecerem. Ver §3.3. **No fim ele escolheu outra coisa**, da família sem
letra; o estudo do asfalto fica registrado porque explica o que já foi
tentado.

---

## 2. PARTE A — busca sem acento (v8.24.0)

### 2.1 O que está errado hoje

`renderChecklist()`, no `index.html`, **linhas 3764 a 3770**:

```js
const termo = buscaAtual.trim().toLowerCase();
const gruposVisiveis = !termo ? gruposClientes : gruposClientes
  .map(g => {
    const grupoBate   = !!g.grupo && g.grupo.toLowerCase().includes(termo);
    const clienteBate = g.cliente.toLowerCase().includes(termo);
    const pontos = (grupoBate || clienteBate) ? g.pontos
                 : g.pontos.filter(pt => pt.name.toLowerCase().includes(termo));
    return pontos.length ? { cliente: g.cliente, grupo: g.grupo, pontos } : null;
  })
  .filter(Boolean);
```

`toLowerCase()` resolve maiúscula, e só. "É" minúsculo é "é", nunca "e". Então
digitar `educacao` não acha `EDUCAÇÃO`, e digitar `jose` não acha `JOSÉ`.

**São exatamente estes 4 pontos no arquivo inteiro.** Conferido com
`grep -n "toLowerCase" index.html` — não há um quinto. Nenhum outro campo do
app compara texto digitado com texto da base.

### 2.2 A correção

Uma função só, usada nos quatro lugares.

**Onde colocar:** imediatamente **acima de `function renderChecklist(){`**
(hoje linha 3744). Esse é o único consumidor.

```js
// Tira acento e cedilha, para a busca do checklist encontrar "EDUCAÇÃO" quando
// se digita "educacao" — e o contrário também, porque os dois lados passam por
// aqui (v8.24.0, relatado pelo usuário depois de um dia de uso: "o filtro é
// muito específico, não consegue localizar caso o nome tenha acento").
//
// NFD separa a letra do acento ("ç" vira "c" + cedilha solta) e a faixa
// U+0300–U+036F é exatamente a dos acentos soltos — então apagá-la deixa só a
// letra base. Cedilha entra nessa conta: "ç" sai como "c".
//
// ⚠️ Isto NÃO é o mesmo trabalho do Intl.Collator de ordenarBase(). Lá o
// acento precisa CONTAR, para "AÇUDE" cair no lugar certo do alfabeto; aqui ele
// precisa ser IGNORADO, para quem digita achar. Dois problemas opostos, e é por
// isso que o Collator não serve para a busca (ele compara strings inteiras, não
// faz "contém").
function semAcento(txt){
  return (txt || '').normalize('NFD').replace(/[̀-ͯ]/g, '').toLowerCase();
}
```

E as quatro chamadas:

```js
const termo = semAcento(buscaAtual.trim());
const gruposVisiveis = !termo ? gruposClientes : gruposClientes
  .map(g => {
    const grupoBate   = !!g.grupo && semAcento(g.grupo).includes(termo);
    const clienteBate = semAcento(g.cliente).includes(termo);
    const pontos = (grupoBate || clienteBate) ? g.pontos
                 : g.pontos.filter(pt => semAcento(pt.name).includes(termo));
    return pontos.length ? { cliente: g.cliente, grupo: g.grupo, pontos } : null;
  })
  .filter(Boolean);
```

Comentário acima do bloco (o que já existe lá, acrescentando uma frase):
que a comparação ignora acento e cedilha desde a v8.24.0.

### 2.3 Armadilhas — ler antes de escrever

⚠️ **NÃO guardar o nome normalizado dentro do objeto** (`g._busca`,
`pt._busca`) "para economizar". Parece a otimização óbvia e é o caminho mais
curto para um bug silencioso: a **parada avulsa** (v7.0.0) nasce em tempo de
execução e entra em `clientPoints` e num grupo próprio do checklist — se
nascer sem o campo, ela simplesmente **nunca mais aparece na busca**, sem erro
nenhum. Mesma família de problema de `limparAvulsasSoltas` e das comparações
por identidade que a v8.9.0 teve de trocar por `id`. Calcular na hora.

⚠️ **Medir o custo antes de supor que não tem.** `renderChecklist()` roda a
cada tecla digitada. Com a base real (39 clientes) são ~39 + ~80 chamadas a
`normalize()` por tecla. **Medir no app rodando** com
`performance.now()` em volta do bloco e relatar o número no log. Se der acima
de ~2ms, aí sim conversar sobre cache — e aí o cache tem de ser preenchido
também em `criarParadaAvulsa`, com teste.

⚠️ **Não mexer no `Intl.Collator` de `ordenarBase()`** (linha 3568). Ele está
certo: ordenação precisa do acento, busca não. Ver o comentário acima.

⚠️ **A mensagem de "nenhum resultado" continua ecoando o texto COM acento**
(`escaparHtml(buscaAtual.trim())`, linha 3776). Está certo e não se mexe: ela
mostra o que a pessoa digitou, não a versão normalizada. Se alguém "consertar"
isso, o usuário vê `"educacao"` depois de ter digitado `"educação"`.

⚠️ **Não existe realce do trecho encontrado** no checklist (conferido: nenhum
`<mark>` no arquivo). Por isso não há risco de os índices saírem do lugar.
Se um dia houver, saiba que `normalize('NFD')+strip` **preserva o
comprimento** para texto já composto (`'AÇUÃO'.length === semAcento('AÇUÃO')
.length`, medido), mas **não** para texto que já venha decomposto do uMap.

⚠️ **Modo leve (regra 2 do CLAUDE.md):** isto é JavaScript puro, sem
transição, sem animação, sem depender do mapa. Funciona igual nos dois
estados. **Testar nos dois mesmo assim** — a regra não abre exceção.

### 2.4 Como testar — e por que o `exemplo.umap` não serve

⚠️ **`Exemplos/exemplo.umap` não tem um único nome acentuado**
("LABORATORIO EXEMPLO", "PREFEITURA EXEMPLO - EDUCACAO"). Carregar ele e
"testar a busca" **não prova nada**: passa antes e depois da correção.

Fazer um arquivo de teste no **scratchpad da sessão** (não no repositório),
copiando o `exemplo.umap` e acentuando os nomes — pelo menos um com `ç`, um
com `ã`, um com `é` e um com acento na **primeira** letra (`ÓTICA`).

Casos que têm de passar depois (medidos nesta sessão com a função acima):

| digita | encontra | antes |
|---|---|---|
| `educacao` | `PREFEITURA EXEMPLO - EDUCAÇÃO` | ✗ |
| `educação` | `PREFEITURA EXEMPLO - EDUCAÇÃO` | ✓ (continua) |
| `jose` | `SÃO JOSÉ` | ✗ |
| `sao jose` | `SÃO JOSÉ` | ✗ |
| `otica` | `ÓTICA CENTRO` | ✗ |
| `açu` | `ACUDE` (sem acento no arquivo) | ✗ |

E o que **continua não achando**, de propósito (escolha do usuário em §1):

| digita | não encontra | motivo |
|---|---|---|
| `educacao exemplo` | `PREFEITURA EXEMPLO - EDUCAÇÃO` | ordem das palavras importa |
| `acuo` | `AÇUÃO` | é erro de digitação, não acento (`AÇUÃO` → `acuao`) |

Testar nos **três níveis** da cascata, porque são três comparações
diferentes: nome do **grupo** do uMap, nome da **camada** (cliente) e nome da
**unidade** (filial). Um teste que só olha a unidade deixa dois caminhos sem
cobertura.

Testar também que a **parada avulsa** (criada pelo alfinete, grupo "PARADAS
AVULSAS") continua aparecendo na busca — é o caso que o cache teria quebrado.

---

## 3. PARTE B — ícone da aba (v8.24.1)

### 3.1 O que está errado hoje

Não existe `<link rel="icon">` nenhum no `index.html` (conferido). O navegador
mostra o globo padrão e ainda pede um `/favicon.ico` que não existe.

### 3.2 A correção

Uma linha, no `<head>`, **logo abaixo do `<title>`** (hoje linha 6).

A linha é **literal — copiar exatamente, não regerar o SVG.**

### ✅ É ESTA. `rota` · "Da origem ao destino" (399 caracteres)

```html
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32'%3E%3Crect width='32' height='32' rx='7' fill='%230C1418'/%3E%3Cpath d='M9.5 24Q9.5 17 16 17T22.5 10' fill='none' stroke='%23F2A93C' stroke-width='3.8' stroke-linecap='round' stroke-linejoin='miter'/%3E%3Ccircle cx='9.5' cy='24' r='3' fill='%23F2A93C'/%3E%3Ccircle cx='22.5' cy='10' r='4' fill='%23F2A93C'/%3E%3C/svg%3E">
```

Escolhido pelo usuário em 09/10/2026, entre os 14 do estudo:
*"Gostei do rota, Da origem ao destino."*

**É um desvio deliberado de tudo que veio antes, e vale entender o que ele
significa:** é o único finalista **sem letra nenhuma**. Não diz
`hagamorfis` nem `rotas` — diz o que o app **faz**. A curva âmbar é a rota, a
bolinha pequena é a origem e a grande é o destino; e é exatamente o que o app
desenha no mapa, com as mesmas cores (âmbar = origem, rota e paradas, desde
sempre). Quem bater o olho na aba não lê uma sigla: vê um percurso.

⚠️ **O risco que eu apontei e que ele aceitou de olhos abertos:** foi o
candidato que a `COMPARACAO-ICONE.html` chama de *"o mais vago — pode ser
rota, pode ser raio"*. Em 16px a curva tem duas dobras e os pontos quase se
fundem com o traço. **Não é motivo para mexer sem ele pedir** — a decisão foi
tomada vendo o 16px real. Está registrado aqui só para que ninguém "conserte"
isso por conta própria numa sessão futura.

#### Duas variantes do MESMO desenho, se ele pedir

Trocar é substituir a linha inteira, nada mais. **O padrão é a de cima.**

**`rota-b` · traço afinado** — mesma curva, traço de 3,8 → **3,0** e pontos
de 3,0/4,0 → **3,4/4,6**. O traço mais fino faz as duas bolinhas lerem como
**paradas**, e não como engrossamento da ponta da linha. É a correção de
artesanato do desenho escolhido, sem mudar o desenho.

```html
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32'%3E%3Crect width='32' height='32' rx='7' fill='%230C1418'/%3E%3Cpath d='M9.5 24Q9.5 17 16 17T22.5 10' fill='none' stroke='%23F2A93C' stroke-width='3' stroke-linecap='round' stroke-linejoin='miter'/%3E%3Ccircle cx='9.5' cy='24' r='3.4' fill='%23F2A93C'/%3E%3Ccircle cx='22.5' cy='10' r='4.6' fill='%23F2A93C'/%3E%3C/svg%3E">
```

**`rota-f` · crachá âmbar** — a mesma rota, com as cores trocadas: crachá
âmbar e percurso escuro. **É o que tem a silhueta mais forte da família**, por
um motivo simples: numa barra com vinte abas, quem o olho encontra é a mancha
de cor, e aqui a mancha é o crachá inteiro em vez de um risco fino.

```html
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32'%3E%3Crect width='32' height='32' rx='7' fill='%23F2A93C'/%3E%3Cpath d='M9.5 24Q9.5 17 16 17T22.5 10' fill='none' stroke='%230C1418' stroke-width='3' stroke-linecap='round' stroke-linejoin='miter'/%3E%3Ccircle cx='9.5' cy='24' r='3.4' fill='%230C1418'/%3E%3Ccircle cx='22.5' cy='10' r='4.6' fill='%230C1418'/%3E%3C/svg%3E">
```

⚠️ **Os outros 13 candidatos não estão mais aqui de propósito** — a decisão
foi tomada. Se um dia precisarem, a `COMPARACAO-ICONE.html` imprime a linha
de qualquer um deles, já encodada, ao clicar no cartão.

E, acima da linha escolhida, um comentário HTML explicando as três coisas que
ninguém adivinha olhando: por que vetor, o `%23`, e por que data-URI.

### 3.3 O que o estudo de 16px descobriu (e por que o desenho é esse)

Catorze propostas em quatro famílias, todas conferidas num canvas de **16×16
de verdade**, não num desenho grande. O que saiu de lá:

⚠️ **Duas letras nunca ganham desta briga.** Num quadro de 16px o "HR" sobra
~7px por letra e o buraco do R fica em **1,5px** — fecha e vira borrão. Com
uma letra só ele vai a **3,4px**. Foi isso que matou o "HR" que o usuário
tinha pedido; e, vendo os 14 lado a lado, ele acabou saindo da família das
letras de vez. **Vale para o próximo que for desenhar um:** sigla de duas
letras em 16px é dinheiro jogado fora.

⚠️ **O "R de asfalto com faixa central" NÃO cabe em 16px, e isto é um
resultado, não uma desistência.** Era a forma mais literal da ideia do
usuário. A conta: a fita precisa de ≥4px para ler como via e a faixa central
de ≥1,5px; somadas as duas bordas da fita, sobram menos de 2px de altura para
o buraco do R, que então fecha. **Não tentar de novo sem refazer essa conta.**
O que resolveu, no estudo, foi o `r-via`: **núcleo âmbar com borda escura** —
que é como o Waze e o Google desenham uma via — sobre um crachá cor de
asfalto. Duas camadas em vez de três, e 16px comporta duas. **Não foi o
escolhido**, mas é a prova de que a ideia tinha forma viável; se um dia a
marca voltar a querer letra, começar por ele.

⚠️ **"Asfalto em perspectiva" lê como a letra A.** Testado em duas versões
(convergindo num ponto, e com o topo largo). As duas leem como um A.
Perspectiva depende de profundidade, e 16px não tem profundidade nenhuma.
**Está na página de propósito, para não ser proposto de novo.**

⚠️ **A rota em S lia como ponto de interrogação.** Foi corrigida com uma
parada cheia em cada ponta — e é esta, já corrigida, que o usuário escolheu.

Geometria do desenho escolhido (`rota`):

- Quadro `viewBox="0 0 32 32"`, para a conta "metade de 32 é 16" fechar.
- Crachá `--ink` `#0C1418`, cantos `rx=7` — os mesmos 7/32 que todos os
  candidatos usam, e que em 16px dão 3,5px de raio.
- Percurso: `M9.5 24 Q9.5 17 16 17 T22.5 10` — **duas curvas quadráticas**,
  a segunda espelhando a primeira (é o que o comando `T` faz). Traço **3,8**
  (1,9px em 16px), ponta **redonda**.
- Origem: círculo em (9.5, 24) r=**3**. Destino: (22.5, 10) r=**4**. O destino
  é maior de propósito — é o que dá sentido de direção a uma linha que, sozinha,
  não tem começo nem fim.
- ⚠️ **Ponta redonda (`stroke-linecap='round'`) não é enfeite**: com ponta
  reta a linha termina num corte seco dentro da bolinha e aparece um degrau de
  um pixel na junção.

⚠️ **Este ícone não tem texto nenhum, então a armadilha de fonte não se
aplica a ele** — mas a regra continua valendo para qualquer ícone futuro:
dentro de um favicon não existe webfont, e um `<text>` sairia com a fonte do
sistema, outra no Windows do escritório e outra no Android da equipe. Foi o
que descartou os monogramas desenhados com fonte. Mesma lição da v7.9.0.

### 3.4 Armadilhas — ler antes de escrever

⚠️ **O `#` da cor é a armadilha número um.** Num `data:` URI o `#` começa o
fragmento: deixar `#F2A93C` cru **corta o SVG no meio**, o navegador volta a
mostrar o globo e **não aparece erro nenhum no console**. Todas as linhas
acima já estão com `%23`. Se o SVG for editado à mão depois, reencodar.

⚠️ **Os atributos do SVG usam aspas simples de propósito.** É o que permite a
linha inteira morar dentro de um atributo HTML com aspas duplas sem escapar
nada. Não "arrumar" para aspas duplas.

⚠️ **Depois de publicar, o globo pode continuar aparecendo.** O navegador
guarda favicon num cache próprio, que **não cai no F5 comum**. Conferir com
Ctrl+Shift+R ou em aba anônima — e avisar isso ao usuário no relato, senão ele
testa, vê o globo e reporta como defeito.

⚠️ **O ícone NÃO acompanha o tema claro/escuro do app**, e isso é decisão. A
aba do navegador não é parte da página: ela tem o tema **do navegador**, que
não é o do app. Trocar o favicon junto com o botão de tema seria o ícone
mudando por um motivo que não tem nada a ver com onde ele aparece.

⚠️ **Numa barra de abas ESCURA o crachá quase some, e isso é esperado.** O
crachá é `#0C1418` e a aba escura do Chrome é `#35363A`: sobra praticamente
só o percurso âmbar flutuando. Fica bom — mas **não é defeito nem erro de
encode**, é o desenho. Se o usuário achar apagado demais, a resposta pronta é
a variante `rota-f` (crachá âmbar) em §3.2, que inverte as cores e vira a
mancha mais forte da barra.

⚠️ **Vale nas duas telas de graça.** Escritório e campo são o mesmo arquivo, e
o `<head>` é anterior à divisão entre elas. Nada a fazer para o campo —
**mas conferir**, abrindo um link de roteiro de verdade.

⚠️ **No iPhone, "adicionar à tela de início" não vai pegar este ícone**
(o iOS ignora SVG e data-URI no `apple-touch-icon`). Foi aceito pelo usuário
ao escolher o data-URI. **Não acrescentar um PNG por conta própria** — isso
criaria um arquivo versionado novo, que é decisão dele.

⚠️ **Modo leve:** um favicon não custa desenho nenhum e não tem transição.
Testar nos dois estados assim mesmo (regra 2).

---

## 4. Material de apoio

`COMPARACAO-ICONE.html`, na raiz do projeto — **fora do repositório**
(`COMPARACAO-*.html` está no `.gitignore`, linha 35). Catorze propostas em
quatro famílias (monograma · a letra é a estrada · sem letra · fora da
paleta), com:

- o **16px REAL** de cada uma: um canvas de 16×16 de verdade, ampliado por CSS
  com os pixels à mostra, mais o **32px** (atalho da área de trabalho e aba em
  tela de alta densidade). Um desenho grande esconde exatamente o que quebra;
- barras de aba falsas, clara e escura, no formato do Chrome, com o "hoje"
  (o globo) ao lado para comparar;
- **clicar no cartão** manda o candidato para a aba da própria página — o
  único teste que não mente — e imprime a linha do `<link>` pronta;
- um bloco de **veredito** com o raciocínio, e os dois desenhos que
  **falharam** mantidos na lista com o motivo escrito.

⚠️ **O escolhido foi o `rota`, da família 3 — e não estava entre os três que
eu recomendei no veredito**, que eram todos monogramas. O veredito ficou lá
como está, errado de propósito: ele media *legibilidade*, e o usuário decidiu
por *significado*. Vale como lição para a próxima vez em que eu for ranquear
alguma coisa.

⚠️ Acrescentar essa página à lista de "material de trabalho na pasta" da
seção 8 do `CLAUDE.md`, junto das outras `COMPARACAO-*`. A lição que ela
registra, e que vale para o próximo ícone que alguém for desenhar:
**16×16 são 256 pixels — comparar em tamanho real, com os pixels à mostra, ou
não comparar.**

## 5. Versões, documentação e backups

Duas versões, na ordem, cada uma fechada e testada antes da seguinte:

| versão | o que é | por que esse número |
|---|---|---|
| **8.24.0** | busca sem acento | muda o que o app faz → mediana (§11 do CLAUDE.md) |
| **8.24.1** | ícone da aba | não muda comportamento → leve |

⚠️ Se o usuário preferir **uma versão só** (8.24.0 para as duas), tudo bem —
mas aí é escolha dele, não do executor. **Perguntar antes**, se ele não tiver
dito.

Para **cada** uma das duas versões:

1. `const VERSAO` no `index.html` (hoje linha 2286).
2. `backups/vX.Y.Z_2026-10-08_apelido/` com o `index.html` daquela versão e um
   `LEIA-ME.txt` dizendo o que mudou e por quê. Apelidos sugeridos:
   `busca-sem-acento` e `icone-da-aba`.
3. `LOG-ALTERACOES.txt` — entrada no formato das últimas, **com os números
   medidos** (custo da busca em ms, tamanho da linha do favicon em caracteres).
4. `CLAUDE.md`:
   - §2, no item da **busca no checklist**: acrescentar que ignora acento e
     cedilha desde a v8.24.0, com o porquê e o que continua não achando;
   - §3 (identidade visual): um item novo para o ícone da aba. ⚠️ **Registrar
     que ele não tem letra**, e por quê: é a primeira peça da marca que mostra
     o que o app **faz** em vez de como ele se chama, e as cores são as mesmas
     do mapa (âmbar = origem, rota e paradas). Registrar também o que o estudo
     descartou (§3.3), senão alguém propõe o monograma de novo daqui a seis
     meses;
   - §8, lista de material de trabalho: `COMPARACAO-ICONE.html`;
   - §9, tabela de histórico: duas linhas novas (passos 119 e 120);
   - §11, "Versão atual" e a tabela de versões: duas linhas novas.
5. `MAPA-DO-CODIGO.md`: conferir se a linha do `<head>` e a de
   `renderChecklist` ainda descrevem a verdade; acrescentar `semAcento` onde
   as funções do checklist são listadas (hoje linha 79) e o favicon na
   descrição do `<head>` (hoje linha 36).

⚠️ **O `README.md` provavelmente também precisa de um toque** se ele descrever
a busca. Conferir.

---

## 6. Verificação final, antes de relatar

- [ ] As duas telas abrem **sem erro de console** — escritório e campo
      (campo por um link de roteiro de verdade, gerado na hora).
- [ ] Busca com acento funciona nos **três níveis** (grupo, cliente, unidade).
- [ ] Busca acha a **parada avulsa**.
- [ ] Busca **sem** termo continua mostrando a base inteira (caminho
      `!termo`, que não passa pela função nova).
- [ ] A mensagem de "nenhum resultado" mostra o texto **com** acento.
- [ ] Ordenação alfabética da base **não mudou** (o Collator não foi tocado).
- [ ] Ícone aparece na aba, nas duas telas, em aba clara e escura do navegador
      (conferido com Ctrl+Shift+R).
- [ ] Ícone conferido **no meio de outras abas**, não sozinho: é assim que ele
      vai ser visto, e é o único teste que diz se ele se acha na barra.
- [ ] Nenhum `/favicon.ico` 404 no painel de rede.
- [ ] Tudo testado com o **modo leve ligado e desligado** (regra 2).
- [ ] Custo da busca medido e anotado no log.

---

## 7. Regras do projeto que valem aqui

1. ⚠️ **Não commitar nem publicar.** Implementar, testar, relatar e **esperar
   o `.haga`** do usuário — ele testa antes de publicar (regra 1 do
   `CLAUDE.md`). Vale mesmo que tudo passe.
2. ⚠️ **Toda alteração precisa funcionar com o modo leve ligado** (regra 2).
3. ⚠️ `PLANO-SERVIDOR.md` está modificado e é de outra frente. **Não tocar,
   não commitar junto.**
4. ⚠️ O repositório é **público**. Nada de nome real de cliente em arquivo
   versionado — inclusive nos exemplos de busca do `CLAUDE.md` e do
   `LOG-ALTERACOES.txt`. Usar nomes fictícios.
