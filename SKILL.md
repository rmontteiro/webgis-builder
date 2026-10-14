---
name: webgis-builder
description: Monta sistemas WebGIS e geoportais do zero, dos dados do QGIS ao mapa publicado numa URL (Leaflet, GeoJSON, GitHub, Vercel, Supabase/PostGIS). Use ao criar, evoluir ou publicar mapas interativos na web.
---

# WebGIS Builder

Método para construir um geoportal (WebGIS) completo: organizar os dados, gerar o mapa interativo, testar localmente, publicar numa URL e evoluir para banco de dados, login e dashboards.

Ideia central: o GIS desktop gera arquivos que viajam por e-mail (shapefile v2, v3, PDF...). O WebGIS troca isso por **uma URL sempre atualizada**. A análise pesada continua no QGIS ou em Python; o WebGIS serve para **comunicar resultados** a quem não usa GIS.

## Fluxo de trabalho

Siga as fases na ordem. Não pule a Fase 1: a maioria dos erros de WebGIS nasce de dados mal preparados.

### Fase 0: briefing (antes de qualquer código)

Responda, ou pergunte ao usuário só o que não der para inferir:

1. **Objetivo e público**: o que o mapa precisa mostrar e para quem (cliente, secretaria, investidor, público geral).
2. **Área de interesse**: município, bacia, empreendimento.
3. **Camadas mínimas**: escolha poucas camadas e bem direcionadas. Camada demais deixa o portal lento e o usuário perdido.
4. **Sigilo**: dados públicos podem ir num repositório aberto. Dados sensíveis ou pessoais (LGPD) pedem repositório privado e controle de acesso (Fase 6).
5. **Arquitetura**: estático (GeoJSON no próprio repositório, o padrão) ou com banco (Supabase/PostGIS) quando houver muitos dados, edição, login ou atualização frequente.

### Fase 1: "organizar a casa" (dados)

**Estrutura de pastas** (nunca trabalhar na pasta Downloads):

```
projeto/
  dados-brutos/      # zips e arquivos originais baixados
  qgis/              # projeto .qgz e arquivos intermediários
  geoportal/         # SOMENTE o que vai para a web (vira o repositório)
    data/            # GeoJSON finais
    assets/          # logo, favicon, imagens
  apoio/             # instruções em .md, PDFs convertidos para .md, referências
```

**Fontes de dados brasileiras**:
- Limites municipais: IBGE, Malha Municipal (baixar por UF).
- Imóveis rurais: SICAR, consulta pública, Downloads, por município (perímetro dos imóveis, reserva legal, vegetação nativa). Atenção: são dados **autodeclarados**, a maioria não validada, com muitas sobreposições. Nomeie a camada "Imóveis rurais declarados".
- Hidrografia: ANA, Base Hidrográfica Ottocodificada (trechos de drenagem, GeoPackage nacional, pesado).
- Rodovias, ferrovias e unidades de conservação: DNIT (download direto, às vezes já em GeoJSON).

**Regras de preparo**:
1. **Tudo em WGS 84 (EPSG:4326)**. Os mapas de fundo da web estão em WGS 84 / Web Mercator. No QGIS a reprojeção "on the fly" esconde o problema, mas na web uma camada em SIRGAS 2000 UTM gera incompatibilidade. A diferença SIRGAS 2000 x WGS 84 é desprezível na escala de um WebGIS, mas o correto é transformar e não só redefinir o SRC.
2. **Exportar em GeoJSON** (formato texto, nativo da web). Shapefile na web obriga conversão no navegador e perde performance.
3. **Codificação UTF-8**. Acento virando "?" ou "Ã©" significa que a codificação se perdeu. **Corrija na fonte**: no QGIS, Propriedades da camada > Fonte > Codificação (shapefiles brasileiros costumam ser Windows-1252 ou ISO-8859-1), confira a tabela e só então exporte em UTF-8. Remendos no JavaScript pioram o problema.
4. **Recorte por área de influência**, não pelo limite exato: faça um buffer da área de interesse (em graus, cerca de 0,5° ≈ 50 km, ou em metros num SRC projetado) e use **Selecionar por localização (interseção)** em vez de Recortar, para não cortar rios e rodovias ao meio. Exporte só os selecionados.
5. **Leveza**: remova atributos inúteis, simplifique geometrias (Simplificar, ou mapshaper) e reduza a precisão das coordenadas (6 casas decimais bastam). Mire em menos de 5 MB por camada; acima disso use tiles (Fase 6).
6. Cálculo de área e distância se faz em SRC projetado (ex.: SIRGAS 2000 / Albers de área igual para o Brasil, ou a zona UTM) no QGIS ou com turf.js no navegador, nunca em graus.

