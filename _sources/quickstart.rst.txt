Quickstart
==========

Run the app
-----------

From the ``swow-dash/`` folder:

.. code-block:: bash

   python app.py

By default the app runs on port ``8087``.

Environment variables
---------------------

You can override data locations without changing code:

- ``SWOW_DATA_DIR``: base folder for inputs
- ``DATA_PATH``: path to ``partial_dash.json``
- ``CLUSTER_PATH``: path to the domain cluster JSON
- ``CSV_ZH_EN``, ``CSV_NL_EN``, ``CSV_ZH_NL_EN``: analysis TSVs for tables
- ``USE_TRANSLATION_CSV``: set to ``0``/``false`` to disable translation CSV usage
- ``TRANSLATIONS_CSV_PATH``: path to ``translations_filled.csv``

Notes on expected behaviour
---------------------------

- English is always selected (the UI enforces EN as an anchor language).
- The Common panel shows associations shared by *all* selected languages, using the
  translation pivot described in ``data_and_translations``.
- Clicking an edge reveals Gemini-pairs in the Edge details table.
- Download buttons export the current cue/edge slices.
