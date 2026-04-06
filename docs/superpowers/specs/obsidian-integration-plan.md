# Obsidian Integration Plan

Date: 2026-04-06
Status: Planning
Scope: Obsidian vault integration for ZeroClaw

## Executive Summary

This document outlines the plan for integrating Obsidian with ZeroClaw. Obsidian is a local-first, markdown-based personal knowledge management (PKM) application that stores notes as plain markdown files in a "vault" (directory). Unlike cloud-based services like Notion, Obsidian operates entirely on the local filesystem, which presents unique integration opportunities and challenges.

## Background

### What is Obsidian?

- **Local-first**: All data stored as plain markdown files on the local filesystem
- **Markdown-based**: Native markdown with extensions (wikilinks, frontmatter, callouts)
- **Graph-based**: Notes can link to each other using `[[wikilinks]]`
- **Plugin ecosystem**: Extensible through community plugins
- **No official cloud API**: Designed for local use, not remote access

### Key Characteristics

1. **Storage Model**: Vault = directory of markdown files
2. **Linking**: Uses `[[Note Name]]` syntax for internal links
3. **Metadata**: YAML frontmatter for structured data
4. **Attachments**: Images and files stored in vault subdirectories
5. **Plugins**: JavaScript plugins that extend functionality

## Integration Approaches

Based on research, there are three viable integration approaches:

### Approach 1: Direct Filesystem Access (Recommended)

**Description**: Access the Obsidian vault directly through the filesystem as a directory of markdown files.

**Pros**:
- No dependencies on Obsidian being running
- Simple, reliable implementation
- No authentication complexity
- Works with any vault structure
- Fastest performance

**Cons**:
- Must respect Obsidian's file structure conventions
- No access to plugin-specific features
- Must parse markdown ourselves
- No real-time sync with open Obsidian instance

**Implementation Complexity**: Low

### Approach 2: Obsidian Local REST API Plugin

**Description**: Use the community "Local REST API" plugin that provides HTTP endpoints for vault operations.

**Pros**:
- Structured API interface
- Can interact with running Obsidian instance
- Access to some plugin features
- JSON responses

**Cons**:
- Requires plugin installation and configuration
- Requires Obsidian to be running
- API key management
- Plugin maintenance dependency
- Limited to plugin's API surface

**Implementation Complexity**: Medium

### Approach 3: Hybrid Approach

**Description**: Use filesystem access as primary method, with optional REST API plugin support for enhanced features.

**Pros**:
- Best of both worlds
- Graceful degradation
- Maximum flexibility

**Cons**:
- More complex implementation
- More testing surface
- Configuration complexity

**Implementation Complexity**: High

## Recommended Approach

**Start with Approach 1 (Direct Filesystem Access)**, then optionally add Approach 2 support in a future iteration.

### Rationale

1. **Simplicity**: Filesystem access is simpler and more reliable
2. **Independence**: No dependency on Obsidian being running or plugins
3. **Universality**: Works with any Obsidian vault out of the box
4. **Performance**: Direct file access is faster than HTTP
5. **Security**: No network exposure, no API keys to manage
6. **Incremental**: Can add REST API support later if needed

## Technical Design

### Configuration Schema

```toml
[obsidian]
enabled = true
vault_path = "/Users/username/Documents/ObsidianVault"
# Optional: multiple vaults
vaults = [
    { name = "personal", path = "/Users/username/Documents/Personal" },
    { name = "work", path = "/Users/username/Documents/Work" }
]

# Optional: REST API plugin support (future)
# rest_api_enabled = false
# rest_api_url = "https://127.0.0.1:27124"
# rest_api_key = "..."
```

### Core Operations

1. **Search Notes**
   - Search by content (full-text)
   - Search by title
   - Search by tags
   - Search by frontmatter fields

2. **Read Notes**
   - Get note content by name
   - Get note metadata (frontmatter)
   - Get backlinks (notes linking to this note)
   - Get outgoing links

3. **Create/Update Notes**
   - Create new note
   - Update existing note
   - Append to note
   - Update frontmatter

4. **List/Browse**
   - List all notes
   - List notes by directory
   - List tags
   - List recent notes (by modification time)

5. **Graph Operations**
   - Find notes linking to a note (backlinks)
   - Find notes linked from a note (forward links)
   - Find orphaned notes (no links)

### Data Structures

```rust
pub struct ObsidianVault {
    pub name: String,
    pub path: PathBuf,
}

pub struct ObsidianNote {
    pub path: PathBuf,
    pub title: String,
    pub content: String,
    pub frontmatter: Option<HashMap<String, serde_json::Value>>,
    pub tags: Vec<String>,
    pub links: Vec<String>,
    pub created: Option<SystemTime>,
    pub modified: Option<SystemTime>,
}

pub struct ObsidianConfig {
    pub enabled: bool,
    pub vault_path: Option<PathBuf>,
    pub vaults: Vec<ObsidianVault>,
}
```

