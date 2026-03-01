# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is an Obsidian vault for a second brain knowledge management system. It follows the Zettelkasten-inspired approach of linking atomic notes to build a personal knowledge graph.

## Vault Structure

The vault follows a modified PARA/Areas organizational structure:

- **Root level**: Top-level domain folders (e.g., "AI Engineering")
- **Areas**: Subdirectories for ongoing areas of responsibility within each domain
- **Notes**: Individual markdown files containing atomic concepts

Current structure:
- `AI Engineering/Areas/` - Notes related to AI and engineering topics

## Obsidian Best Practices for This Vault

### Note Creation
- Use descriptive, searchable titles without dates or prefixes
- Keep notes atomic - one concept per note
- Create new notes in appropriate domain/area folders
- Use sentence case for note titles (e.g., "Context Engineering" not "context-engineering")

### Linking Strategy
- Use `[[wikilinks]]` for internal note connections
- Create bidirectional links to build knowledge graph
- Link to related concepts even if target note doesn't exist yet (Obsidian will create stub)
- Favor explicit links over tags for conceptual relationships

### Metadata and Properties
- Add YAML frontmatter only when needed for specific use cases
- Keep metadata minimal and purposeful
- Tags are enabled but should be used sparingly compared to links

### Note Types

There are two complementary note types used in this vault:

1. **Concept note** — atomic, definitional. Captures a single idea: what it is, why it matters, key principles, and links to related concepts. Title is the concept name (e.g., "Tool Calling"). This is the canonical note for that idea and should remain concise.

2. **Source summary note** — longer, narrative. Summarizes a specific article, thread, talk, or resource in detail. Title reflects the source (e.g., "Lessons from Building Claude Code - Seeing like an Agent"). Always includes a `## References` section linking back to the original URL.

When processing a new source:
- Create a **source summary note** with a full breakdown of the content
- Identify the key concepts it introduces or expands on
- Create or update the corresponding **concept notes**, linking to the source summary
- The source summary should link back to all related concept notes via `[[wikilinks]]`

### Content Guidelines
- Write in complete sentences and paragraphs
- Define concepts clearly at the beginning of notes
- Build upon definitions with examples, applications, and connections
- Use markdown formatting: headers for sections, bold for key terms, lists for structure
- When a note is based on articles or external sources, add a `## References` section at the bottom with markdown links to the source URLs

### File Organization
- Don't create deeply nested folder structures (max 2-3 levels)
- Use Areas for ongoing topics, not projects with defined endpoints
- Keep related notes discoverable through links, not just folder proximity

## Enabled Obsidian Features

Core plugins active:
- Daily notes (for journaling/logging)
- Templates (for consistent note structure)
- Graph view (for visualizing connections)
- Backlinks and outgoing links (for relationship tracking)
- Canvas (for visual knowledge mapping)
- Search and tag pane
- Properties and outline views

## Working with This Vault

When creating or modifying notes:
1. Check existing notes to avoid duplication
2. Create links to related concepts liberally
3. Follow existing naming patterns (sentence case, descriptive)
4. Place notes in appropriate domain/area folders
5. Keep content focused and atomic

When organizing:
- Prefer creating new Areas over nested subfolders
- Use graph view to identify orphaned notes that need connections
- Avoid creating README or index files (use Obsidian's graph and search instead)
