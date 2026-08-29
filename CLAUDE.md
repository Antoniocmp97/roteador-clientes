# Roteador de Clientes — Hagamorfis

> Documento de contexto do projeto. Mantido atualizado a cada passo para permitir
> migração do chat para o Claude Code sem perda de contexto.
>
> **Última atualização:** 29/08/2026 — v1.9 (Fase 2: tipo de serviço por parada)

---

## 1. Objetivo

Aplicativo web que lê pontos geográficos de clientes/filiais (arquivo GeoJSON,
originalmente criado no uMap) e traça rotas de visita **dentro do próprio app**,
sem depender de serviços externos de navegação.

Contexto de uso: empresa em Criciúma, Santa Catarina (Brasil). Interface em
português do Brasil.

---

## 2. Estado atual

**Arquivo principal:** `index.html` — HTML único, sem build,
sem dependências instaladas. Abre direto no navegador.

**Status:** funcional e testado pelo usuário.

### Funcionalidades implementadas

- Upload do **backup completo do uMap (`.umap`)** por clique ou arrastar-e-soltar
- Parse das camadas: cada camada é um **cliente**, os pontos dentro dela são as
  **filiais**. Camadas vazias são ignoradas
- Checklist agrupado por cliente, com cascata que abre ao clicar no nome
- **Tipo de serviço por parada** (Fase 2): lista configurável (editável pelo
  próprio escritório, salva no navegador — ex.: "Entrega de toner",
  "Manutenção"). Cada parada selecionada ganha um seletor para escolher o
  tipo; aparece na lista de paradas do escritório e como etiqueta na tela
  do campo
- Plot dos pontos no mapa (Leaflet) com popup mostrando filial e cliente
- Definição de origem: geolocalização do navegador **ou** endereço digitado
  (geocodificação via Nominatim, com viés para Criciúma/SC)
- Checklist de seleção de quais clientes visitar na viagem
- Lista de paradas reordenável manualmente (setas ▲▼ e remoção)
- **Traçar nesta ordem** — rota respeitando a ordem escolhida (OSRM Route API)
- **Otimizar ordem** — OSRM Trip API recalcula a melhor sequência e reordena a
  lista automaticamente (`source=first`, `roundtrip=false`)
- Marcadores numerados conforme a ordem final de visita
- Distância total, tempo estimado e instruções passo a passo agrupadas por parada
- **Modo campo (Fase 1):** depois de traçar a rota, um botão gera um link com o
  roteiro inteiro. Quem abre esse link (ex.: recebido pelo WhatsApp) cai numa
  tela separada, feita para celular: lista das paradas na ordem certa, botão
  de navegação para Waze ou Google Maps em cada uma, e um botão para marcar a
  parada como concluída. O progresso marcado fica salvo no aparelho de quem
  abriu (sobrevive a fechar o navegador) e nenhum dado de cliente passa pelo
  servidor — o roteiro trafega inteiro dentro do link, no trecho depois do
  `#` (ver ADR-01 no documento de arquitetura).

---

## 3. Stack técnica

| Camada | Escolha | Observação |
|---|---|---|
| Mapa | Leaflet 1.9.4 (CDN cdnjs) | Mesma base que o uMap usa |
| Tiles | Esri Dark Gray Canvas (base + rótulos) | Sem cadastro. Substituiu o CARTO em 29/08/2026 |
| Rotas | OSRM — `router.project-osrm.org` | **Servidor público de demonstração** |
| Otimização | OSRM Trip API | Resolve TSP aproximado |
| Geocodificação | Nominatim (OpenStreetMap) | Sujeito a política de uso justo |
| Fontes | Space Grotesk, JetBrains Mono, Inter | Google Fonts |

### Identidade visual

Tema escuro. Variáveis CSS definidas em `:root`:

```
--ink:#0C1418  --panel:#16232B  --panel-2:#1D2E37  --line:#2A3D47
--text:#E7EEF2 --muted:#7C93A0  --amber:#F2A93C   --amber-dim:#B87F26
--teal:#2FB6A6 --danger:#E2604F
```

Âmbar = origem, rota traçada e paradas numeradas. Teal = clientes não
selecionados e estados de sucesso.

---

## 4. Decisões tomadas

1. **uMap não faz roteamento interno.** Ele apenas abre um link externo para o
   OSRM ao pedir direções. Por isso construímos app próprio sobre Leaflet + OSRM,
   usando o uMap apenas como ferramenta de criação/exportação dos pontos.