### Tool Interface

Tools to expose to the agent:

1. `obsidian_search` - Search notes by content/title/tags
2. `obsidian_read` - Read a specific note
3. `obsidian_create` - Create a new note
4. `obsidian_update` - Update an existing note
5. `obsidian_list` - List notes (with filters)
6. `obsidian_graph` - Query the note graph (links/backlinks)

### File Format Parsing

Need to handle:
- YAML frontmatter (between `---` markers)
- Wikilinks: `[[Note Name]]`, `[[Note Name|Display Text]]`, `[[Note Name#Heading]]`
- Tags: `#tag`, `#nested/tag`
- Markdown standard features

## Security Considerations

1. **Path Traversal**: Validate all paths stay within vault directory
2. **File Access**: Only read/write markdown files (`.md`)
3. **Permissions**: Respect filesystem permissions
4. **Validation**: Sanitize note names and paths
5. **Audit Logging**: Log all vault modifications

## Testing Strategy

### Unit Tests

1. **Config Parsing**
   - Valid configurations
   - Invalid vault paths
   - Multiple vaults
   - Missing required fields

2. **Note Parsing**
   - Frontmatter extraction
   - Wikilink parsing
   - Tag extraction
   - Malformed markdown

3. **Path Validation**
   - Valid paths within vault
   - Path traversal attempts
   - Symlink handling
   - Case sensitivity

### Integration Tests

1. **Vault Operations**
   - Create test vault
   - Create/read/update notes
   - Search functionality
   - Graph operations

2. **Error Handling**
   - Missing vault directory
   - Permission denied
   - Corrupted files
   - Concurrent access

### End-to-End Tests

1. **Real Vault Scenarios**
   - Search across large vault
   - Complex wikilink resolution
   - Frontmatter variations
   - Special characters in names

## Implementation Phases

### Phase 1: Core Infrastructure (Issue #1)
- Add `obsidian` module to `src/integrations/`
- Implement `ObsidianConfig` struct and validation
- Add config loading and merging
- Add basic error types
- Write config parsing tests

**Estimated Effort**: 1-2 hours
**Risk**: Low

### Phase 2: Note Parsing (Issue #2)
- Implement markdown frontmatter parser
- Implement wikilink parser
- Implement tag extractor
- Implement `ObsidianNote` struct
- Write parsing unit tests

**Estimated Effort**: 2-3 hours
**Risk**: Medium (parsing complexity)

### Phase 3: Vault Operations (Issue #3)
- Implement vault scanning/indexing
- Implement note reading
- Implement note searching
- Implement path validation
- Write vault operation tests

**Estimated Effort**: 3-4 hours
**Risk**: Medium (filesystem edge cases)

### Phase 4: Tool Interface (Issue #4)
- Implement `obsidian_search` tool
- Implement `obsidian_read` tool
- Implement `obsidian_list` tool
- Register tools in tool registry
- Write tool integration tests

**Estimated Effort**: 2-3 hours
**Risk**: Low

### Phase 5: Write Operations (Issue #5)
- Implement note creation
- Implement note updating
- Implement frontmatter updates
- Add audit logging
- Write write operation tests

**Estimated Effort**: 2-3 hours
**Risk**: Medium (data integrity)

### Phase 6: Graph Operations (Issue #6)
- Implement backlink discovery
- Implement forward link resolution
- Implement `obsidian_graph` tool
- Write graph operation tests

**Estimated Effort**: 2-3 hours
**Risk**: Low

### Phase 7: Documentation (Issue #7)
- Write setup guide
- Document configuration options
- Add usage examples
- Update README
- Add troubleshooting guide

**Estimated Effort**: 1-2 hours
**Risk**: Low

### Phase 8: Polish & Edge Cases (Issue #8)
- Handle concurrent access
- Optimize search performance
- Add caching if needed
- Handle large vaults
- Performance testing

**Estimated Effort**: 2-3 hours
**Risk**: Medium (performance)

## GitHub Issues Checklist

Create the following issues in order:

1. **Issue: Obsidian Integration - Core Infrastructure**
   - Labels: `enhancement`, `integration:obsidian`, `size:S`
   - Milestone: Obsidian Integration
   - Description: Implement basic config and module structure
   - Depends on: None

2. **Issue: Obsidian Integration - Note Parsing**
   - Labels: `enhancement`, `integration:obsidian`, `size:M`
   - Milestone: Obsidian Integration
   - Description: Implement markdown frontmatter, wikilinks, and tag parsing
   - Depends on: Issue #1

