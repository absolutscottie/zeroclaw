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

Based on research, there are four viable integration approaches:

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

### Approach 4: Obsidian CLI (IPC-based) — NEW (February 2026)

**Description**: Use the official Obsidian CLI released in February 2026, which communicates with a running Obsidian instance via IPC (Inter-Process Communication).

**Release Timeline**:
- Early Access: v1.12.0 (February 10, 2026)
- General Availability: v1.12.4 (February 27, 2026)

**Pros**:
- Official, first-party tool backed by Obsidian team
- Purpose-built for automation and agentic tools
- Access to plugin features and JavaScript execution
- Rich command set (read, create, search, tasks, etc.)
- JSON output format support
- Built-in TUI with autocomplete
- No Catalyst license required (GA)
- Handles content rendering/processing (not raw files)
- Strong automation/CI-CD focus

**Cons**:
- Requires Obsidian application to be running
- IPC limits to local machine only (no remote access)
- Cannot operate headless (no server deployment)
- Performance overhead vs. direct filesystem
- Breaks if Obsidian crashes or is closed
- Difficult to run multiple concurrent instances
- Not suitable for CI/CD without GUI environment

**Implementation Complexity**: Low-Medium

**Best Use Cases**:
- Local automation workflows
- Development and debugging
- Agentic tool integration (when Obsidian is running)
- Interactive operations
- Plugin development/testing

## Recommended Approach

**Start with Approach 1 (Direct Filesystem Access) as the primary implementation, with optional future support for Approach 4 (Obsidian CLI).**

### Rationale

1. **Simplicity**: Filesystem access is simpler and more reliable
2. **Independence**: No dependency on Obsidian being running or plugins
3. **Universality**: Works with any Obsidian vault out of the box
4. **Performance**: Direct file access is faster than IPC overhead
5. **Security**: No network exposure, no API keys to manage
6. **Headless Capability**: Works in server/CI environments without GUI
7. **Incremental**: Can add REST API or CLI support later if needed

### Why Not Approach 4 (CLI) as Primary?

While the official Obsidian CLI is excellent for local automation and development workflows, it has limitations for ZeroClaw:

1. **Running App Requirement**: ZeroClaw agents may need to operate in headless environments (servers, CI/CD) where Obsidian cannot run
2. **Scalability**: IPC-based approach limits concurrent operations and remote access
3. **Deployment Flexibility**: Filesystem approach works everywhere; CLI requires Obsidian installation
4. **Reliability**: Filesystem operations are independent; CLI depends on app stability

### Future Enhancement Path

Once Approach 1 is stable, consider adding Approach 4 support for:
- Enhanced user workflows (when Obsidian is running locally)
- Plugin integration capabilities
- Interactive development scenarios
- Graceful fallback to filesystem if CLI unavailable

## Comparison Matrix: Access Methods

| Feature | **Filesystem** | **CLI (IPC)** | **REST API** | **Hybrid** |
|---------|----------------|--------------|--------------|-----------|
| **Official** | N/A | ✅ Yes | ❌ Third-party | Mixed |
| **Requires Running App** | ❌ No | ✅ Yes | ✅ Yes | Conditional |
| **Headless Capable** | ✅ Yes | ❌ No | ❌ No | ✅ Yes |
| **Plugin Access** | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes |
| **Setup Complexity** | None | Low | Medium | High |
| **Performance** | ⚡ Fast | ⚠️ Moderate | ⚠️ Moderate | Variable |
| **Remote Access** | ❌ No | ❌ No | ✅ Yes | ⚠️ Limited |
| **CI/CD Friendly** | ✅ Yes | ❌ No | ❌ No | ✅ Yes |
| **Reliability** | ✅ High | ⚠️ App-dependent | ⚠️ Plugin-dependent | ✅ High |
| **Implementation Effort** | Low | Low-Medium | Medium | High |

## Technical Comparison: CLI vs. Filesystem

### Obsidian CLI Capabilities (New in v1.12+)

**Supported Commands**:
- `read` - Read note content
- `create` - Create new notes
- `search` - Full-text search with JSON output
- `daily` - Access daily notes
- `tasks` - List tasks from notes
- `diff` - Compare versions
- `tags` - View tags with frequency
- `files` - List files with sorting/filtering
- `unresolved` - Find broken links
- `eval` - Execute JavaScript against the app

**Key Advantages**:
- Official API surface maintained by Obsidian team
- Access to plugin ecosystem and rendered content
- JavaScript execution for complex operations
- TUI with autocomplete for interactive use
- Purpose-designed for automation workflows

**Key Limitations**:
- IPC-only (no remote access)
- Requires running Obsidian instance
- Cannot run in headless/server environments
- Performance overhead vs. direct file access

### Filesystem Access Capabilities

**Core Operations**:
- Direct markdown file reading
- Wikilink and tag parsing
- Frontmatter extraction
- Note searching and indexing
- Path validation and traversal

**Key Advantages**:
- Works in any environment (servers, CI/CD, headless)
- No dependencies on Obsidian being installed/running
- Simple, reliable, fast
- Complete control over parsing logic

**Key Limitations**:
- Must implement own parsing (frontmatter, wikilinks, tags)
- No access to plugin features
- Raw markdown, not rendered content
- No JavaScript execution

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

1. **Obsidian CLI Support (v1.12+)**
   - Add optional support for official Obsidian CLI
   - Use when Obsidian is running locally
   - Graceful fallback to filesystem access if CLI unavailable
   - Enable plugin integration and JavaScript execution
   - Support for development/debugging workflows

2. **REST API Plugin Support**
   - Add optional support for Local REST API plugin
   - Graceful fallback to filesystem access
   - Enable network-based access

3. **Advanced Search**
   - Dataview query support
   - Full-text search with ranking
   - Regex search

4. **Template Support**
   - Use Obsidian templates for new notes
   - Template variable substitution

5. **Daily Notes**
   - Special handling for daily notes
   - Quick daily note access

6. **Canvas Support**
   - Read Obsidian Canvas files
   - Export canvas as markdown

7. **Sync Detection**
   - Detect when vault is syncing (Obsidian Sync, iCloud, etc.)
   - Wait for sync to complete

8. **Watch Mode**
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
- [Obsidian CLI Documentation](https://obsidian.md/cli) — Official CLI (v1.12+, released February 2026)
- [Obsidian CLI Help](https://obsidian.md/help/cli) — CLI command reference
- [Obsidian API Docs](https://docs.obsidian.md/)
- [Obsidian Plugin API](https://github.com/obsidianmd/obsidian-api)
- [Local REST API Plugin](https://github.com/coddingtonbear/obsidian-local-rest-api)
- [Markdown Spec](https://commonmark.org/)
- [YAML Frontmatter Spec](https://jekyllrb.com/docs/front-matter/)

## Conclusion

The Obsidian integration will provide ZeroClaw agents with powerful knowledge management capabilities. By starting with direct filesystem access, we ensure a simple, reliable foundation that works universally, including in headless and CI/CD environments. The phased approach allows for incremental delivery and testing, while the modular design enables future enhancements like REST API support or the official Obsidian CLI (released February 2026).

The new Obsidian CLI offers exciting opportunities for local automation workflows and plugin integration, and will be evaluated as a complementary approach in future phases. This hybrid strategy ensures ZeroClaw can serve diverse deployment scenarios—from headless servers to interactive development environments.

**Total Estimated Effort**: 15-23 hours (Phase 1-8)
**Risk Level**: Low-Medium
**Priority**: Medium
**Target Release**: Next minor version
**Future CLI Enhancement**: Post-MVP phase
