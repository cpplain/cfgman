# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed

- **Major refactoring for better maintainability** - Simplified codebase architecture:
  - Removed logger abstraction (`logger.go`) in favor of direct output functions
  - Removed input handling abstraction (`input.go`) for non-interactive operation
  - Removed unnecessary interface abstractions (`interfaces.go`) for cleaner, more idiomatic Go code
  - Added centralized output helpers (`output.go`) for consistent formatting
  - Added path utilities (`path_utils.go`) for home directory contraction
- **Improved user messages** - Replaced log-style output with user-friendly CLI messages:
  - Error messages now show as "Error: description" without timestamps
  - Success messages use visual indicators (✓) for completed actions
  - Warning messages are clearly marked without log prefixes
  - Debug output only shown when `CFGMAN_DEBUG` environment variable is set
- **Better status output** - Status command now uses tabwriter for aligned table format with summary statistics
- **Cleaner error handling** - All errors in main.go now use consistent format without log.Fatal timestamps
- **Consistent prune-links behavior** - Updated prune-links to match remove-links: no confirmation prompt, immediate execution with "Pruning broken symlinks..." message
- **Consistent home directory display** - All paths shown to users now consistently display home directory as `~` instead of the full path
- **Non-interactive operation** - Removed all user prompts for scriptability:
  - `adopt` now requires source directory as a mandatory argument
  - `orphan` executes immediately without confirmation (use `--dry-run` for safety)
  - Removed all input handling code and interfaces
- **Cleaner orphan output** - Removed redundant initial file listing; progress is shown as files are processed
- **Simplified adopt output** - Adopt command now uses single "✓ Adopted: <path>" line per file, matching the pattern of other commands
- **Standardized all output helpers** - All commands now use consistent output helper functions (PrintError, PrintSuccess, PrintWarning, etc.) for uniform formatting across the entire CLI
- **Simplified status output** - Status command now uses the same single-line format as other commands (e.g., "✓ Active: ~/.bashrc") for consistency
- **Standardized error messages** - All error messages now follow consistent "Failed to <action>: <reason>" format with lowercase actions for better readability
- **Enhanced error context** - Error messages now include actionable suggestions where appropriate (e.g., "Use 'cfgman adopt' to adopt this file first", "Run 'cfgman init' to create a config file")
- **Added summary output for bulk operations** - Commands that operate on multiple files now show clear summaries:
  - `create-links` shows "Created X symlink(s) successfully" and failure counts
  - `remove-links` shows "Removed X symlink(s) successfully" and failure counts  
  - `prune-links` shows "Pruned X broken symlink(s) successfully" and failure counts
  - `adopt` (directories) shows "Successfully adopted X file(s)" with skip counts
  - `orphan` (directories) shows "Successfully orphaned X file(s)"
- **Improved skip messages** - Skip messages now clearly explain why files were skipped (e.g., "file already exists in repository at <path>")
- **Enhanced help output formatting** - Improved help text structure and formatting:
  - All command help uses consistent structure: Usage, Description, Arguments (if applicable), Options, Examples (if applicable)
  - Command descriptions use cyan color for better readability
  - Arguments and options are properly aligned with bold formatting
  - Added "(none)" placeholder when commands have no options
  - Main usage output organized into clear sections: Configuration, Link Management, Other

### Fixed

- **Circular symlink validation** - Fixed incorrect circular reference error when symlinks already exist and point to the correct target. The validation now properly allows existing symlinks that point to their intended source.
- **Test expectations** - Updated validation tests to reflect the new behavior where relinking (creating a symlink that already points to the correct location) is considered valid and not an error.
- **Error message consistency** - Fixed inconsistent error message formats across different commands (e.g., "could not" vs "Failed to")
- **Summary output for orphan command** - Added missing summary output when orphaning directories with multiple files, matching the pattern of other bulk operations

## [0.3.0] - 2025-06-28

### Changed - MAJOR REWRITE

This release represents a major rewrite of cfgman's core functionality.

- **BREAKING: File-only linking** - Removed directory linking feature. Cfgman now ONLY creates individual file symlinks, never directory symlinks. This ensures:
  - Consistent behavior across all operations
  - No conflicts between different source mappings
  - Ability to mix files from different sources in the same directory
  - Local-only files can coexist with managed configs
- **Configuration-driven design** - All behavior now controlled by `.cfgman.json` with no hardcoded defaults
- **Simplified codebase** - Removed obsolete features and redundant logic:
  - Removed LinkStrategy (file/directory linking modes)
  - Removed hardcoded directory list
  - Removed platform-specific home directory logic
  - Consolidated redundant orphan command logic
  - Simplified git operations to basic rm-with-fallback pattern
- **Improved create-links workflow** - Refactored to use clear three-phase approach:
  1. Discovery phase - collect all files to link
  2. Validation phase - validate all targets before making changes
  3. Execution phase - create all symlinks
- **Better error messages** - More descriptive errors throughout, especially for adopt command when source directory is not in mappings

## [0.2.0] - 2025-06-28

### Changed

- Switched from beta versioning (v1.0.0-beta.x) to standard pre-1.0 versioning (v0.x.x) to better align with semantic versioning practices for pre-release software

## [0.1.1] - 2025-06-27

### Fixed

- **Orphan command message order** - Messages now correctly reflect the actual operation order:
  1. Remove symlink
  2. Copy content back to original location
  3. Remove file from repository
- **Redundant orphan messages** - Fixed duplicate confirmation messages when orphaning directories with multiple symlinks
- **Untracked file removal** - Fixed issue where files not tracked by git were left in the repository after orphaning. The orphan command now properly removes all files from the repository regardless of git tracking status.

## [0.1.0] - 2025-06-24

Initial release of cfgman.

- **Directory-based operation** - Works from repository directory (like git, npm, make)
- **Simple configuration format** - Single `.cfgman.json` file with link mappings
- **Built-in ignore patterns** - Gitignore-style pattern matching without git dependency
- **Flexible link mappings** - Map any source directory to any target location
- **Smart linking strategies** - Choose between file-level or directory-level linking
- **Safety features**:
  - Dry-run mode for all operations
  - Confirmation prompts for destructive actions
  - Cross-repository symlink protection
- **Core commands**:
  - `init` - Create minimal configuration template
  - `status` - Show all managed symlinks with their state
  - `adopt` - Move existing files into repository management
  - `orphan` - Remove files from management (restore to original location)
  - `create-links` - Create symlinks based on configuration
  - `remove-links` - Remove all managed symlinks
  - `prune-links` - Remove broken symlinks
- **Performance** - Concurrent operations for status checking
- **Zero dependencies** - Pure Go implementation using only standard library

[unreleased]: https://github.com/cpplain/cfgman/compare/v0.3.0...HEAD
[0.3.0]: https://github.com/cpplain/cfgman/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/cpplain/cfgman/compare/v0.1.1...v0.2.0
[0.1.1]: https://github.com/cpplain/cfgman/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/cpplain/cfgman/releases/tag/v0.1.0