3. **Issue: Obsidian Integration - Vault Operations**
   - Labels: `enhancement`, `integration:obsidian`, `size:M`
   - Milestone: Obsidian Integration
   - Description: Implement vault scanning, reading, and searching
   - Depends on: Issue #2

4. **Issue: Obsidian Integration - Read-Only Tools**
   - Labels: `enhancement`, `integration:obsidian`, `tool`, `size:S`
   - Milestone: Obsidian Integration
   - Description: Implement search, read, and list tools
   - Depends on: Issue #3

5. **Issue: Obsidian Integration - Write Operations**
   - Labels: `enhancement`, `integration:obsidian`, `size:M`
   - Milestone: Obsidian Integration
   - Description: Implement note creation and updates with audit logging
   - Depends on: Issue #4

6. **Issue: Obsidian Integration - Graph Operations**
   - Labels: `enhancement`, `integration:obsidian`, `size:S`
   - Milestone: Obsidian Integration
   - Description: Implement link graph queries and backlink discovery
   - Depends on: Issue #5

7. **Issue: Obsidian Integration - Documentation**
   - Labels: `documentation`, `integration:obsidian`, `size:S`
   - Milestone: Obsidian Integration
   - Description: Write setup guide, config docs, and examples
   - Depends on: Issue #6

8. **Issue: Obsidian Integration - Performance & Polish**
   - Labels: `enhancement`, `integration:obsidian`, `size:M`
   - Milestone: Obsidian Integration
   - Description: Optimize search, handle edge cases, performance testing
   - Depends on: Issue #7

## Dependencies

### External Crates

- `serde` - Already in use
- `serde_yaml` - For frontmatter parsing (likely already in use)
- `regex` - For wikilink/tag parsing (likely already in use)
- `walkdir` - For vault traversal (may already be in use)

No new heavy dependencies expected.

### System Requirements

- Filesystem access
- Read/write permissions to vault directory
- No network requirements (for Phase 1 approach)

## Success Criteria

1. ✅ Agent can search Obsidian vault by content/tags
2. ✅ Agent can read note content and metadata
3. ✅ Agent can create new notes
4. ✅ Agent can update existing notes
5. ✅ Agent can discover note relationships (links)
6. ✅ All operations are secure (path validation)
7. ✅ All operations are audited
8. ✅ Comprehensive test coverage (>80%)
9. ✅ Documentation is complete and clear
10. ✅ Works with real-world Obsidian vaults

## Future Enhancements

Post-MVP features to consider:

1. **REST API Plugin Support**
   - Add optional support for Local REST API plugin
   - Graceful fallback to filesystem access

2. **Advanced Search**
   - Dataview query support
   - Full-text search with ranking
   - Regex search

3. **Template Support**
   - Use Obsidian templates for new notes
   - Template variable substitution

4. **Daily Notes**
   - Special handling for daily notes
   - Quick daily note access

5. **Canvas Support**
   - Read Obsidian Canvas files
   - Export canvas as markdown

6. **Sync Detection**
   - Detect when vault is syncing (Obsidian Sync, iCloud, etc.)
   - Wait for sync to complete

7. **Watch Mode**
   - Monitor vault for changes
   - Trigger events on note updates

## Open Questions

1. **Multiple Vaults**: Should we support multiple vaults in initial release?
   - **Answer**: Yes, add to config schema but implement single vault first

2. **Caching**: Should we cache vault index for performance?
   - **Answer**: Not in MVP, add in Phase 8 if needed

3. **Concurrent Access**: How to handle Obsidian writing while we read?
   - **Answer**: Read operations are safe, writes need file locking

4. **Attachments**: Should we support reading/writing attachments?
   - **Answer**: Not in MVP, file operations cover this

5. **Plugins**: Should we respect `.obsidian/` config directory?
   - **Answer**: Read-only access for templates/settings if needed

## References

- [Obsidian Official Docs](https://help.obsidian.md/)
- [Obsidian API Docs](https://docs.obsidian.md/)
- [Obsidian Plugin API](https://github.com/obsidianmd/obsidian-api)
- [Local REST API Plugin](https://github.com/coddingtonbear/obsidian-local-rest-api)
- [Markdown Spec](https://commonmark.org/)
- [YAML Frontmatter Spec](https://jekyllrb.com/docs/front-matter/)

## Conclusion

The Obsidian integration will provide ZeroClaw agents with powerful knowledge management capabilities. By starting with direct filesystem access, we ensure a simple, reliable foundation that works universally. The phased approach allows for incremental delivery and testing, while the modular design enables future enhancements like REST API support.

**Total Estimated Effort**: 15-23 hours
**Risk Level**: Low-Medium
**Priority**: Medium
**Target Release**: Next minor version
