==============================
🌐 SWOW Enhancement
==============================

Introduction
============

The **Small World of Words (SWOW) dataset** captures **word associations** through a **cue-response paradigm**. Each cue word elicits **three responses** (R1, R2, R3) from participants, revealing semantic relationships between words.

**Input CSV Example:**

.. code-block:: csv

    cue,R1,R2,R3
    music,notes,band,rhythm
    book,pages,author,story

**Goal:** Enhance SWOW by adding **linguistic dimensions** to both cues and associations:

1. **POS:** Part of Speech (e.g., NOUN, VERB)
2. **Relations:** Lexical relations (e.g., synonym, hypernym)
3. **Sense:** BabelNet sense (ID and gloss description)
4. **SME Annotations:** Expert-provided MacroCode and MicroCode relations

---

Core Classes
============

The enhancement pipeline relies on three core classes: **Dimension**, **Association**, and **Cue**.

1. **Dimension Class:** Represents metadata (POS, Relation, Sense, SME annotations).
2. **Association Class:** Represents words associated with cues.
3. **Cue Class:** Represents the main word with associations.

---

📏 **1. Dimension Class**
-------------------------

**File:** `swow_pipeline/models/dimension.py`

**Purpose:** Represents metadata associated with a word (POS, Relation, Sense, SME Annotations).

**Key Code:**

.. code-block:: python

    class Dimension:
        """Base class for linguistic dimensions like POS, Relation, and Sense."""
        def __init__(self, name: str, value: str):
            self.name = name
            self.value = value

        def to_dict(self):
            return {"name": self.name, "value": self.value}

    class POS(Dimension):
        """Part-of-Speech Dimension."""
        def __init__(self, value: str):
            super().__init__("POS", value)

    class Relation(Dimension):
        """Lexical Relation Dimension."""
        def __init__(self, value: str):
            super().__init__("Relation", value)

    class Sense(Dimension):
        """Sense Mapping Dimension (BabelNet ID and gloss)."""
        def __init__(self, value: str):
            super().__init__("Sense", value)

    class MacroCode(Dimension):
        """SME-provided MacroCode relations."""
        def __init__(self, value: str):
            super().__init__("MacroCode", value)

    class MicroCode(Dimension):
        """SME-provided MicroCode relations."""
        def __init__(self, value: str):
            super().__init__("MicroCode", value)

**Example Output for "music":**

.. code-block:: json

    [
      { "name": "POS", "value": "NOUN" },
      { "name": "Relation", "value": "has_part" },
      { "name": "Sense", "value": "bn:00060282n: sound art form" },
      { "name": "MacroCode", "value": "Situation" },
      { "name": "MicroCode", "value": "S3 Object" }
    ]

---

🔗 **2. Association Class**
---------------------------

**File:** `swow_pipeline/models/association.py`

**Purpose:** Represents an **association** word linked to a cue, along with its **frequency** and **dimensions**.

**Key Code:**

.. code-block:: python

    from .dimension import Dimension

    class Association:
        """Association class representing word associations with dimensions."""
        def __init__(self, word: str, frequency: int):
            self.word = word
            self.frequency = frequency
            self.dimensions = []

        def add_dimension(self, dimension: Dimension):
            if dimension not in self.dimensions:
                self.dimensions.append(dimension)

        def to_dict(self):
            return {
                "word": self.word,
                "frequency": self.frequency,
                "dimensions": [dim.to_dict() for dim in self.dimensions]
            }

---

🎧 **3. Cue Class**
-------------------

**File:** `swow_pipeline/models/cue.py`

**Purpose:** Represents a **cue word**, its **associations**, and enriched **dimensions**.

**Key Code:**

.. code-block:: python

    from .association import Association
    from .dimension import Dimension

    class Cue:
        """Cue class representing a cue word with associations and dimensions."""
        def __init__(self, word: str):
            self.word = word
            self.associations = {}
            self.dimensions = []

        def add_association(self, word: str, frequency: int):
            if word not in self.associations:
                self.associations[word] = Association(word, frequency)

        def add_dimension(self, dimension: Dimension):
            if dimension not in self.dimensions:
                self.dimensions.append(dimension)

        def to_dict(self):
            return {
                "word": self.word,
                "associations": {word: assoc.to_dict() for word, assoc in self.associations.items() if assoc.dimensions},
                "dimensions": [dim.to_dict() for dim in self.dimensions]
            }

---

Filtering Enhancements
=======================

**New Feature:** We now **filter associations** that do not contain any dimensions.

**Key Code:**

.. code-block:: python

    def filter_associations(cue_dict):
        """Remove associations with no dimensions."""
        for cue in cue_dict.values():
            cue.associations = {word: assoc for word, assoc in cue.associations.items() if assoc.dimensions}
        return cue_dict

**Example:** Before Filtering:

.. code-block:: json

    "associations": {
        "traditions": {"word": "traditions", "frequency": 3, "dimensions": []},
        "society": {"word": "society", "frequency": 21, "dimensions": [{"POS": "NOUN"}]} 
    }

**After Filtering:**

.. code-block:: json

    "associations": {
        "society": {"word": "society", "frequency": 21, "dimensions": [{"POS": "NOUN"}]} 
    }

---

🔄 **Final Execution Command:**

.. code-block:: bash

    python main.py --input data/swow_sample.csv --output_json output/clean_swow.json --output_json_with_metadata output/enhanced_swow.json --sample_size 100 --annotations data/coding.csv

---

This documentation should now cover **all the enhancements** clearly! 🚀