2. **Dados carregados por upload**, não fixos no código nem buscados de URL.
   Usuário carrega o backup do uMap (`.umap`) quando quiser.
3. **Múltiplas paradas** em vez de destino único — o caso de uso é visitar
   vários clientes na mesma viagem.
4. **Filiais de exemplo removidas** (Rio Maina / Próspera do primeiro protótipo).
   Apenas os pontos carregados pelo usuário aparecem.
5. **Arquivo HTML único** por enquanto — sem build step, sem framework.

---

## 5. Problemas resolvidos

**"Failed to fetch" na pré-visualização do chat.** A pré-visualização embutida
do Claude.ai bloqueia `fetch()` para domínios externos, quebrando as chamadas ao
OSRM e Nominatim. O código estava correto — funciona ao abrir o arquivo baixado
diretamente no navegador.

Solução aplicada: função `safeFetchJSON()` que envolve todas as chamadas de rede
e retorna mensagem explicativa em português quando a conexão falha, orientando o
usuário a baixar o arquivo.

**Marca d'água "API KEY REQUIRED" sobre o mapa (29/08/2026).** A CARTO passou
a exigir chave de acesso para os tiles `dark_all`. O servidor não recusava o
pedido — devolvia a imagem carimbada, sujando o mapa inteiro, inclusive no site
já publicado.

Solução aplicada: troca para o **Esri Dark Gray Canvas**, que serve sem
cadastro. Vem em duas camadas empilhadas (ruas + rótulos) e a ordem dos números
no endereço é `{z}/{y}/{x}`, invertida em relação ao CARTO. O Esri só tem imagem
real até o zoom 16 nesta região, então usamos `maxNativeZoom: 16` com
`maxZoom: 19` — o Leaflet continua deixando aproximar, ampliando a última
imagem disponível, em vez de deixar o mapa em branco.

Custo da troca: o fundo ficou cinza médio, mais claro que o preto-azulado
anterior, portanto menos integrado ao painel escuro. Alternativa avaliada e
descartada: Stadia Maps combinaria melhor visualmente, mas responde 401 fora do
`localhost` — exigiria cadastro e uma chave exposta no repositório público.

---

## 6. Formato de dados esperado

**Backup completo do uMap (`.umap`)** — é o único formato que preserva a
separação por camada. No uMap: painel **"Compartilhar e baixar"** → backup
completo.

```json
{
  "type": "umap",
  "layers": [
    {
      "type": "FeatureCollection",
      "_umap_options": { "name": "LABORATORIO BURIGO" },
      "features": [
        { "type": "Feature",
          "geometry": { "type": "Point", "coordinates": [-49.3697, -28.6775] },
          "properties": { "name": "CENTRAL" } }
      ]
    }
  ]
}
```

**Cada camada é um cliente; cada ponto dentro dela é uma filial.** O nome do
cliente muda de lugar conforme a instalação do uMap, e o app lê as três formas,
nesta ordem:

| Onde | Instalação |
|---|---|
| `properties.name` | `umap.hotosm.org` — **é o que a base real usa** |
| `_umap_options.name` | instalações mais recentes do uMap |
| `_storage.name` | instalações antigas |

O nome da filial vem sempre de `properties.name` da feature. Coordenadas em
ordem GeoJSON: `[longitude, latitude]`.

⚠️ **O download simples em `.geojson` não serve.** Ele achata todas as camadas
numa lista única e descarta os nomes — a informação de cliente não chega ao app.
Verificado em 29/08/2026 com um export real: as 11 features vinham com
`properties` contendo apenas `name`. O app recusa esse arquivo com uma mensagem
que ensina o caminho certo, em vez de carregar os pontos sem agrupamento.

⚠️ **Exportação do uMap só inclui camadas visíveis.** Camada com o "olho"
desligado fica de fora do arquivo. Se faltar cliente no app, é o primeiro lugar
a conferir.

Arquivo de exemplo versionado: `Exemplos/exemplo.umap` — 4 clientes fictícios
com 6 filiais na região de Criciúma. Cobre de propósito as **três** variantes de
nome de camada (`properties`, `_umap_options`, `_storage`) e inclui uma camada
vazia, para servir de teste de todos os casos.

O arquivo com os pontos reais fica **fora do repositório**, porque o
repositório é público (ver `.gitignore`).

## 7. Limitações conhecidas

