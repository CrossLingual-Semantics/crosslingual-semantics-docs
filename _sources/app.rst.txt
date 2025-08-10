.. _app:

“Cultural Nuances” Dash Application (app.py)
============================================

This page documents the **Dash front-end** used to explore cross-lingual word-association graphs derived from the SWOW enhancement pipeline. The app renders per-language subgraphs, a **shared “Common” panel** (cross-lingual overlaps), and interactive edge tables exposing synset pairs (“Gemini pairs”) with domains and POS.

.. contents::
   :local:
   :depth: 2

High-level Role
---------------

``app.py`` is the **entrypoint**:

- Creates a Dash server (``server = app.server`` for WSGI hosting).
- Loads Cytoscape extra layouts.
- Delegates **UI construction** and **callbacks** to :mod:`view`.
- Mounts a clean, CDN-hosted CSS baseline (Normalize + Milligram).

Minimal Lifecycle
-----------------

1. **Import UI and callbacks**: ``from view import make_layout, register_callbacks``  
2. **Instantiate app**: ``app = Dash(..., suppress_callback_exceptions=True)``  
3. **Load layouts**: ``cyto.load_extra_layouts()``  
4. **Attach layout**: ``app.layout = make_layout()``  
5. **Register interactivity**: ``register_callbacks(app)``  
6. **Expose WSGI**: ``server = app.server``  
7. **Run locally** (port ``8085``)

.. code-block:: python

   from dash import Dash
   import dash_cytoscape as cyto
   from view import make_layout, register_callbacks

   app = Dash(__name__, external_stylesheets=[...], suppress_callback_exceptions=True)
   app.title = "Cultural Nuances"
   server = app.server

   cyto.load_extra_layouts()
   app.layout = make_layout()
   register_callbacks(app)

   if __name__ == "__main__":
       app.run_server(debug=False, port=8085)

Architecture & Module Map
-------------------------

The app is deliberately thin; most logic is factored into small, focused modules:

- :mod:`view` – Page layout, control panel, **callbacks** (graph updates, edge-detail table).
- :mod:`graph` – Graph construction for **single** and **multi-language** views  
  (``graph_for_lang``, ``build_graph``).
- :mod:`ui` – Cytoscape stylesheet + convenience constructor (``cygraph``).
- :mod:`utils` – Text normalization, **lemma extraction** and EN translation heuristics.
- :mod:`config` – **Data loading** at import time (SWOW JSON, domain clusters, cue map) and UI constants.

Data & Control Flow
-------------------

``config`` (loads once) → ``graph`` (builds nodes/edges) → ``view`` (turns them into Cytoscape elements via ``ui.cygraph``) → **Dash** renders canvases and interactive tables.

.. code-block:: text

    config  ──►  graph  ──►  view  ──►  ui  ──► dash_cytoscape
      ▲                        │
      └────────────── utils ◄──┘

Key Responsibilities (per file)
--------------------------------

``app.py`` (this file)
~~~~~~~~~~~~~~~~~~~~~~

- Entrypoint, WSGI server export.
- Title, CSS, and Cytoscape layout registration.
- No data logic; keeps concerns separated.

``view.py`` (UI & Callbacks)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- **Layout**: header card, controls (cue dropdown, language checklist), three-panel graph grid, edge-detail table.
- **Callbacks**:
  - ``update_graphs(cue, langs)`` → renders per-language panels + **Common** panel and a **domain legend**.
  - ``display_gemini(tapEdgeData)`` → deduplicates and formats the Gemini-pair table with POS, gloss, and domain.

.. autofunction:: view.make_layout
.. autofunction:: view.register_callbacks

``graph.py`` (Graph Construction)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- ``graph_for_lang``: builds **native-label** node set + edges for a language; clusters cue senses to collaborator-defined domains.
- ``build_graph``: composes **Common** overlaps and per-language **Unique** panels using:
  1) **BN-ID** matches, 2) **lemma-set** overlaps, 3) **attachment of leftovers**.

.. autofunction:: graph.graph_for_lang
.. autofunction:: graph.build_graph

``ui.py`` (Styling & Widgets)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Central Cytoscape stylesheet (node/edge classes: ``.cue``, ``.common``, ``.EN`` ``.ZH`` ``.NL``).
- ``cygraph`` helper that sets layout (``cose-bilkent``), zoom bounds, container styling.

.. autofunction:: ui.cygraph

``utils.py`` (Text Utilities)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- English lemma extraction from translation tags (``WN:EN:...``).
- Heuristics to pick best EN hints for **ZH/NL** cue and association labels.
- NFKC normalization and simple stemming for EN/NL.

.. autofunction:: utils.english_lemmas
.. autofunction:: utils.best_en_from_translations_for_assoc
.. autofunction:: utils.best_en_from_translations_for_cue

``config.py`` (Configuration & Data)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Absolute paths to input JSON/YAML (replace with env-vars for portability in production).
- Loads:
  - ``DATA`` (enhanced SWOW JSON),
  - ``DOMAIN_CLUSTERS`` (cluster → domain labels),
  - ``CUE_MAP`` + ``CUE_OPTIONS`` (for the dropdown).
- UI constants: palette, language options.

.. automodule:: config
   :members:
   :undoc-members:
   :noindex:

Run & Deploy
------------

Local development
~~~~~~~~~~~~~~~~~

.. code-block:: bash

   # install
   poetry install
   # run
   poetry run python app.py
   # open
   xdg-open http://127.0.0.1:8085/  # or open your browser manually

Production (WSGI)
~~~~~~~~~~~~~~~~~

Expose ``server`` to a WSGI server (e.g., Gunicorn or Waitress):

.. code-block:: bash

   poetry run pip install gunicorn
   poetry run gunicorn app:server --bind 0.0.0.0:8085 --workers 2

Documentation Build & Pages Publish
-----------------------------------

**Build HTML**:

.. code-block:: bash

   poetry run sphinx-build -b html docs docs/_build/html

**Quick publish via ghp-import** (manual Pages deploy to ``gh-pages``):

.. code-block:: bash

   poetry run ghp-import -n -p -f docs/_build/html -b gh-pages
   # then (if needed)
   git push origin gh-pages

**CI alternative (recommended):** use a GitHub Actions workflow that builds Sphinx
and deploys to GitHub Pages on push. See ``.github/workflows/docs.yml``.

Reproducibility & Academic Use
------------------------------

The front-end isolates **visual inference** from **data enhancement**. All cross-lingual overlaps shown in the “Common” panel are derived algorithmically (BN-ID intersections, lemma overlaps, and conservative attachment of leftovers). This separation facilitates **replication** and **ablation** (swap ranking heuristics, change clustering, or toggle EN-hint display) without rewriting the UI.

API Reference (Autodoc)
-----------------------

.. automodule:: app
   :members:
   :undoc-members:

.. automodule:: view
   :members:
   :undoc-members:

.. automodule:: graph
   :members:
   :undoc-members:

.. automodule:: ui
   :members:
   :undoc-members:

.. automodule:: utils
   :members:
   :undoc-members:
