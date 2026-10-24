# WikiGraph Brain --- Architecture, APIs, Libraries, and Development Guide

## 1. What the application does

WikiGraph is a client-side knowledge graph explorer. Its current
implementation is one HTML document containing markup, CSS, and
JavaScript. It retrieves a Wikipedia topic extract and up to 20 linked
page titles, renders those titles as nodes connected to a central topic,
and offers navigation, history, bookmarks, theming, and graph controls.

## 2. Languages

-   **HTML5:** page structure, navigation, inputs, cards, modal, graph
    container.
-   **CSS:** scrollbar styling, glass effects, transitions, and
    animation.
-   **JavaScript (ES6+):** API calls, event handling, graph data, local
    storage, rendering, and UI state.
-   **Tailwind utility classes:** styling conventions in HTML. Tailwind
    is a CSS framework, not a programming language.

No Python, Java, PHP, or Node.js backend is required by the supplied
HTML file.

## 3. Libraries and external services

  -----------------------------------------------------------------------
  Dependency/service      How it is used          How it is loaded now
  ----------------------- ----------------------- -----------------------
  Vis Network             Interactive             CDN script
                          nodes/edges, physics,   
                          clustering, navigation  

  Tailwind CSS            Utility classes and     CDN script
                          responsive styling      

  Font Awesome 6.4.0      Icons                   CDN stylesheet

  Google Fonts / Inter    Typography              Google Fonts stylesheet

  Wikipedia MediaWiki API Topic extract and page  Browser `fetch()`
                          links                   

  `localStorage`          Search history and      Browser built-in API
                          bookmarks               
  -----------------------------------------------------------------------

These dependencies are loaded externally, so an internet connection is
needed. The project does not currently use a package manager or package
lockfile.

## 4. API endpoint and request

The source uses this endpoint pattern:

``` text
https://en.wikipedia.org/w/api.php
  ?action=query
  &format=json
  &origin=*
  &prop=extracts|links
  &exintro=1
  &explaintext=1
  &pllimit=20
  &plnamespace=0
  &titles=URL_ENCODED_TOPIC
```

The JavaScript builds the URL and calls `fetch(url)`, then parses the
response using `response.json()`. It reads `data.query.pages`, takes the
first page, and uses `title`, `extract`, and `links`. If the page
identifier is `-1`, it displays a not-found notification.

### API concepts to understand

-   `fetch()` sends the HTTP request.
-   `async` / `await` makes asynchronous code easier to read.
-   `encodeURIComponent()` safely encodes the topic in a URL query.
-   JSON is the response data format.
-   `try/catch` handles request and parsing failures.
-   A public API does not automatically require an API key. This
    endpoint is used without a private key in this implementation.

## 5. Main application flow

1.  The page loads and initializes the bookmark count.
2.  A default topic, "Artificial Intelligence," is searched.
3.  The API returns the topic summary and linked page titles.
4.  `updateUI()` updates the selected topic, summary, reading-time
    estimate, Wikipedia link, and related concepts.
5.  `buildAdvancedGraph()` creates one central node, linked nodes, and
    edges.
6.  `renderVisNetwork()` constructs the Vis Network instance and binds
    node click events.
7.  Selecting another node searches for it and rebuilds the graph.
8.  History and bookmarks are stored as JSON strings in local storage.

## 6. Important functions

-   `fetchWikiData(topic)`: requests and parses Wikipedia data.
-   `updateUI()`: updates the topic card and related concepts.
-   `buildAdvancedGraph(centerNode, connectedNodes)`: creates node and
    edge arrays.
-   `renderVisNetwork()`: creates the interactive graph.
-   `filterGraphNodes(query)`: selects nodes matching a text filter.
-   `togglePhysics()`: enables or disables graph physics.
-   `toggleClustering()`: clusters or opens a graph branch.
-   `resetGraphZoom()`: fits the graph into the view.
-   `addToHistory(topic)` / `renderHistoryList()`: manages recent
    searches.
-   `toggleBookmarkCurrent()` / `renderBookmarksList()`: manages saved
    topics.
-   `toggleDarkMode()`: changes the theme.
-   `copyTopicSummary()`: copies the current topic and summary.

## 7. Build and run

### Quick method: no build tools

1.  Keep the file as `wikigraph_advanced_knowledge_dashboard.html`.
2.  Open it in a modern browser with internet access.
3.  Use a local static server during development if preferred.
4.  Check the browser developer console and Network tab when API or CDN
    requests fail.

### Maintainable method: split into modules