- **OSRM público** é servidor de demonstração, sem garantia de disponibilidade
  nem uso comercial. Para produção: instância própria de OSRM ou GraphHopper
  (auto-hospedadas, gratuitas) ou API paga (Mapbox Directions, Google Directions).
- **Nominatim** tem política de uso justo — limite aproximado de 1 requisição por
  segundo. Volume alto de geocodificação exige alternativa.
- **Sem persistência.** Nada é salvo: recarregar a página zera tudo.
- **Sem busca no checklist.** Com dezenas ou centenas de clientes, rolar a lista
  fica impraticável.
- **Otimização puramente geográfica.** Considera apenas distância/tempo de carro.
  Não trata prioridade de cliente, janelas de horário ou duração da visita.
- **Sem navegação por voz dentro do app** — resolvido de propósito (ADR-02):
  o app não compete com Waze/Google Maps, só abre um deles por parada.

---

## 8. Direções em aberto

Decididas em 22/08/2026, registradas em detalhe no documento de arquitetura
(`ARQUITETURA-E-REQUISITOS.html`, fora do repositório público por descrever a
operação — ver `.gitignore`):

- **Quem usa:** dois papéis — escritório planeja, campo (2-3 pessoas) executa
  e marca progresso. Operação: entrega de toner + manutenção.
- **Origem da base de clientes:** uMap exportado como backup completo (`.umap`), volume
  pequeno (dezenas de pontos). Sem integração com ERP/planilha por ora.
- **Navegação:** app + botão para Waze/Google Maps por parada (ADR-02) —
  ✅ implementado na Fase 1.
- **Hospedagem:** GitHub Pages — ✅ feito, ver seção 10.
- **Como o roteiro chega ao campo:** link com o roteiro codificado no
  fragmento da URL (ADR-01) — ✅ implementado na Fase 1.

Ainda em aberto:
- Busca/filtro no checklist quando a lista crescer (Fase 3)
- Tipo de serviço por parada — ✅ implementado na v1.9
- Troca do OSRM público antes do uso diário sério (Fase 4)

Adiados a pedido do usuário em 29/08/2026 (continuam descritos no documento
de arquitetura, para retomar quando fizer sentido):
- Campo de observação livre por parada (Fase 2)
- Dividir clientes entre as 2-3 pessoas da equipe, um link por pessoa (Fase 2)

---

## 9. Histórico

| Passo | O que foi feito |
|---|---|
| 1 | Protótipo inicial: destino único, filiais fixas no código (Rio Maina / Próspera) |
| 2 | Reconstrução: upload de GeoJSON, múltiplas paradas, otimização de ordem, filiais de exemplo removidas |
| 3 | Correção do "Failed to fetch" — `safeFetchJSON()` com mensagem explicativa |
| 4 | **Fase 1 do roadmap:** modo campo — gerar link do roteiro, tela separada para celular, navegação via Waze/Maps, marcar parada concluída com progresso salvo no aparelho |
| 5 | **v1.1** — Troca do provedor de tiles: CARTO (passou a carimbar o mapa) → Esri Dark Gray Canvas, sem cadastro. Adotado o versionamento numerado (seção 11) |
| 6 | **v1.2** — Clientes separados por camada: app passa a ler o backup completo do uMap (`.umap`) e o checklist vira uma lista agrupada por cliente, com cascata |
| 7 | **v1.3** — Correção: a instalação usada (`umap.hotosm.org`) guarda o nome da camada em `properties.name`, forma que a v1.2 não lia |
| 8 | **v1.4** — Correção: o checklist agrupado não rolava (filhos de flex sendo espremidos em vez de gerar rolagem) |
| 9 | **v1.5** — Nome do cliente passa a aparecer no modo campo, acima do nome da filial |
| 10 | **v1.6** — Ícones do Waze/Maps nos botões de navegação; "Marcar como concluída" maior e com confirmação em duas etapas |
| 11 | **v1.7** — Ícones reais do Waze e do Google Maps (recortados pelo usuário); parada concluída esconde os botões de navegação |
| 12 | **v1.8** — Reabrir uma parada concluída também passa a pedir confirmação em duas etapas |
| 13 | **v1.9** — Fase 2 (parcial, a pedido do usuário): tipo de serviço por parada, com lista configurável |

---

## 10. Estrutura do projeto e hospedagem

O projeto é versionado em Git e publicado no GitHub Pages.

**Estrutura (a partir de 22/08/2026):**

