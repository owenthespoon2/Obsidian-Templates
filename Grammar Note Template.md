---
secondary_tags:
attribute_tags:
tags:
  - grammarpoint
  - template
---
<%*
//==================================================================
// Obsidian Grammar Point Templater Script (v40)
// Author: [Your Name/Handle, or leave blank]
// Date: 2025-04-10
// Description: Creates a structured note for Japanese grammar points,
//              including YAML frontmatter and a DataviewJS display.
// Changes:
// - General code cleanup, added comments, switched let->const.
// - Minor refactor of DataviewJS display logic for readability.
// - Fixed DataviewJS "Illegal return statement" error.
// - Removed redundant ankiReading and ankiMeaning from YAML output.
// - Removed Anki CSV Export functionality.
//==================================================================
try { // Add try...catch around the main Templater logic

    // --- Utility Functions ---

    /**
     * Calculates the study phase based on the week number.
     * @param {number} weekNum - The current week number.
     * @returns {string} The calculated study phase description.
     */
    const calculatePhase = (weekNum) => {
      if (weekNum <= 6) return "Phase 1 - Foundation (1-6)";
      if (weekNum <= 12) return "Phase 2 - Reinforcement (7-12)";
      if (weekNum <= 18) return "Phase 3 - Application (13-18)";
      return "Phase 4 - Final Review (19-22)";
    };

    /**
     * Formats a date string into YYYY-MM-DD format.
     * @param {string} dateString - The date string to format.
     * @returns {string} The formatted date or the original string if invalid.
     */
    const formatDate = (dateString) => {
      if (!dateString) return "Unknown";
      try {
        const date = new Date(dateString);
        if (isNaN(date.getTime())) return dateString;
        return date.toISOString().split('T')[0];
      } catch (e) { return dateString; }
    };

    /**
     * Formats an array into a YAML list string.
     * @param {Array<string>} items - Array of strings.
     * @param {string} indent - Indentation string (default '  ').
     * @returns {string} Formatted YAML list or '[]'.
     */
    const formatYamlList = (items, indent = '  ') => {
        if (!items || items.length === 0) return "[]";
        return '\n' + items.map(item => {
            let stringItem = String(item);
            // Quote if contains problematic chars like colon or existing quotes
            if (stringItem.includes(':') || stringItem.includes('"') || stringItem.includes("'")) {
                 stringItem = `"${stringItem.replace(/"/g, '\\"')}"`;
            }
            return `${indent}- ${stringItem}`;
        }).join('\n');
    };

    // --- Main Template Logic ---

    let currentFileName = tp.file.title;
    // Avoid suggesting the template name itself as the title
    if (currentFileName.toLowerCase().includes("template")) {
        currentFileName = "";
    }

    // --- Collect User Inputs ---
    let titleInput = await tp.system.prompt("[Title] Grammar Point (e.g., ～てしまう)", currentFileName);
    // Use current filename if prompt is empty, otherwise use input
    let title = (!titleInput || titleInput.trim() === "") ? tp.file.title : titleInput.trim();

    let jlpt = await tp.system.suggester(["N5", "N4", "N3", "N2", "N1", "Unknown"], ["N5", "N4", "N3", "N2", "N1", "Unknown"], false, "Roughly what JLPT Level is this?") || "Unknown";
    let confidence = await tp.system.suggester(["Low", "Medium", "High"], ["Low", "Medium", "High"], false, "How confidently can you use this grammar?") || "Low";
    let weekInput = await tp.system.prompt("[Week] What week of study is it (number)?");
    let totalWeeksInput = await tp.system.prompt("[Total Weeks] Total weeks in study plan (default: 22)", "22");
    let structureInput = await tp.system.prompt("[Structure] Structure (e.g., V-て form + しまう)");
    let meaningInput = await tp.system.prompt("[Meaning] Core meaning (1-3 words)");
    let contextInput = await tp.system.prompt("[Context] Where encountered? (e.g., Genki L5, Anime Name Ep3)");
    let englishPromptsInput = await tp.system.prompt("[Prompts] English Situation Prompts (comma-separated)");
    let usageNotesInput = await tp.system.prompt("[Usage Notes] Notes on how to use this grammar (can use Shift+Enter for new lines)");
    let relatedGrammarInput = await tp.system.prompt("[Related Grammar] Add related grammar points (comma separated, no spaces)", "");

    // --- Process Inputs ---
    const englishPromptsArray = (englishPromptsInput && englishPromptsInput.trim() !== "")
        ? englishPromptsInput.split(',').map(item => item.trim()).filter(Boolean)
        : [];
    const usageNotesArray = (usageNotesInput && usageNotesInput.trim() !== "")
        ? usageNotesInput.split('\n').map(item => item.trim()).filter(Boolean)
        : [];
    const relatedGrammarArray = (relatedGrammarInput && relatedGrammarInput.trim() !== "")
        ? relatedGrammarInput.split(',').map(item => item.trim()).filter(Boolean)
        : [];

    // --- Calculations and Formatting ---
    const weekNum = parseInt(weekInput) || 1;
    const totalWeeksNum = parseInt(totalWeeksInput) || 22;
    const phase = calculatePhase(weekNum);
    const now = tp.date.now("YYYY-MM-DD");
    const tags = ["grammar", "japanese", "grammarpoint"];
    if (jlpt && jlpt !== "Unknown") {
        tags.push(`jlpt-${jlpt.toLowerCase()}`);
    }

    // --- File Rename ---
    let finalTitle = tp.file.title; // Start with current title
    // Only rename if prompted title is different from current filename and not empty
    if (title && title !== tp.file.title) {
      try {
          await tp.file.rename(title);
          finalTitle = title; // Use the prompted title if rename succeeds
          console.log(`Templater: Renamed file to ${finalTitle}`);
      } catch (e) {
          console.error("Templater Error: Renaming file failed:", e);
          new Notice("Error renaming file. Check console.", 5000);
          // Keep original title if rename fails
          finalTitle = tp.file.title;
      }
    } else {
        // Ensure finalTitle reflects the intended title even if no rename happened
        finalTitle = title;
    }


    // --- Generate YAML Frontmatter ---
    const yamlOutput = `---
aliases:
  - ${finalTitle}
tags:${formatYamlList(tags)}
confidence: ${confidence}
created: ${now}
lastReviewed: ${now}
phase: ${phase}
jlpt-level: ${jlpt}
week: ${weekNum}
totalWeeks: ${totalWeeksNum}
structure: ${structureInput || ""}
meaning: ${meaningInput || ""}
context: ${contextInput || ""}
relatedGrammar:${formatYamlList(relatedGrammarArray)}
englishSituationPrompt:${formatYamlList(englishPromptsArray)}
exampleSentences:
  - # Example 1 Japanese
  - # Example 2 Japanese
exampleSentencesEnglish:
  - # Example 1 English
  - # Example 2 English
usageNotes:${formatYamlList(usageNotesArray)}
ankiExpression: ${finalTitle} # Keep this - used by Python script
---

# ${finalTitle}

`; // End of YAML and start of note body

    // --- DataviewJS Block ---
    const dataviewJsBlock = `
\`\`\`dataviewjs
// --- DataviewJS Display for Grammar Note (v40) ---
try {
    const currentPage = dv.current();
    if (!currentPage) {
        dv.el("div", "Error: Could not get current page data.");
    } else {
        // --- DataviewJS Helpers ---
        const getProp = (key, defaultValue = "") => dv.current()[key] ?? defaultValue;
        const formatDate = (dateStr) => { /* ... (same as above) ... */
            if (!dateStr) return "Unknown";
            try {
                if (/^\\d{4}-\\d{2}-\\d{2}$/.test(dateStr)) return dateStr;
                const date = new Date(dateStr);
                return isNaN(date.getTime()) ? String(dateStr) : date.toISOString().split('T')[0];
            } catch (e) { return String(dateStr); }
        };
        const createInfoItem = (container, label, value, styles = {}, valueClass = "info-value") => { /* ... (same as below) ... */
            const item = dv.el("div", "", { cls: "info-item" });
            dv.el("div", label, { cls: "info-label" }, item);
            dv.el("div", String(value || ""), { cls: valueClass, style: styles }, item);
            container.appendChild(item);
        };
        const createBadge = (container, text, baseClass, modifier) => { /* ... (same as below) ... */
             if (text && text !== 'N/A' && text !== 'Unknown') {
                dv.el("span", text, { cls: \`\${baseClass} \${baseClass}-\${modifier.toLowerCase()}\` }, container);
            }
        };
         const createExternalLink = (container, text, baseUrl, query) => { /* ... (same as below) ... */
            dv.el("a", text, { cls: "external-link", href: baseUrl + encodeURIComponent(query), target: "_blank" }, container);
        };

        // --- Extract Data ---
        const fileName = getProp('file.name', 'Unnamed Note');
        const confidence = getProp('confidence', 'Low');
        const jlptLevel = getProp('jlpt-level', 'N/A');
        const weekNum = parseInt(getProp('week', 1));
        const totalWeeks = parseInt(getProp('totalWeeks', 22));
        const progressPercentage = Math.min(100, Math.round((weekNum / totalWeeks) * 100));
        const meaning = getProp('meaning');
        const structure = getProp('structure');
        const phase = getProp('phase');
        const context = getProp('context');
        const relatedGrammar = getProp('relatedGrammar', []);
        const exampleSentences = getProp('exampleSentences', []);
        const exampleSentencesEnglish = getProp('exampleSentencesEnglish', []);
        const usageNotes = getProp('usageNotes', []); // Will be array from YAML
        const createdDate = formatDate(getProp('created'));
        const reviewedDate = formatDate(getProp('lastReviewed'));

        // --- Create Display Elements ---
        const cardContainer = dv.el("div", "", { cls: "grammar-card" });

        // Header
        const header = dv.el("div", "", { cls: "grammar-header" }, cardContainer);
        dv.el("h1", fileName, { cls: "grammar-title" }, header);
        const badges = dv.el("div", "", { cls: "badges-container" }, header);
        createBadge(badges, jlptLevel, "jlpt-badge", jlptLevel);
        createBadge(badges, confidence, "confidence-badge", confidence);
        // Anki Export Button Removed

        // Progress Bar
        const progressSection = dv.el("div", "", { cls: "progress-section" }, cardContainer);
        // ... (progress bar rendering unchanged) ...
        const progressHeader = dv.el("div", "", { cls: "progress-header" }, progressSection);
        dv.el("span", "Study Progress : ", { cls: "progress-title" }, progressHeader);
        dv.el("span", \`Week \${weekNum} of \${totalWeeks}\`, { cls: "progress-stats" }, progressHeader);
        const progressHtml = \`<div class="progress-container"><progress value="\${progressPercentage}" max="100" class="custom-progress"></progress><div class="progress-label">\${progressPercentage}%</div></div>\`;
        dv.el("div", progressHtml, { cls: "html-progress-bar" }, progressSection);


        // Main Content Area
        const contentContainer = dv.el("div", "", { cls: "grammar-content" }, cardContainer);
        const primaryContent = dv.el("div", "", { cls: "primary-content" }, contentContainer);

        // Meaning & Structure
        if (meaning) createInfoItem(primaryContent, "Meaning", meaning, { "font-size": "1.2em", "font-weight": "500" });
        if (structure) createInfoItem(primaryContent, "Structure", structure, { "font-size": "1.1em", "font-family": "monospace" }, "info-value code-block");

        // Examples Table
        const examplesItem = dv.el("div", "", { cls: "info-item examples-info" }, primaryContent);
        dv.el("div", "Example Sentences", { cls: "info-label" }, examplesItem);
        const examplesValueContainer = dv.el("div", "", { cls: "info-value" }, examplesItem);
        const validExamples = Array.isArray(exampleSentences) ? exampleSentences.filter(s => s && String(s).trim() && !String(s).trim().startsWith('#')) : [];
        const validTranslations = Array.isArray(exampleSentencesEnglish) ? exampleSentencesEnglish.filter(s => s && String(s).trim() && !String(s).trim().startsWith('#')) : [];
        if (validExamples.length > 0 || validTranslations.length > 0) {
            // ... (table creation logic unchanged) ...
            const table = dv.el("table", "", { cls: "example-sentences-table" }, examplesValueContainer);
            const thead = dv.el("thead", "", {}, table); const hr = dv.el("tr", "", {}, thead);
            dv.el("th", "#", { cls: "example-number-header" }, hr); dv.el("th", "Japanese", { cls: "example-japanese-header" }, hr); dv.el("th", "English", { cls: "example-english-header" }, hr);
            const tbody = dv.el("tbody", "", {}, table);
            for (let i = 0; i < Math.max(validExamples.length, validTranslations.length); i++) {
                const row = dv.el("tr", "", { cls: "example-row" }, tbody);
                dv.el("td", (i + 1).toString(), { cls: "example-number" }, row);
                dv.el("td", validExamples[i] || "-", { cls: "example-japanese" + (!validExamples[i] ? " missing-example" : "") }, row);
                dv.el("td", validTranslations[i] || "-", { cls: "example-english" + (!validTranslations[i] ? " missing-example" : "") }, row);
            }
        } else { dv.el("div", "Add example sentences in the frontmatter (remove # comments).", { cls: "example-placeholder" }, examplesValueContainer); }

        // Usage Notes
        const usageItem = dv.el("div", "", { cls: "info-item usage-info" }, primaryContent);
        dv.el("div", "Usage Notes", { cls: "info-label" }, usageItem);
        const usageValueContainer = dv.el("div", "", { cls: "info-value usage-notes-container" }, usageItem);
        const currentUsageNotes = Array.isArray(usageNotes) ? usageNotes : [usageNotes]; // Ensure array
        const validUsageNotes = currentUsageNotes.filter(note => note && String(note).trim());
        if (validUsageNotes.length > 0) { validUsageNotes.forEach((note, i) => { dv.el("div", \`\${i + 1}. \${note}\`, { cls: "usage-note" }, usageValueContainer); });
        } else { dv.el("div", "Add usage notes in the frontmatter.", { cls: "usage-placeholder" }, usageValueContainer); }

        // Secondary Content (Study Info, Related, Links)
        const secondaryContent = dv.el("div", "", { cls: "secondary-content" }, contentContainer);
        const studyInfoItem = dv.el("div", "", { cls: "info-item study-info" }, secondaryContent);
        dv.el("div", "Study Info", { cls: "info-label" }, studyInfoItem);
        const studyValueContainer = dv.el("div", "", { cls: "info-value meta-grid" }, studyInfoItem);
        if (phase) dv.el("span", \`🏷️ Phase: \${phase}\`, {}, studyValueContainer);
        dv.el("span", \`📚 Week: \${weekNum} of \${totalWeeks}\`, {}, studyValueContainer);
        if (context) dv.el("div", \`📝 Context: \${context}\`, { cls: "context-value" }, studyValueContainer);
        const validRelated = Array.isArray(relatedGrammar) ? relatedGrammar.filter(g => g && String(g).trim()) : [];
        if (validRelated.length > 0) { /* ... (related grammar rendering unchanged) ... */
            const relatedItem = dv.el("div", "", { cls: "related-grammar" }, secondaryContent);
            dv.el("span", "🔄 Related Grammar: ", {}, relatedItem);
            const links = validRelated.map(item => \`[[\${item}]]\`).join(", ");
            dv.el("span", links, {}, relatedItem);
        }

        // External Links
        const linksContainer = dv.el("div", "", { cls: "external-links-container" }, secondaryContent);
        const linksDiv = dv.el("div", "", { cls: "external-links" }, linksContainer);
        createExternalLink(linksDiv, "Jisho", "https://jisho.org/search/", fileName);
        createExternalLink(linksDiv, "Bunpro", "https://bunpro.jp/grammar_points?search=", fileName);
        createExternalLink(linksDiv, "JWA", "https://www.japanesewithanime.com/search?q=", fileName);

        // Review History Footer
        dv.el("div", \`Created: \${createdDate} | Last Reviewed: \${reviewedDate}\`, { cls: "review-history" }, cardContainer);
    } // End of else block for if(currentPage)

} catch (error) {
    dv.el('div', 'Error rendering DataviewJS display: ' + error.message);
    console.error("DataviewJS Error:", error);
}
\`\`\`
`; // End of DataviewJS block string

    // --- Combine YAML and DataviewJS ---
    const finalOutput = yamlOutput + dataviewJsBlock;

    // Set the template content
    tR = finalOutput;

} catch (templaterError) {
    // Catch errors during the Templater prompt/processing phase
    console.error("Templater Script Error:", templaterError);
    new Notice("Error running Templater script. Check console.", 5000);
    tR = `# Templater Error\n\nAn error occurred while running the template script:\n\n\`\`\`\n${templaterError.message}\n\`\`\`\n\nPlease check the developer console (Ctrl+Shift+I) for more details.`;
}
%>
