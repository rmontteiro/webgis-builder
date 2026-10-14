# webgis-builder

**Português** · [Español](#español) · [English](#english)

Skill para agentes de IA (Claude) criarem geoportais WebGIS do QGIS à publicação: Leaflet, dashboards, GitHub, Vercel e Supabase/PostGIS.

## O que a skill faz

Guia o agente por um método em 6 fases:

1. **Briefing**: objetivo, público, área de interesse, camadas mínimas e sigilo dos dados.
2. **Organizar os dados**: tudo em WGS 84 (EPSG:4326), GeoJSON UTF-8, recorte pela área de influência, fontes brasileiras (IBGE, SICAR, ANA, DNIT).
3. **Plano antes do código**: o agente propõe o plano e só codifica depois da aprovação.
4. **Geoportal em Leaflet**: 3 mapas de fundo, legenda, busca, popups, estatísticas, filtro cruzado e tema claro/escuro.
5. **Rodar e publicar**: servidor local (evita o erro das camadas que não carregam), GitHub e Vercel, domínio próprio.
6. **Evoluir**: Supabase/PostGIS, login com níveis de acesso (RLS), ortofotos em COG, tiles vetoriais e 3D.

## Instalação

**claude.ai (app web, desktop ou celular)**
1. Baixe este repositório: botão **Code > Download ZIP**.
2. Em claude.ai, abra **Settings > Capabilities > Skills** e envie o ZIP (ou a pasta com o `SKILL.md`).
3. Ative a skill.

**Claude Code**
```bash
git clone https://github.com/rmontteiro/webgis-builder ~/.claude/skills/webgis-builder
```
Para usar só em um projeto, clone em `.claude/skills/webgis-builder` dentro dele.

## Como usar

Basta pedir em linguagem natural. A skill é carregada automaticamente quando o pedido é sobre WebGIS. Exemplos:

- "Crie um geoportal do município de Petrolina-PE com limite municipal, rodovias do DNIT e unidades de conservação."
- "Tenho estes shapefiles na pasta `dados/`. Prepare para a web e monte um WebGIS com três mapas de fundo."
- "Adicione um gráfico por situação do CAR que filtre o mapa ao clicar."
- "Publique meu geoportal no GitHub e na Vercel."
- "Coloque login: a secretaria vê tudo e o empreendedor só o próprio processo."

Dica: quanto mais contexto você der (objetivo, cliente, dados, identidade visual), melhor o resultado.

---

## Español

Skill para agentes de IA (Claude) que crean geoportales WebGIS de QGIS a la publicación: Leaflet, dashboards, GitHub, Vercel y Supabase/PostGIS.

### Qué hace la skill

Guía al agente con un método de 6 fases:

1. **Briefing**: objetivo, público, área de interés, capas mínimas y confidencialidad de los datos.
2. **Organizar los datos**: todo en WGS 84 (EPSG:4326), GeoJSON UTF-8, recorte por área de influencia, fuentes brasileñas (IBGE, SICAR, ANA, DNIT) o equivalentes de tu país.
3. **Plan antes del código**: el agente propone el plan y solo programa después de la aprobación.
4. **Geoportal en Leaflet**: 3 mapas base, leyenda, búsqueda, popups, estadísticas, filtro cruzado y tema claro/oscuro.
5. **Ejecutar y publicar**: servidor local (evita el error de capas que no cargan), GitHub y Vercel, dominio propio.
6. **Evolucionar**: Supabase/PostGIS, login con niveles de acceso (RLS), ortofotos en COG, tiles vectoriales y 3D.

### Instalación

**claude.ai (web, escritorio o móvil)**
1. Descarga este repositorio: botón **Code > Download ZIP**.
2. En claude.ai, abre **Settings > Capabilities > Skills** y sube el ZIP (o la carpeta con el `SKILL.md`).
3. Activa la skill.

**Claude Code**
```bash
git clone https://github.com/rmontteiro/webgis-builder ~/.claude/skills/webgis-builder
```
Para usarla solo en un proyecto, clónala en `.claude/skills/webgis-builder` dentro de él.

### Cómo usarla

Pide en lenguaje natural; la skill se carga automáticamente cuando el pedido trata de WebGIS. Ejemplos:

- "Crea un geoportal del municipio X con el límite municipal, la hidrografía y los predios rurales."
- "Tengo estos shapefiles en la carpeta `datos/`. Prepáralos para la web y arma un WebGIS con tres mapas base."
- "Agrega un gráfico por categoría que filtre el mapa al hacer clic."
- "Publica mi geoportal en GitHub y Vercel."
- "Agrega login: la secretaría ve todo y cada usuario solo sus propios registros."

Las instrucciones internas de la skill (`SKILL.md`) están en portugués; el agente las entiende y responde en tu idioma.

---

## English

A skill for AI agents (Claude) to build WebGIS geoportals end to end: QGIS data prep, Leaflet maps and dashboards, GitHub + Vercel deploy, Supabase/PostGIS.

### What the skill does

It walks the agent through a 6-phase method:

1. **Briefing**: goal, audience, area of interest, minimum layers and data confidentiality.
2. **Data preparation**: everything in WGS 84 (EPSG:4326), UTF-8 GeoJSON, clipped by an influence area, Brazilian open data sources (IBGE, SICAR, ANA, DNIT) or your country's equivalents.
3. **Plan before code**: the agent proposes a plan and only writes code after approval.
4. **Leaflet geoportal**: 3 basemaps, legend, search, popups, statistics, cross-filtering and light/dark theme.
5. **Run and publish**: local server (avoids the "layers don't load" CORS error), GitHub and Vercel, custom domain.
6. **Grow**: Supabase/PostGIS, login with access levels (RLS), orthophotos as COG, vector tiles and 3D.

### Installation

**claude.ai (web, desktop or mobile)**
1. Download this repository: **Code > Download ZIP**.
2. On claude.ai, open **Settings > Capabilities > Skills** and upload the ZIP (or the folder containing `SKILL.md`).
3. Enable the skill.

**Claude Code**
```bash
git clone https://github.com/rmontteiro/webgis-builder ~/.claude/skills/webgis-builder
```
To use it in a single project, clone it into `.claude/skills/webgis-builder` inside that project.

### Usage

Just ask in plain language; the skill loads automatically when the request is about WebGIS. Examples:

- "Build a geoportal for municipality X with the municipal boundary, hydrography and rural properties."
- "I have these shapefiles in `data/`. Prepare them for the web and build a WebGIS with three basemaps."
- "Add a chart by category that filters the map on click."
- "Publish my geoportal to GitHub and Vercel."
- "Add login: the agency sees everything, each user only their own records."

The skill's internal instructions (`SKILL.md`) are written in Portuguese; the agent understands them and replies in your language.
