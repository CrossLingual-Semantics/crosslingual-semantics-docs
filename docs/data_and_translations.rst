Data inputs and translation layer
=================================

Runtime JSON inputs
-------------------

The app loads two JSON files (paths configured in ``swow-dash/config.py``):

- ``partial_dash.json`` (``DATA``): cue → language blocks for graph construction
- ``80_cue_word_sense_dict_domain_clusters_input.json`` (``DOMAIN_CLUSTERS``):
  cluster metadata (kept for future extensions)

Cross-linguistic analysis TSVs (tables)
---------------------------------------

Cue slice tables (shared/unique) are read from TSV/CSV outputs produced by the
cross-linguistic analysis pipeline. These are configured in ``config.py`` via
``CSV_MAP`` and selected with ``resolve_csv_for_langs``.

Translation pivot used for Common alignment
-------------------------------------------

The Common graph aligns associations via an English translation pivot ``assoc_en``.

In your workflow, these translations are generated with **Gemini** (and saved into a
translations table/CSV that the app can load). At runtime the app reads the minimal
translations CSV:

- default path: ``/home/ubuntu/kabir/SWOWCompareViz/data/translations_filled.csv``
- toggled by: ``USE_TRANSLATION_CSV`` and ``TRANSLATIONS_CSV_PATH`` (see ``graph.py``)

This file provides per-language mappings:

- cue native → cue English (for labelling)
- assoc native → assoc English (for alignment and label hints)

The resulting ``assoc_en`` values are used as the pivot key for Common intersection.

Preprocessing utilities
-----------------------

Two helper scripts are included to build/complete the translation CSV used by the app:

- ``make_app_translations_csv.py``:
  converts a richer translation export into the minimal schema expected by the app.

- ``export_translations_csv.py``:
  optionally fills missing translations in batch (useful when some Gemini rows are
  missing), producing a completed ``translations_filled.csv``.

These scripts are not required for Dash runtime, but they document how the translation
layer is produced and maintained.
