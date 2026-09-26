# Roteador de Clientes — Hagamorfis

Aplicativo web para montar rotas de visita com múltiplas paradas a partir de
uma lista de clientes em GeoJSON. Feito para uso em Criciúma/SC.

**➡️ Acesse: https://antoniocmp97.github.io/roteador-clientes/**

---

## O que faz

- Carrega os clientes do backup completo do uMap (`.umap`) — clique ou arraste
- Separa por cliente: cada camada do uMap é um cliente, e os pontos dentro dela
  são as filiais. A lista abre em cascata ao clicar no nome do cliente
- Mostra todos os pontos no mapa
- Define o ponto de partida por endereço digitado ou pela sua localização
- Permite escolher quais clientes visitar na viagem
- **Rota otimizada** — encontra automaticamente a melhor sequência de visitas
- **Nesta ordem** — calcula a rota respeitando a ordem que você definiu
- Mostra distância total, tempo estimado e o roteiro passo a passo
- Marca paradas como prioritárias (★), que ficam fixas no início
- Escolhe o tipo de serviço de cada parada (lista configurável)
- Opcionalmente fecha o roteiro voltando para a origem
- Divide o dia entre vários técnicos, em abas, com as rotas juntas no mapa
- Gera um **link do roteiro** para mandar ao técnico: ele abre numa tela própria
  de celular, com o mapa do trajeto, navegação por Waze/Maps e marcação do que
  já foi feito
- Tema claro/escuro e um **modo leve** para computador mais antigo

## Como usar

1. Abra o link acima (funciona no computador e no celular)
2. Carregue o backup completo do uMap (`.umap`) com seus clientes
3. Defina o ponto de partida: digite o endereço, toque no alfinete para usar sua
   localização, ou use o alvo para marcar o ponto exato clicando no mapa
4. Marque os clientes que vai visitar
5. Clique em **Rota otimizada** (ou **Nesta ordem**, se já sabe a sequência)
6. Se for mandar para alguém na rua, gere o link do roteiro e envie

## Formato do arquivo de clientes

O app lê o **backup completo do uMap** (`.umap`). No uMap, abra o painel
**"Compartilhar e baixar"** e escolha o backup completo.

É o único formato que preserva as camadas — e no uso deste app **cada camada é
um cliente** e os pontos dentro dela são as **filiais**:

```json
{
  "type": "umap",
  "layers": [
    {
      "type": "FeatureCollection",
      "_umap_options": { "name": "LABORATORIO EXEMPLO" },
      "features": [
        { "type": "Feature",
          "geometry": { "type": "Point", "coordinates": [-49.3697, -28.6775] },
          "properties": { "name": "CENTRAL" } }
      ]
    }
  ]
}
```

Coordenadas na ordem `[longitude, latitude]`. O nome da filial vem de
`properties.name`. O nome do cliente muda de lugar conforme a instalação do
uMap — o app lê `properties.name`, `_umap_options.name` e `_storage.name`, então
funciona tanto no `umap.hotosm.org` quanto nas demais.

> **O download simples em `.geojson` não serve.** Ele junta todas as camadas
> numa lista só e descarta os nomes, então a informação de cliente não chega ao
> app. Se você carregar um por engano, o app avisa e explica o caminho certo.
>
> Atenção também: a exportação do uMap **só inclui camadas visíveis**. Camada
> com o "olho" desligado fica de fora do arquivo.

Há um arquivo de exemplo em [`Exemplos/exemplo.umap`](Exemplos/exemplo.umap),
com 4 clientes fictícios na região de Criciúma apenas para demonstrar o formato.

## Privacidade dos dados

Os dados de clientes **não ficam neste repositório**. O aplicativo lê o arquivo
direto no navegador — nada é enviado nem armazenado em servidor. Cada pessoa
carrega o próprio arquivo ao usar.

## Tecnologia

Arquivo HTML único, sem build e sem dependências instaladas.

| Função | Serviço |
|---|---|
| Mapa | [Leaflet](https://leafletjs.com/) + ladrilhos [Esri](https://www.esri.com/) Dark/Light Gray Canvas |
| Rotas e otimização | [OSRM](http://project-osrm.org/) |
| Busca de endereços | [Nominatim](https://nominatim.openstreetmap.org/) (OpenStreetMap) |

> **Nota:** o OSRM usado é o servidor público de demonstração, sem garantia de
> disponibilidade e não indicado para uso comercial intenso. Para produção,
> considere hospedar uma instância própria ou usar uma API paga.

## Documentação

- [`CLAUDE.md`](CLAUDE.md) — contexto completo do projeto, decisões e limitações
- [`LOG-ALTERACOES.txt`](LOG-ALTERACOES.txt) — histórico detalhado de alterações