Equivalente em linha de comando, quando não houver QGIS:

```bash
ogr2ogr -f GeoJSON -t_srs EPSG:4326 -lco COORDINATE_PRECISION=6 \
  -lco RFC7946=YES --config SHAPE_ENCODING CP1252 \
  -select "cod_imovel,municipio,situacao,area_ha" \
  geoportal/data/imoveis.geojson dados-brutos/AREA_IMOVEL.shp
```

**Instruções em Markdown**: antes de passar um PDF grande para o agente, converta só o trecho útil para `.md` e guarde em `apoio/`. Gasta muito menos tokens e o agente entende melhor.

### Fase 2: plano antes do código

Escreva um **plano de implementação** curto e aprove antes de codar: tecnologias, camadas e simbologia de cada uma, interatividade, arquivos que serão criados. O usuário pode pedir em linguagem natural ("quero um geoportal interativo com as camadas da pasta X, três mapas de fundo, completo, funcional e com aparência profissional"); o agente transforma isso no plano.

**Stack padrão (estático, sem build)**: HTML5 + CSS puro + JavaScript + **Leaflet**. Simples de hospedar em qualquer lugar. Use MapLibre GL quando precisar de tiles vetoriais, muitos dados ou 3D; OpenLayers quando precisar de projeções não Web Mercator ou de OGC avançado.

**Requisitos de um geoportal "completo e profissional"**:
- 3 mapas de fundo com seletor: ruas (OSM/Carto), satélite (Esri World Imagery) e escuro (Carto Dark Matter), com atribuição.
- Painel de camadas liga/desliga e legenda gráfica gerada pela própria simbologia.
- Simbologia legível: limite municipal só com contorno (tracejado, sem preenchimento), hidrografia em linhas azuis, polígonos com preenchimento translúcido e borda fina.
- **Ordem de camadas e cliques**: polígonos de contorno (limite municipal) ficam por baixo e com `interactive: false`; senão "capturam" todo clique e o popup mostra sempre o município. A camada principal (ex.: imóveis) fica por cima.
- Popup ou painel com os atributos, com rótulos amigáveis e números formatados em pt-BR; destaque ao passar o mouse.
- Busca por código ou nome (ex.: código do CAR) com zoom automático na feição.
- Simbologia temática: categórica (ex.: situação do CAR) e por classes (ex.: área em 5 classes), com seletor quando houver mais de uma.
- Dashboard de estatísticas (contagens, área total, gráficos de barras por classe/categoria).
- **Filtro cruzado**: clicar numa barra do gráfico filtra mapa e dados; os não selecionados ficam esmaecidos (não somem); contagens e legenda refletem o filtro; limpar clicando de novo na barra ou num botão "Limpar filtro".
- Identidade visual: logo, favicon, cores da marca. Tema escuro como padrão quando a logo pedir, com alternância claro/escuro. Cores semânticas dos dados não mudam com o tema.
- Responsivo (funciona no celular).

Esqueleto de referência:

