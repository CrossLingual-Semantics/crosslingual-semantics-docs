Graph construction and visual semantics
======================================

This page describes how the Cytoscape graphs are built and how colours and labels
are defined.

Per-language graphs
-------------------

Per-language graphs are generated in ``swow-dash/graph.py`` by ``graph_for_lang``.
A key design choice is to model **each native cue variant** as its own cue node for
ZH/NL (rather than collapsing variants early). This keeps variant structure visible
and avoids discarding associations.

Node identifiers use a scoped scheme:

- cue nodes:  ``"{LANG}:{native_cue}"``
- assoc nodes: ``"{LANG}:{native_cue}:{bn_or_surface}"``

Associations are added as outgoing edges from cue → assoc, and each edge carries a
payload including:

- the language tag,
- the native cue and assoc surfaces,
- the Gemini-pairs (for later inspection in the edge table).

Deterministic cue colouring
---------------------------

Colours are assigned **per cue-variant** (not per language) to support consistent
cross-panel interpretation.

In ``build_graph`` the palette is stabilised by constructing the cue→colour map over
**all three languages (EN, ZH, NL)** regardless of the user’s selection. This ensures
that the same cue variant is always rendered with the same colour.

.. code-block:: python

   ALL_LANGS = ("EN", "ZH", "NL")
   ...
   global_cue_keys = sorted(global_cue_keys)
   cue_color_global = {key: _PALETTE[i % len(_PALETTE)] for i, key in enumerate(global_cue_keys)}

This is also mirrored in the legend builder in ``view.py`` (``_cue_palette_for_legend``)
to ensure the legend and graph colours cannot drift.

Label policy and translation hints
----------------------------------

For side panels, the app displays bilingual labels for ZH/NL:

- cue nodes:  ``native (EN cue)``
- assoc nodes: ``native (EN translation)``

The English hints come from the translations layer loaded in ``graph.py`` (see
``swow_dash/data_and_translations``). EN nodes are shown as plain English.

Common graph definition (translation-aligned intersection)
---------------------------------------------------------

The Common panel is constructed in ``swow-dash/graph.py`` by aligning associations
through a translation pivot:

- For each language edge, the app retrieves an English association lemma ``assoc_en``.
- These English keys are normalised (lowercased, whitespace collapsed).
- The **Common set** is the intersection of English keys across all selected languages.

Conceptually, this treats ``assoc_en`` as a pivot representation for alignment, while
still preserving the original native association surfaces in the node label.

Gemini-pairs are retained on the edge payload as interpretability evidence, but they
do not define the Common intersection.

Two-language and three-language modes
-------------------------------------

Two-language mode (EN+ZH or EN+NL) anchors Common cue bubbles using the non-English
language when available; three-language mode uses a single combined cue bubble
(``TRI:CUE:...``) whose label concatenates EN / ZH / NL representatives for the cue.

Interaction: focusing cue variants
----------------------------------

The focus/highlight behaviour is implemented in ``view.py`` (``focus_on_cue``):

- Clicking a ZH/NL cue node fades the panel and highlights the cue + outgoing targets.
- The callback also attempts to highlight the corresponding anchor cue in the Common
  panel (when present) using the consistent ID scheme.