Create `index.html`, `src/main.js`, `src/api/wikipedia.js`,
`src/graph/graph.js`, `src/features/history.js`,
`src/features/bookmarks.js`, and `src/styles/main.css`. Move each
function into the module responsible for it, then import the modules
from `main.js`. Use a bundler such as Vite if you want dependency
management, development reloads, and production builds.

Example setup:

``` bash
npm create vite@latest wikigraph -- --template vanilla
cd wikigraph
npm install
npm install vis-network
npm run dev
```

Then move the existing markup and logic into the new project, replace
CDN references with installed packages where appropriate, and verify all
features. Tailwind, Font Awesome, and Google Fonts can remain CDN-based
initially or be installed/configured locally later.

## 8. How to create your own reusable JavaScript package

A package is reusable code with a documented public interface. Start
with a small, independent function---for example, a Wikipedia API
client---rather than packaging the entire UI immediately.

### Example package layout

``` text
wikigraph-api/
├── src/
│   └── index.js
├── package.json
├── README.md
└── LICENSE
```

Example `src/index.js`:

``` js
export async function searchWikipedia(topic) {
  if (!topic || !topic.trim()) {
    throw new Error("A topic is required.");
  }

  const params = new URLSearchParams({
    action: "query",
    format: "json",
    origin: "*",
    prop: "extracts|links",
    exintro: "1",
    explaintext: "1",
    pllimit: "20",
    plnamespace: "0",
    titles: topic.trim()
  });

  const response = await fetch(
    `https://en.wikipedia.org/w/api.php?${params}`
  );

  if (!response.ok) {
    throw new Error(`Wikipedia request failed: ${response.status}`);
  }

  const data = await response.json();
  const pages = data?.query?.pages;
  const page = pages ? Object.values(pages)[0] : undefined;

  if (!page || page.pageid === undefined) {
    throw new Error("Topic was not found.");
  }

  return {
    title: page.title,
    summary: page.extract ?? "",
    links: (page.links ?? []).map(item => item.title)
  };
}
```

Example `package.json`:

``` json
{
  "name": "wikigraph-api",
  "version": "1.0.0",
  "description": "Small client for retrieving Wikipedia topic summaries and links",
  "type": "module",
  "main": "./src/index.js",
  "exports": {
    ".": "./src/index.js"
  },
  "license": "MIT"
}
```

Before publishing, add tests, write clear documentation, choose a
license, check package-name availability, and test a clean installation.
Publish to npm only when you are ready to make the package public;
publishing is optional. For a private project, use a local file
dependency or a private registry. Do not publish secrets, tokens,
private data, or generated build artifacts unnecessarily.

## 9. Testing and quality

-   Test valid topics, missing topics, network errors, malformed
    responses, and empty extracts.
-   Test graph construction with zero, one, and many related links.
-   Test history and bookmarks after page reload.
-   Test light/dark themes and small screen layouts.
-   Add ESLint and a formatter such as Prettier to a modular project.
-   Consider unit tests with Vitest and browser-level tests with
    Playwright.

## 10. Security, privacy, and reliability

The app stores history and bookmarks locally in the current browser; it
does not define account sync. Local storage is not encrypted, so avoid
saving sensitive information. Treat API data as external input: prefer
`document.createElement()` and `textContent` over inserting untrusted
strings into `innerHTML`. Validate URLs before creating links. Add
loading-state cleanup in `finally`, handle non-OK HTTP responses, and
avoid excessive requests. Use HTTPS and review dependency versions and
licenses.

## 11. Current capabilities versus future work

**Present in the supplied HTML:** topic search, introductory extract,
linked-page list, single-hop graph display, node navigation, filter
selection, physics toggle, branch clustering, theme toggle, history,
bookmarks, copy summary, and Wikipedia article link.

**Not implemented in the supplied HTML:** user accounts, custom backend,
database, AI/LLM integration, private API keys, offline data, automatic
multi-level graph crawling, graph export, tests, or a package published
to npm. These can be added as separate features; do not describe them as
existing until implemented.

## 12. Suggested roadmap

1.  Split the single HTML file into modules.
2.  Create a dedicated Wikipedia API client and unit tests.
3.  Replace unsafe HTML interpolation with safe DOM rendering.
4.  Add request cancellation and clearer error handling.
5.  Improve accessibility and mobile graph controls.
6.  Add optional multi-hop expansion with request limits and
    deduplication.
7.  Add graph export and data-source attribution.
8.  Document installation, contribution rules, and licensing.
9.  Create a reusable package only after the API client has a stable
    interface.
