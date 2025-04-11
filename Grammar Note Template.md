---
secondary_tags:
attribute_tags:
tags:
  - grammarpoint
  - template
---
<%*
//==================================================================
// Obsidian Grammar Point Templater Script (v45 - Multi-Prompt/Target Lists)
// Author: [Your Name/Handle, or leave blank]
// Date: 2025-04-11
// Description: Creates a structured note for Japanese grammar points,
//              optimized for Anki sync with parallel lists for prompts
//              and target sentences.
// Changes:
// - Reverted to multi-value englishSituationPrompt (comma-separated list).
// - Added targetSentencesJP field (comma-separated list) for direct translations.
// - Emphasized matching order in prompts.
// - Updated YAML generation and DataviewJS.
//==================================================================
try { // Add try...catch around the main Templater logic

    // --- Utility Functions ---
    const calculatePhase = (weekNum) => {
        if (weekNum <= 6) return "Phase 1 - Foundation (1-6)";
        if (weekNum <= 12) return "Phase 2 - Reinforcement (7-12)";
        if (weekNum <= 18) return "Phase 3 - Application (13-18)";
        return "Phase 4 - Final Review (19-22)";
    };
    const formatDate = (dateString) => {
        if (!dateString) return "Unknown";
        try {
          const date = new Date(dateString);
          if (isNaN(date.getTime())) return dateString;
          return date.toISOString().split('T')[0];
        } catch (e) { return dateString; }
    };
    const formatYamlList = (items, indent = '  ') => {
        if (!items || items.length === 0) return " []";
        return '\n' + items.map(item => {
            let stringItem = String(item || "").trim(); // Ensure string, trim whitespace
            // Quote if contains special chars or starts with #
            if (stringItem.includes(':') || stringItem.includes('"') || stringItem.includes("'") || stringItem.startsWith('#')) {
                 // Escape existing double quotes within the string
                 stringItem = `"${stringItem.replace(/"/g, '\\"')}"`;
            }
            return `${indent}- ${stringItem}`;
        }).join('\n');
    };
    const quoteYamlString = (value) => {
        const str = String(value || "").replace(/"/g, '\\"');
        return `"${str}"`;
    };


    // --- Main Template Logic ---
    let currentFileName = tp.file.title;
    if (currentFileName.toLowerCase().includes("template")) { currentFileName = ""; }

    // --- Collect User Inputs (MODIFIED) ---
    let titleInput = await tp.system.prompt("[Title] Grammar Point (e.g., ～てしまう)", currentFileName);
    let title = (!titleInput || titleInput.trim() === "") ? tp.file.title : titleInput.trim();
    let jlpt = await tp.system.suggester(["N5", "N4", "N3", "N2", "N1", "Unknown"], ["N5", "N4", "N3", "N2", "N1", "Unknown"], false, "Roughly what JLPT Level is this?") || "Unknown";
    let confidence = await tp.system.suggester(["Low", "Medium", "High"], ["Low", "Medium", "High"], false, "How confidently can you use this grammar?") || "Low";
    let weekInput = await tp.system.prompt("[Week] What week of study is it (number)?");
    let totalWeeksInput = await tp.system.prompt("[Total Weeks] Total weeks in study plan (default: 22)", "22");
    let structureInput = await tp.system.prompt("[Structure] Structure (e.g., V-て form + しまう)");
    let meaningInput = await tp.system.prompt("[Meaning] Core meaning (1-3 words)");
    let contextInput = await tp.system.prompt("[Context] Where encountered? (e.g., Genki L5, Anime Name Ep3)");
    // --- RESTORED/NEW Prompts ---
    let englishPromptsInput = await tp.system.prompt("[Prompts] English Situations (COMMA-separated list)");
    let targetSentencesJPInput = await tp.system.prompt("[Targets] Japanese Translations (COMMA-separated list - MUST match prompt order!)");
    // --- End RESTORED/NEW Prompts ---
    let usageNotesInput = await tp.system.prompt("[Usage Notes] Notes on how to use this grammar (can use Shift+Enter for new lines)");
    let relatedGrammarInput = await tp.system.prompt("[Related Grammar] Add related grammar points (comma separated, no spaces)", "");

    // --- Process Inputs ---
    const englishPromptsArray = (englishPromptsInput && englishPromptsInput.trim() !== "") ? englishPromptsInput.split(',').map(item => item.trim()).filter(Boolean) : [];
    const targetSentencesJPArray = (targetSentencesJPInput && targetSentencesJPInput.trim() !== "") ? targetSentencesJPInput.split(',').map(item => item.trim()).filter(Boolean) : []; // Process new input
    const usageNotesArray = (usageNotesInput && usageNotesInput.trim() !== "") ? usageNotesInput.split('\n').map(item => item.trim()).filter(Boolean) : [];
    const relatedGrammarArray = (relatedGrammarInput && relatedGrammarInput.trim() !== "") ? relatedGrammarInput.split(',').map(item => item.trim()).filter(Boolean) : [];

    // --- Calculations and Formatting ---
    const weekNum = parseInt(weekInput) || 1;
    const totalWeeksNum = parseInt(totalWeeksInput) || 22;
    const phase = calculatePhase(weekNum);
    const now = tp.date.now("YYYY-MM-DD");
    const tags = ["grammar", "japanese", "grammarpoint"];
    if (jlpt && jlpt !== "Unknown") { tags.push(`jlpt-${jlpt.toLowerCase()}`); }

    // --- File Rename ---
    let finalTitle = tp.file.title;
    if (title && title !== tp.file.title) {
      try {
          await tp.file.rename(title);
          finalTitle = title;
          console.log(`Templater: Renamed file to ${finalTitle}`);
      } catch (e) {
          console.error("Templater Error: Renaming file failed:", e);
          new Notice("Error renaming file. Check console.", 5000);
          finalTitle = tp.file.title;
      }
    } else { finalTitle = title; }


    // --- Generate YAML Frontmatter (MODIFIED) ---
    const yamlOutput = `---
aliases:${formatYamlList([finalTitle]) || " []"}
tags:${formatYamlList(tags)}
confidence: ${confidence}
created: ${now}
lastReviewed: ${now}
phase: ${quoteYamlString(phase)}
jlpt-level: ${jlpt}
week: ${weekNum}
totalWeeks: ${totalWeeksNum}
structure: ${quoteYamlString(structureInput)}
meaning: ${quoteYamlString(meaningInput)}
context: ${quoteYamlString(contextInput)}
relatedGrammar:${formatYamlList(relatedGrammarArray) || " []"}
englishSituationPrompt:${formatYamlList(englishPromptsArray) || " []"}
targetSentencesJP:${formatYamlList(targetSentencesJPArray) || " []"}
exampleSentences:
  - # Example 1 Japanese
  - # Example 2 Japanese
exampleSentencesEnglish:
  - # Example 1 English
  - # Example 2 English
usageNotes:${formatYamlList(usageNotesArray) || " []"}
ankiExpression: ${quoteYamlString(finalTitle)}
---

`; // End of YAML

    // Add note title below YAML
    const noteBodyStart = `\n\n# ${finalTitle}\n\n`;

    // --- DataviewJS Block (Updated for Lists) ---
    const dataviewJsBlock = `
\`\`\`dataviewjs
// --- DataviewJS Display for Grammar Note (v45) ---
try {
    const currentPage = dv.current();
    if (!currentPage) {
        dv.el("div", "Error: Could not get current page data.");
    } else {
        // --- DataviewJS Helpers ---
        const getProp = (key, defaultValue = "") => dv.current()[key] ?? defaultValue;
        const formatDate = (dateStr) => {
             if (!dateStr) return "Unknown";
             try { if (/^\\d{4}-\\d{2}-\\d{2}$/.test(dateStr)) return dateStr; const date = new Date(dateStr); return isNaN(date.getTime()) ? String(dateStr) : date.toISOString().split('T')[0]; } catch (e) { return String(dateStr); }
        };
        const createInfoItem = (container, label, value, styles = {}, valueClass = "info-value", placeholderText = "Not set. Edit frontmatter.") => {
            const item = dv.el("div", "", { cls: "info-item" });
            dv.el("div", label, { cls: "info-label" }, item);
            if (value && String(value).trim()) {
                 dv.el("div", String(value), { cls: valueClass, style: styles }, item);
            } else {
                 dv.el("div", placeholderText, { cls: \`\${valueClass} placeholder-text\`, style: styles }, item);
            }
            container.appendChild(item);
        };
         const createListInfoItem = (container, label, values, styles = {}, valueClass = "info-value", itemClass = "list-item", placeholderText = "Not set. Edit frontmatter.") => {
            const item = dv.el("div", "", { cls: "info-item" });
            dv.el("div", label, { cls: "info-label" }, item);
            const listContainer = dv.el("div", "", { cls: valueClass, style: styles }, item);
            if (values && Array.isArray(values) && values.length > 0 && values.some(v => v && String(v).trim())) {
                const ul = dv.el("ul", "", { cls: "dataview-list" }, listContainer);
                values.forEach(val => {
                    if (val && String(val).trim()) {
                        dv.el("li", String(val), { cls: itemClass }, ul);
                    }
                });
            } else {
                 dv.el("div", placeholderText, { cls: \`placeholder-text\`, style: styles }, listContainer);
            }
            container.appendChild(item);
        };
        const createBadge = (container, text, baseClass, modifier) => {
             if (text && text !== 'N/A' && text !== 'Unknown') { const modifierStr = String(modifier || "").toLowerCase(); dv.el("span", text, { cls: \`\${baseClass} \${baseClass}-\${modifierStr}\` }, container); }
        };
        const createExternalLink = (container, text, baseUrl, query) => {
            dv.el("a", text, { cls: "external-link", href: baseUrl + encodeURIComponent(query || ""), target: "_blank" }, container);
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
        const usageNotes = getProp('usageNotes', []);
        const createdDate = formatDate(getProp('created'));
        const reviewedDate = formatDate(getProp('lastReviewed'));
        // --- Get Prompt/Target Lists ---
        const prompts = getProp('englishSituationPrompt', []);
        const targets = getProp('targetSentencesJP', []);

        // --- Create Display Elements ---
        const cardContainer = dv.el("div", "", { cls: "grammar-card" });
        // Header
        const header = dv.el("div", "", { cls: "grammar-header" }, cardContainer);
        dv.el("h1", fileName, { cls: "grammar-title" }, header);
        const badges = dv.el("div", "", { cls: "badges-container" }, header);
        createBadge(badges, jlptLevel, "jlpt-badge", jlptLevel);
        createBadge(badges, confidence, "confidence-badge", confidence);

        // Progress Bar
        const progressSection = dv.el("div", "", { cls: "progress-section" }, cardContainer);
        const progressHeader = dv.el("div", "", { cls: "progress-header" }, progressSection);
        dv.el("span", "Study Progress : ", { cls: "progress-title" }, progressHeader);
        dv.el("span", \`Week \${weekNum} of \${totalWeeks}\`, { cls: "progress-stats" }, progressHeader);
        const progressHtml = \`<div class="progress-container"><progress value="\${progressPercentage}" max="100" class="custom-progress"></progress><div class="progress-label">\${progressPercentage}%</div></div>\`;
        dv.el("div", progressHtml, { cls: "html-progress-bar" }, progressSection);


        // Main Content Area
        const contentContainer = dv.el("div", "", { cls: "grammar-content" }, cardContainer);
        const primaryContent = dv.el("div", "", { cls: "primary-content" }, contentContainer);

        // --- Display Prompts and Targets as Lists ---
        createListInfoItem(primaryContent, "English Prompts (Anki Front)", prompts, {}, "info-value", "list-item", "No prompts set.");
        createListInfoItem(primaryContent, "Target Sentences (Anki Back)", targets, {}, "info-value target-sentence-jp", "list-item target-jp-item", "No target sentences set.");


        // Meaning & Structure
        createInfoItem(primaryContent, "Meaning", meaning, { "font-size": "1.2em", "font-weight": "500" }, "info-value", "No meaning set.");
        createInfoItem(primaryContent, "Structure", structure, { "font-size": "1.1em", "font-family": "monospace" }, "info-value code-block", "No structure set.");

        // Examples Table
        const examplesItem = dv.el("div", "", { cls: "info-item examples-info" }, primaryContent);
        dv.el("div", "Additional Examples", { cls: "info-label" }, examplesItem);
        const examplesValueContainer = dv.el("div", "", { cls: "info-value" }, examplesItem);
        const validExamples = Array.isArray(exampleSentences) ? exampleSentences.filter(s => s && String(s).trim() && !String(s).trim().startsWith('#')) : [];
        const validTranslations = Array.isArray(exampleSentencesEnglish) ? exampleSentencesEnglish.filter(s => s && String(s).trim() && !String(s).trim().startsWith('#')) : [];
        if (validExamples.length > 0 || validTranslations.length > 0) {
            const table = dv.el("table", "", { cls: "example-sentences-table" }, examplesValueContainer);
            const thead = dv.el("thead", "", {}, table); const hr = dv.el("tr", "", {}, thead);
            dv.el("th", "#", { cls: "example-number-header" }, hr); dv.el("th", "Japanese", { cls: "example-japanese-header" }, hr);
            dv.el("th", "English", { cls: "example-english-header" }, hr);
            const tbody = dv.el("tbody", "", {}, table);
            for (let i = 0; i < Math.max(validExamples.length, validTranslations.length); i++) {
                 const row = dv.el("tr", "", { cls: "example-row" }, tbody);
                 dv.el("td", (i + 1).toString(), { cls: "example-number" }, row);
                 dv.el("td", validExamples[i] || "-", { cls: "example-japanese" + (!validExamples[i] ? " missing-example" : "") }, row);
                 dv.el("td", validTranslations[i] || "-", { cls: "example-english" + (!validTranslations[i] ? " missing-example" : "") }, row);
             }
        } else {
            dv.el("div", "No additional examples added yet. Edit frontmatter.", { cls: "example-placeholder placeholder-text" }, examplesValueContainer);
        }


        // Usage Notes
        const usageItem = dv.el("div", "", { cls: "info-item usage-info" }, primaryContent);
        dv.el("div", "Usage Notes", { cls: "info-label" }, usageItem);
        const usageValueContainer = dv.el("div", "", { cls: "info-value usage-notes-container" }, usageItem);
        const currentUsageNotes = Array.isArray(usageNotes) ? usageNotes : (usageNotes ? [usageNotes] : []);
        const validUsageNotes = currentUsageNotes.filter(note => note && String(note).trim());
        if (validUsageNotes.length > 0) {
             validUsageNotes.forEach((note, i) => { dv.el("div", \`\${i + 1}. \${note}\`, { cls: "usage-note" }, usageValueContainer); });
        } else {
             dv.el("div", "No usage notes added yet. Edit frontmatter.", { cls: "usage-placeholder placeholder-text" }, usageValueContainer);
        }


        // Secondary Content
        const secondaryContent = dv.el("div", "", { cls: "secondary-content" }, contentContainer);
        const studyInfoItem = dv.el("div", "", { cls: "info-item study-info" }, secondaryContent);
        dv.el("div", "Study Info", { cls: "info-label" }, studyInfoItem);
        const studyValueContainer = dv.el("div", "", { cls: "info-value meta-grid" }, studyInfoItem);
        if (phase) dv.el("span", \`🏷️ Phase: \${phase}\`, {}, studyValueContainer);
        dv.el("span", \`📚 Week: \${weekNum} of \${totalWeeks}\`, {}, studyValueContainer);

        if (context && context.trim()) {
             dv.el("div", \`📝 Context: \${context}\`, { cls: "context-value" }, studyValueContainer);
        } else {
             dv.el("div", '📝 Context: <span class="placeholder-text">No context set. Edit frontmatter.</span>', { cls: "context-value" }, studyValueContainer);
        }

        const validRelated = Array.isArray(relatedGrammar) ? relatedGrammar.filter(g => g && String(g).trim()) : [];
        const relatedItem = dv.el("div", "", { cls: "related-grammar" }, secondaryContent);
        dv.el("span", "🔄 Related Grammar: ", {}, relatedItem);
        if (validRelated.length > 0) {
            const links = validRelated.map(item => \`[[\${item}]]\`).join(", ");
            dv.el("span", links, {}, relatedItem);
        } else {
             dv.el("span", "No related grammar linked yet. Edit frontmatter.", { cls: "placeholder-text" }, relatedItem);
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
    const finalOutput = yamlOutput + noteBodyStart + dataviewJsBlock;
    // Set the template content
    tR = finalOutput;

} catch (templaterError) {
    console.error("Templater Script Error:", templaterError);
    new Notice("Error running Templater script. Check console.", 5000);
    tR = `# Templater Error\n\nAn error occurred while running the template script:\n\n\`\`\`\n${templaterError.message}\n\`\`\`\n\nPlease check the developer console (Ctrl+Shift+I) for more details.`;
}
%>