```
PROJETO APP LOGISTICA/        ← raiz do repositório Git
├── index.html                ← o app (servido pelo GitHub Pages)
├── CLAUDE.md                 ← este arquivo (contexto do projeto)
├── LOG-ALTERACOES.txt        ← log de todas as alterações
├── README.md
├── .gitignore
├── Exemplos/
│   └── exemplo.umap          ← 3 clientes fictícios, só para demonstrar o formato
├── .claude/launch.json       ← config do servidor local de testes
└── backups/                  ← pontos de restauração (NÃO versionado)
```

- **Repositório:** github.com/antoniocmp97/roteador-clientes (público)
- **Site publicado:** https://antoniocmp97.github.io/roteador-clientes/
- Todo `git push` para o branch `main` republica o site automaticamente.

**Regra de dados:** o repositório é público, então dados reais de clientes
**nunca** são versionados. O app carrega o `.umap` por upload — os dados
ficam só na máquina de quem usa. O `.gitignore` bloqueia `clientes*.geojson`
e a pasta `dados-reais/` por precaução.

**Para testar localmente:** abrir o `index.html` no navegador, ou usar o
servidor local (`.claude/launch.json`). Não usar a pré-visualização embutida
do chat — ela bloqueia as chamadas de rede ao OSRM e ao Nominatim.

---

## 11. Versionamento e backups

Adotado em 29/08/2026, a pedido do usuário.

### Como o número é contado

Formato `MAIOR.MENOR`, subindo de 0.1 em 0.1 (1.0 → 1.1 → 1.2 … → 1.9 → 2.0).

Uma versão nova é fechada quando acontece **um** destes dois gatilhos:

1. **Uma alteração mediana ou grande** — sobe a versão na hora.
2. **Cinco alterações leves acumuladas** — sobem a versão juntas.

O objetivo do segundo gatilho é não gerar uma versão nova a cada ajuste de
texto ou de cor, o que encheria a pasta de backups sem necessidade.

**O que é alteração leve:** não muda o comportamento do app. Texto, cor,
espaçamento, comentário no código, documentação, renomear variável.

**O que é mediana ou grande:** muda o que o app faz ou o que o usuário vê.
Funcionalidade nova, troca de serviço externo, mudança de fluxo, correção de
bug que atrapalhava o uso.

### Contador de alterações leves

Quando houver alteração leve, incrementar aqui. Ao chegar em 5, fechar versão
nova e zerar o contador.

```
Leves acumuladas desde a v1.9:  1 / 5
```

### Onde o número aparece

Na constante `VERSAO` no início do script do `index.html`, e a partir dela num
selo discreto ao lado da marca **nas duas telas** — escritório e campo. A ideia
é que quem está na rua consiga informar a versão ao relatar um problema.

Ao fechar versão nova, atualizar a constante `VERSAO` junto com o resto.

### Backups

Cada versão fechada ganha uma cópia em `backups/vX.Y_AAAA-MM-DD_apelido/`,
com o `index.html` daquela versão e um `LEIA-ME.txt` explicando o que mudou e
por quê.

⚠️ **A pasta `backups/` não é versionada** (bloqueada no `.gitignore`), então
essas cópias existem **apenas nesta máquina**. Numa próxima troca de computador
elas se perdem se a pasta não for copiada junto. A rede de segurança real
continua sendo o Git, que guarda todas as versões no GitHub de qualquer forma —
os backups locais são conveniência, não garantia.

### Versões existentes

| Versão | Data | O que é |
|---|---|---|
| 1.0 | 22/08/2026 | Fase 1 — modo campo. Validada em celular real. Resgatada do commit `2c87046` |
| 1.1 | 29/08/2026 | Troca CARTO → Esri e adoção do versionamento |
| 1.2 | 29/08/2026 | Leitura do `.umap` e checklist agrupado por cliente |
| 1.3 | 29/08/2026 | Correção do nome da camada no formato do `umap.hotosm.org`. **Validada com a base real** |
| 1.4 | 29/08/2026 | Correção da rolagem do checklist agrupado |
| 1.5 | 29/08/2026 | Nome do cliente no modo campo (formato do link v2, retrocompatível com v1) |
| 1.6 | 29/08/2026 | Ícones dos apps de navegação e confirmação em duas etapas ao concluir |
| 1.7 | 29/08/2026 | Ícones reais (Waze/Maps) e trava de navegação na parada concluída |
| 1.8 | 29/08/2026 | Confirmação em duas etapas também para reabrir uma parada concluída |
| 1.9 | 29/08/2026 | Fase 2 (parcial): tipo de serviço por parada, lista configurável |
