 # WikiGraph ---> Advanced Knowledge Ecosystem

## Project description

WikiGraph is a browser-based knowledge exploration dashboard that turns
Wikipedia topics into an interactive visual network. Instead of reading
isolated pages, users can search for a subject, view its introductory
summary, and explore connected concepts through a graph of nodes and
relationships. The interface is designed for students, researchers,
developers, and curious learners who want to discover how ideas relate
to one another.


live link -https://wikigraph-search-through-wiki.netlify.app/

The dashboard uses a responsive sidebar, a search bar, topic cards, and
a large graph workspace. A selected topic appears with its summary,
estimated reading time, related article links, and a direct link to the
corresponding Wikipedia page. Selecting a connected concept searches for
that topic and rebuilds the graph around it, enabling step-by-step
exploration across multiple topics.

WikiGraph includes search history and bookmarks stored in the browser's
local storage, so saved topics remain available in the same browser
profile. Users can switch between light and dark themes, filter graph
nodes, zoom and pan around the network, reset the graph view, and enable
or pause the graph's physics simulation. A clustering control groups a
subset of related nodes to make larger networks easier to inspect. Toast
notifications provide feedback for common actions, while loading and
error states communicate the status of searches.

The frontend is built with HTML, CSS, and vanilla JavaScript. Tailwind
CSS, loaded from its CDN, provides utility-based styling; Vis Network
renders and manages the interactive graph; Font Awesome supplies icons;
and Google Fonts provides the Inter typeface. Topic summaries and links
are retrieved from the public Wikipedia MediaWiki API using browser
fetch requests. No custom backend server or separate application
database is defined in the supplied HTML file.

The project is a useful foundation for learning API integration,
asynchronous JavaScript, DOM manipulation, data visualization, local
storage, responsive interfaces, and reusable frontend architecture. To
make it production-ready, developers should add robust API error
handling, safer DOM rendering for external content, accessibility
improvements, tests, and a build process. Optional future upgrades
include a modular codebase, TypeScript, automated tests, a backend
cache, alternate knowledge sources, exportable graphs, and richer
relationship data. WikiGraph is an exploration tool, not an
authoritative research source; users should open source articles and
verify important claims before relying on them.

## Features

-   Search Wikipedia topics and show introductory extracts.
-   Visualize a central topic and linked article nodes.
-   Navigate by selecting graph nodes or related concepts.
-   Filter graph nodes, zoom, pan, fit the graph, and toggle physics.
-   Cluster selected graph branches.
-   Save search history and bookmarks in browser local storage.
-   Toggle light/dark theme and copy the current summary.
-   Open the corresponding Wikipedia article.

## Technology stack

  -----------------------------------------------------------------------
  Layer                   Technology              Purpose
  ----------------------- ----------------------- -----------------------
  Structure               HTML5                   Page layout and
                                                  accessible content
                                                  structure

  Styling                 CSS and Tailwind CSS    Custom effects,
                          CDN                     responsive layout,
                                                  utility styling

  Application logic       Vanilla JavaScript      Search, UI updates,
                          (ES6+)                  state, event handlers

  Graph visualization     Vis Network             Nodes, edges, physics,
                                                  zoom, clustering

  Knowledge API           Wikipedia MediaWiki API Topic extracts and
                                                  linked pages

  Icons                   Font Awesome 6.4.0      Interface icons

  Typography              Google Fonts: Inter     UI typeface

  Persistence             Browser localStorage    Search history and
                                                  bookmarks

  Runtime                 Modern web browser      Executes the frontend
  -----------------------------------------------------------------------

## API and data flow

The page calls the MediaWiki API with `action=query`, `format=json`,
`prop=extracts|links`, `exintro=1`, `explaintext=1`, `pllimit=20`, and
`plnamespace=0`. The `origin=*` parameter supports cross-origin requests
from a browser. The search term is URL-encoded before being sent. The
response is parsed as JSON; the first returned page supplies the title,
extract, and linked page titles. Those titles become graph nodes
connected to the selected central topic.

This project does not contain a private API key, custom REST server,
authentication service, or server-side database. The visible "Wikipedia
API v2 Live" label is interface text; the code uses the Wikipedia
MediaWiki API endpoint shown in the JavaScript.

## Run locally

1.  Save the supplied file as
    `wikigraph_advanced_knowledge_dashboard.html`.
2.  Open it in a current browser while connected to the internet.
3.  Search for a topic, or use one of the suggested topic buttons.
4.  Allow the CDN-hosted libraries and Wikipedia API requests to load.
5.  For development, use a local static server (for example, VS Code
    Live Server) and the browser developer console to inspect errors.

## Suggested project structure

``` text
wikigraph/
├── index.html
├── src/
│   ├── main.js
│   ├── api/
│   │   └── wikipedia.js
│   ├── graph/
│   │   └── graph.js
│   ├── features/
│   │   ├── history.js
│   │   └── bookmarks.js
│   └── styles/
│       └── main.css
├── tests/
├── package.json
├── README.md
└── brain.md
```

The supplied project is currently a single HTML file. This tree is a
recommended refactor, not the existing file layout.

## Limitations and recommended improvements

-   The graph currently shows the selected topic and its linked pages;
    it does not fetch every node's relationships in one request.
-   Node filtering highlights/selects matching nodes rather than
    removing non-matching nodes.
-   Search and bookmark values are inserted into some HTML template
    strings. Use safe DOM APIs and `textContent` to reduce injection
    risk.
-   Add request timeouts, retry/backoff, clearer empty-result handling,
    and API rate-limit awareness.
-   Add keyboard and screen-reader support, responsive graph controls,
    and automated tests.
-   Pin dependency versions and consider npm-based builds for repeatable
    deployments.

## License and attribution 

This HTML file uses third-party libraries and a public knowledge API.
Check the current licenses and usage terms for Vis Network, Tailwind
CSS, Font Awesome, Google Fonts, and Wikipedia before redistribution or
commercial deployment. Attribute Wikipedia content appropriately and
follow the applicable reuse terms.
