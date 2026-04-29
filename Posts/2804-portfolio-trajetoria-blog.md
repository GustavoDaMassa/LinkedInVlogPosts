---
status: publicado 29/04/2026
contexto: Implementação do módulo de Trajetória (timeline da jornada de desenvolvimento) e do Blog técnico no portfólio, com a estratégia de usar o LinkedIn como entrada e o blog como profundidade para quem quiser mais detalhes.
objetivo: Mostrar a estratégia de conteúdo montada (LinkedIn como funil, blog como prova técnica) e os detalhes de implementação dos dois módulos — para atrair devs e recrutadores técnicos que valorizam raciocínio além do código.
---

# Portfólio além do GitHub: trajetória visual, blog técnico e uma estratégia de conteúdo

GitHub é um bom lugar para guardar código. Não é um bom lugar para contar uma história.

Repositórios mostram *o que* foi feito. Não mostram *por que*, não mostram a sequência, não mostram o raciocínio por trás das decisões. Para quem está se posicionando no mercado, isso é uma lacuna — e foi exatamente ela que me motivou a construir dois módulos novos no meu portfólio: a **Trajetória** e o **Blog**.

---

## A estratégia de conteúdo

A ideia central é simples: o LinkedIn é a entrada, o blog é a profundidade.

Posts no LinkedIn tendem a ser curtos por necessidade. Quem lê quer o gancho, não o tutorial completo. Mas existe uma parcela — recrutadores técnicos, devs curiosos, pessoas avaliando fit — que quer mais. Para essas pessoas, o post no LinkedIn vira um convite para continuar a leitura no blog, onde os detalhes técnicos reais estão documentados.

Isso cria uma separação clara de responsabilidades no conteúdo: o LinkedIn atrai, o blog retém e prova.

---

## Trajetória — a timeline da jornada

A Trajetória é uma linha do tempo de tudo que construí e estudei desde o início. Cada entrada é um arquivo Markdown com frontmatter YAML:

```md
---
id: "financeapi"
date: "2024-03"
title: "FinanceAPI"
type: "marco"
tags: ["Java", "Spring Boot", "PostgreSQL"]
parallel: ["outro-projeto"]
github: "https://github.com/..."
---

<!-- NARRATIVA -->
Texto em prosa sobre o contexto e motivação.

<!-- TECNICO -->
Detalhes de arquitetura, decisões e trade-offs.
```

Cada entrada pode ter dois modos de leitura: **narrativo** (prosa, contexto, por que foi feito) e **técnico** (arquitetura, decisões, trade-offs). O usuário alterna entre eles com um toggle, e a preferência fica salva no `localStorage`.

No componente, as entradas são agrupadas por trimestre usando uma função simples sobre a data de cada entry:

```js
function getPeriodKey(dateStr, t) {
  const [year, month] = dateStr.split('-');
  const q = Math.ceil(parseInt(month, 10) / 3);
  return `${year} · ${t('trajetoria.quarter', { count: q })}`;
}
```

Isso dá uma granularidade melhor do que agrupar por ano, sem sobrecarregar visualmente.

---

## Blog — posts técnicos com estrutura dupla

O Blog segue a mesma lógica de conteúdo: cada post é um Markdown com frontmatter e seções marcadas por comentários HTML. A separação narrativa/técnica existe aqui também, mas para posts do blog o corpo inteiro é tratado como conteúdo único se não houver marcadores.

O carregamento dos posts usa `import.meta.glob` do Vite, que resolve todos os arquivos `.md` do diretório em tempo de build:

```js
const modules = import.meta.glob('./*.md', { query: '?raw', import: 'default', eager: true });
```

Com `eager: true`, todos os arquivos são importados de forma síncrona no bundle — sem lazy loading, sem requisição em runtime. Os posts existem como strings no bundle, prontos para serem parseados.

O parser de frontmatter é custom, sem dependência externa:

```js
function parseYaml(yaml) {
  const result = {};
  for (const line of yaml.split('\n')) {
    const colonIdx = line.indexOf(':');
    if (colonIdx === -1) continue;
    const key = line.slice(0, colonIdx).trim();
    const raw = line.slice(colonIdx + 1).trim();
    if (raw.startsWith('[')) {
      result[key] = inner.split(',').map(v => v.trim()).filter(Boolean);
    } else {
      result[key] = raw.replace(/^["']|["']$/g, '');
    }
  }
  return result;
}
```

A decisão de não usar `gray-matter` ou similares foi intencional: o parser precisa de exatamente o que o projeto usa. Mais dependência, mais surface area de problema.

---

## i18n

Tanto o Blog quanto a Trajetória suportam português e inglês via `react-i18next`. As strings fixas da UI são chaves de tradução. O conteúdo dos posts e entries tem versões `_en` embutidas no próprio Markdown, extraídas pelo parser.

Isso mantém tudo em um único arquivo por post — sem duplicar arquivos por idioma, sem sincronizar manualmente.

---

## Decisões e trade-offs

**Sem CMS, sem banco de dados.** Todo o conteúdo vive em arquivos Markdown versionados no git. Adicionar um post é abrir um arquivo, escrever e fazer commit. Nenhuma interface de admin, nenhum token de API, nenhuma dependência externa de conteúdo.

**Build estático.** O `import.meta.glob` com `eager: true` significa que os posts fazem parte do bundle. Em um blog de escala isso seria inviável — aqui, com dezenas de posts, é razoável e elimina qualquer necessidade de servidor para conteúdo.

**Modo narrativo/técnico.** A mesma entrada pode ser lida por um recrutador não técnico (narrativa) ou por um dev avaliando decisões de arquitetura (técnico). Isso evita a dicotomia comum de "portfolio para RH" vs "portfolio para devs".

---

Esse sistema ainda está evoluindo. A próxima etapa é exatamente essa: usar cada post do blog como referência nos posts do LinkedIn, criando um fluxo consistente entre o que aparece na rede e o que está documentado aqui.

Esse post é o primeiro exemplo disso. Se você chegou até aqui, é porque o filtro funcionou.