```html
<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Geoportal</title>
  <link rel="icon" href="assets/favicon.png">
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
  <style>html,body,#map{height:100%;margin:0}</style>
</head>
<body>
<div id="map"></div>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script>
const map = L.map('map');
const bases = {
  'Ruas': L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png', {attribution:'© OpenStreetMap © CARTO'}),
  'Satélite': L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {attribution:'© Esri'}),
  'Escuro': L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {attribution:'© OpenStreetMap © CARTO'})
};
bases['Ruas'].addTo(map);
const overlays = {};
const load = (url) => fetch(url).then(r => { if (!r.ok) throw new Error(url); return r.json(); });

Promise.all([load('data/limite.geojson'), load('data/drenagem.geojson'), load('data/imoveis.geojson')])
.then(([lim, dren, imo]) => {
  const limite = L.geoJSON(lim, {interactive:false, style:{color:'#7b2cbf', weight:2, dashArray:'6 4', fill:false}}).addTo(map);
  overlays['Limite municipal'] = limite;
  overlays['Drenagem'] = L.geoJSON(dren, {style:{color:'#1e88e5', weight:1}}).addTo(map);
  overlays['Imóveis rurais declarados'] = L.geoJSON(imo, {
    style:{color:'#2e7d32', weight:0.8, fillOpacity:0.25},
    onEachFeature:(f, l) => l.bindPopup(Object.entries(f.properties)
      .map(([k, v]) => `<b>${k}</b>: ${v ?? '-'}`).join('<br>'))
  }).addTo(map);
  map.fitBounds(limite.getBounds());
  L.control.layers(bases, overlays, {collapsed:false}).addTo(map);
})
.catch(e => alert('Erro ao carregar camadas: ' + e.message));
</script>
</body>
</html>
```

Para projetos maiores, separe em `index.html`, `css/style.css` e `js/` (map.js, layers.js, stats.js, filters.js) e use caminhos **relativos**.

### Fase 3: rodar localmente

Abrir o `index.html` com duplo clique (endereço `file://`) faz o navegador bloquear o `fetch` dos GeoJSON (CORS): o mapa abre, mas as camadas não aparecem. É o erro mais comum. Sirva por HTTP:

```bash
python -m http.server 8000   # depois abra http://localhost:8000
```

Para usuários que não usam terminal, crie um atalho de um clique (`iniciar-geoportal.bat` no Windows; `.sh` no Linux/macOS):

```bat
@echo off
cd /d "%~dp0"
start "" http://localhost:8000
python -m http.server 8000
```

### Fase 4: iterar

- Teste cada mudança no navegador (F12 > Console para erros). Recarregue com Ctrl+F5 para não ver cache; reinicie o servidor se necessário.
- Relate problemas com print e uma frase ("a logo ficou cortada", "a simbologia por situação não deu certo"). O agente diagnostica.
- **Contexto vence insistência**: se o mesmo erro volta duas vezes, procure a causa na fonte (dados, codificação, caminho do arquivo) em vez de repetir "não deu certo".
- Ao gerar uma versão corrigida de um dado (ex.: `imoveis_2.geojson`), atualize o código para apontar para ela e **apague a versão antiga**, para nenhum script continuar lendo o arquivo corrompido.
- Em ambiguidades de design (esmaecer ou esconder no filtro? legenda clicável?), pergunte com opções curtas e siga.
- Para melhorar o visual, aplique uma skill ou guia de UI/UX e peça "estética profissional alinhada à identidade visual de X".
- Use a IA também para aprender: peça para explicar cada tecnologia escolhida e as alternativas, para conseguir justificar o projeto ao cliente.

### Fase 5: publicar (uma URL para o cliente)

1. `git init` na pasta `geoportal/`, com `.gitignore` (`.env`, `*.zip`, `dados-brutos/`, `__pycache__/`).
2. Crie o repositório no GitHub. **Público só se os dados forem públicos**; senão privado.
3. Faça commit e push do código e dos GeoJSON (cada arquivo abaixo de 100 MB; para dados grandes use banco ou storage).
4. **Vercel**: Add New > Project > importar o repositório > Framework Preset "Other" > sem build command, output na raiz > Deploy. Cada push no GitHub atualiza o site sozinho. O plano gratuito atende bem portais estáticos. Alternativas: GitHub Pages ou Netlify.
5. Domínio próprio (ex.: `geoportal.suaempresa.com.br`): Project > Settings > Domains, e crie o CNAME no provedor de DNS.
6. Abra a URL publicada, confira camadas, filtros, tema e celular, e só então envie ao cliente.

