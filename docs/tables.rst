Tables and downloads
====================

The application provides two table views:

1) **Edge details**: Gemini-pairs for a clicked edge.
2) **Cue slice tables**: shared/unique association rows from cross-linguistic TSVs.

Edge details table (Gemini-pairs)
---------------------------------

When the user clicks an edge in any panel, ``view.py:display_gemini`` renders an
edge-inspection table.

The table is intentionally designed as an “evidence view”:

- It shows cue and association synset IDs (BN), glosses, and POS when available.
- It accepts both legacy and DB-shaped Gemini entries via ``_unpack_side``.
- It deduplicates rows so repeated Gemini-pair rows do not overwhelm the reader.

Cue slice tables (shared vs unique)
-----------------------------------

Cue slice tables are derived from the precomputed cross-linguistic TSVs rather than
from the graph structure itself. This ensures:

- reproducibility against the exported analysis tables, and
- decoupling of visual layout decisions from quantitative tables.

The routing to the correct TSV is determined by the selected language set in
``config.py`` (``resolve_csv_for_langs``).

Grouping is performed by the ``status`` column in the TSV:

- ``shared``
- ``unique_to_l1``
- ``unique_to_l2``
- (and optionally ``unique_to_l3`` in tri-lingual mode)

Implementation details live in ``csv_slices.py``:

- ``slice_for_cue`` filters the TSV rows for the selected cue.
- ``slice_groups_for_cue`` splits by status.
- ``slice_for_edge`` additionally narrows the cue slice to the clicked edge’s
  association(s), matching on the per-language response columns.

Downloads
---------

Two download buttons export the current slices as CSV:

- **Download cue slice (CSV)** uses ``slice_for_cue``.
- **Download edge slice (CSV)** uses ``slice_for_edge``.

This is implemented in ``view.py:download_slices`` and guarantees that the exported
CSV corresponds exactly to what the UI is currently showing.
