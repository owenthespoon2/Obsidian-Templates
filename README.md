# My Obsidian Configuration Files

This repository stores configuration files, templates, and snippets for my Obsidian vault, primarily focused on Japanese language learning.

## Contents

Currently, this repository contains the following templates:

1.  **Grammar Note Template (`Grammar Note Template vX.md`)**
    * **Description:** A [Templater](https://github.com/SilentVoid13/Templater) script that generates a structured note for Japanese grammar points. It prompts the user for details (JLPT level, meaning, structure, usage context, English situation prompts, etc.), creates YAML frontmatter, and includes a [Dataview](https://github.com/blacksmithgu/obsidian-dataview)JS block to render a nicely formatted display within the note.
    * **Purpose:** Standardizes the creation of grammar notes, making them consistent and easier to process by other tools (like an external Anki sync script).

*(More templates or snippets can be added here later)*

## General Requirements

* **Obsidian:** The core note-taking application.
* **Obsidian Plugins:** Specific templates may require certain community plugins to be installed and enabled. Common ones used here include:
    * **Templater:** For executing template scripts (`<%* ... %>`) and user prompts.
    * **Dataview:** For rendering dynamic information and the styled views within notes (```dataviewjs ... ``` blocks).

## Templates

### Grammar Note Template (`Grammar Note Template vX.md`)

This template helps create detailed notes for Japanese grammar points.

**Required Plugins:**

* **Templater:** To run the script and prompts.
* **Dataview:** To render the display block within the note.

**How to Use:**

1.  **Save:** Place this template file (`.md`) into the folder designated in your Templater plugin settings.
2.  **Trigger:** Create a new note and trigger the template using the Obsidian Command Palette (`Templater: Insert Template`) or a configured Templater hotkey / QuickAdd macro.
3.  **Fill Prompts:** Answer the prompts for Title, JLPT Level, Confidence, Week, Structure, Meaning, Context, English Situation Prompts (comma-separated), Usage Notes (Shift+Enter for new lines), and Related Grammar (comma-separated).
4.  **Add Examples:** After the note is created, manually edit the YAML frontmatter (Properties view or Source Mode) to add specific Japanese example sentences under `exampleSentences:` and their corresponding English translations under `exampleSentencesEnglish:`. Remove the `# placeholder` comments.
    ```yaml
    exampleSentences:
      - 宿題をしてしまいました。 # Replace placeholders
      - お金を全部使ってしまった。
    exampleSentencesEnglish:
      - I (completely) finished my homework. # Replace placeholders
      - I ended up spending all my money.
    ```
5.  **Refine:** Add any further details directly into the note body below the DataviewJS block if desired.

**YAML Fields Generated:**

This template generates the following key YAML frontmatter fields:

* `aliases`: Based on the note title.
* `tags`: Includes `grammarpoint`, `japanese`, `grammar`, and `jlpt-nX`.
* `confidence`, `created`, `lastReviewed`, `phase`, `jlpt-level`, `week`, `totalWeeks`: Study metadata.
* `structure`, `meaning`, `context`: Core grammar info.
* `relatedGrammar`: List of related grammar point note names.
* `englishSituationPrompt`: List of English prompts (used by external Anki sync script for card fronts).
* `exampleSentences`: List for Japanese example sentences (fill manually).
* `exampleSentencesEnglish`: List for English translations (fill manually).
* `usageNotes`: List of notes on usage.
* `ankiExpression`: Set to the note title (used by external Anki sync script as an identifier).

**DataviewJS Display:**

The template includes a `dataviewjs` block that renders a styled summary of the note's frontmatter directly within the note in Live Preview or Reading mode. (Note: You might need a custom CSS snippet in Obsidian for optimal styling of the classes used, like `.grammar-card`, `.jlpt-badge`, etc.).

## Installation

1.  Clone or download this repository.
2.  Copy the desired template `.md` files into your Obsidian vault's designated templates folder (configure this in Templater plugin settings).
3.  Ensure the required plugins (Templater, Dataview) are installed and enabled in Obsidian.