Nunca envie a pasta com executável para o cliente "abrir no computador dele": isso volta à lógica do desktop. Mande a URL.

### Fase 6: evoluir

- **Banco de dados (Supabase = Postgres + PostGIS, tem plano gratuito)**: quando os dados são grandes, editados com frequência ou precisam de login. Importe com `ogr2ogr -f PostgreSQL` ou `shp2pgsql`, crie índice espacial (`CREATE INDEX ... USING GIST (geom)`) e sirva GeoJSON por função RPC ou view, por exemplo com `ST_AsGeoJSON` e filtro `ST_Intersects` pela extensão do mapa.
- **Níveis de acesso e LGPD**: login pelo Supabase Auth e Row Level Security por perfil (ex.: a secretaria vê tudo; o empreendedor vê só o próprio processo). A chave `anon` pode ir para o front-end **apenas** com RLS ativo; a `service_role` nunca sai do servidor. Chaves ficam em `.env`, que nunca é commitado. Rode uma revisão de segurança antes de entregar.
- **Dados que atualizam sozinhos**: consuma APIs ou webhooks em vez de arquivos (ex.: Conecta gov.br, que exige solicitação de acesso pelo órgão). Agende a atualização quando a fonte não empurra dados.
- **Raster pesado (ortofoto de drone, MDT)**: converta para **COG** (no QGIS 4: "Create Cloud Optimized GeoTIFF"; ou `gdal_translate -of COG`) e sirva por tiles (XYZ, TiTiler ou GeoServer). Nunca carregue o GeoTIFF inteiro no navegador.
- **Vetor pesado**: tiles vetoriais (PMTiles via tippecanoe, Martin ou pg_tileserv, GeoServer com WMS/WFS/vector tiles) e MapLibre GL.
- **3D**: nuvem de pontos ou modelo de LiDAR/drone com Cesium (3D Tiles) ou Potree, integrado na mesma interface (não numa aba solta), com ferramentas de anotação e captura quando o cliente pedir.
- **Múltiplos mapas**: abas ou menu, cada item um mapa temático.
- **Planilha para dashboard**: dados tabulares (ex.: portal da transparência) viram dashboard interativo com a mesma abordagem, muitas vezes substituindo um Power BI.

**Arquitetura auto-hospedada** (empresa com servidor próprio): Docker Compose com PostGIS + GeoServer (ou pg_tileserv/Martin) + Nginx servindo o front, HTTPS, backups do banco e WMS/WFS para consumo no QGIS.

## Checklist de entrega

- [ ] Todas as camadas em EPSG:4326, GeoJSON UTF-8, acentos corretos no popup
- [ ] Camadas recortadas pela área de influência e leves (menos de 5 MB cada, ou tiles)
- [ ] Limite municipal não bloqueia cliques; camada principal por cima
- [ ] 3 mapas de fundo com atribuição; legenda e painel de camadas
- [ ] Popups com rótulos amigáveis; busca com zoom
- [ ] Estatísticas e filtro cruzado funcionando (esmaecer + limpar filtro)
- [ ] Logo, favicon, tema claro/escuro, layout no celular
- [ ] Funciona por HTTP local e na URL publicada
- [ ] Repositório com visibilidade correta, sem `.env`, chaves ou dados sigilosos
- [ ] Fonte e data de cada dado indicadas (ex.: "SICAR: dados autodeclarados")

## Ferramentas de vibecoding

O código pode ser gerado em linguagem natural com agentes de IA: Google Antigravity (gratuito com limites, modo agente ou IDE) e Claude Code (pago, mais capaz em código; recomendado). Cursor, Codex e Windsurf funcionam do mesmo jeito. Quanto mais contexto (objetivo, dados, cliente, padrões visuais), melhor o resultado. Use modelos mais simples para ajustes pequenos e os mais fortes para funcionalidades complexas.
