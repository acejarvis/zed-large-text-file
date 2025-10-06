# zed-large-text-file

## High-Performance Large Text File Extension for Zed

**ECE1724 Rust Course Project Proposal**

## 1. Motivation

Modern code editors, including Visual Studio Code and Zed, struggle significantly when handling large text files (>100MB), particularly simulation logs, database dumps, server logs, and other data files that can reach multiple gigabytes in size. This is a well-documented issue in the Zed editor community (see [Issue #4701](https://github.com/zed-industries/zed/issues/4701)), where users experience freezing, unresponsiveness, and crashes when attempting to open large files.

For developers and system administrators working with large log files, this limitation forces them to resort to command-line tools like `less`, `grep`, or custom scripts, which lack the convenience and features of a modern editor. The problem is particularly acute in domains such as:

- **Systems Programming**: Analyzing extensive compiler outputs, build logs, and debugging traces
- **Data Science**: Viewing large CSV files or data dumps before processing
- **DevOps/SRE**: Investigating production logs that can span gigabytes
- **Scientific Computing/Silicon Design**: Examining simulation outputs and numerical results

While Zed is built in Rust and designed for performance, it currently lacks specialized handling for extremely large files. This project aims to fill that gap by creating a Zed extension that leverages Rust's performance characteristics—zero-cost abstractions, efficient memory management, and powerful concurrency primitives—to handle multi-gigabyte text files smoothly.

The motivation for this project is threefold:
1. **Practical Need**: Address a real pain point for users who regularly work with large files
2. **Technical Challenge**: Explore advanced Rust techniques including memory mapping, lazy evaluation, multithreading, and async I/O
3. **Ecosystem Contribution**: Create an open-source extension that benefits the growing Zed code editor community

## 2. Objective and Key Features

### Primary Objective
Develop a high-performance Zed extension that enables seamless viewing, searching, and basic editing of text files at least 4GB in size, with responsive performance comparable to dedicated tools like [Large text viewer](https://apps.microsoft.com/detail/9nblggh4mcm8?hl=en-US&gl=UShttps://apps.microsoft.com/detail/9nblggh4mcm8?hl=en-US&gl=US) or custom file viewers.

### Key Features

#### 2.1 Automatic Large File Detection (Auto-enable)
- Automatically detect files larger than 4MB when opened
- Seamlessly activate the extension without user intervention
- Display a lightweight status indicator showing the extension is active
- Automatically disable language servers and syntax highlighting for large files to conserve resources

#### 2.2 Lazy Loading and Viewport-Based Rendering
- Implement memory-mapped file I/O using `memmap2` for efficient large file access
- Load and render only the visible viewport (lines currently on screen)
- Implement a buffer window around the viewport for smooth scrolling
- Support efficient jumping to arbitrary line numbers without loading the entire file
- Dynamically adjust buffer size based on scrolling behavior and available memory

#### 2.3 Multithreaded Search (Find)
- Implement parallel search across file chunks using [Rayon](https://docs.rs/rayon/latest/rayon/) or [Tokio](https://docs.rs/tokio/latest/tokio/)
- Display search progress and intermediate results as they are found
- Support both literal string search and regular expressions
- Optimize search using memory mapping and SIMD instructions where applicable
- Provide search result navigation (next/previous/all occurrence)
- Display match count and position information

#### 2.4 Multithreaded Replace
- Support find-and-replace operations with multithreading
- Implement safe in-place replacement for same-length or smaller replacements
- For size-changing replacements, use a copy-on-write strategy with temporary files
- Show preview of changes before applying
- Support undo/redo for replace operations

#### 2.5 Performance Optimizations
- Use memory-mapped I/O to avoid loading entire file into RAM
- Implement line indexing for fast random access to any line
- Utilize Rust's ownership system to minimize allocations
- Leverage SIMD operations for text scanning where possible
- Implement efficient UTF-8 validation and handling for the visible viewport only

#### 2.6 Cross-Platform Support
- Ensure compatibility with macOS, Linux, and Windows
- Handle platform-specific file system and memory mapping differences
- Test on all three major platforms

#### 2.7 User Experience Features
- Display file statistics (size, line count estimate, encoding)
- Show current position in file (line number, percentage)
- Responsive scrolling with smooth rendering
- Configurable threshold for automatic activation (default 4MB)
- Settings panel for buffer size and performance tuning

### Novelty and Gap Filling
This extension fills a clear gap in the Zed ecosystem. While other editors have various large-file handling solutions, there is currently no native or extension-based solution for Zed. The project will demonstrate how Rust's performance characteristics can be leveraged within an editor extension to handle workloads typically reserved for specialized command-line tools.

## 3. Tentative Plan

### Project Architecture

Following the [official Zed extension structure](https://zed.dev/docs/extensions/developing-extensions), the extension will be organized as follows:

```
zed-large-text-file/
├── extension.toml          # Extension manifest
├── Cargo.toml              # Rust package configuration
├── LICENSE                 # MIT license
├── README.md               # Project documentation
├── src/
│   ├── lib.rs              # Extension entry point with Extension trait impl
│   ├── file_handler.rs     # Memory-mapped file management
│   ├── viewport.rs         # Viewport management and lazy loading
│   ├── search.rs           # Multithreaded search implementation
│   ├── replace.rs          # Multithreaded replace implementation
│   ├── line_index.rs       # Line number indexing for fast access
│   └── config.rs           # Extension configuration and settings
└── tests/
    ├── integration_tests.rs
    └── test_fixtures/      # Sample patterns for test-purposed large file generation
```

### Implementation Plan

**Note**: This is a solo project. All components will be implemented, tested, and documented by a single developer over 8 weeks.

The implementation will follow an iterative approach with four main phases:

1. **Foundation & Infrastructure**: Set up the project repository with CI/CD pipeline (GitHub Actions), configure linting and testing frameworks, study the Zed extension API in depth, and implement basic extension scaffolding with proper `extension.toml` and `Cargo.toml` configuration. Create test infrastructure with generated large files (10MB to 1GB).

2. **Core File Handling**: Implement memory-mapped file I/O using `memmap2`, develop an efficient line indexing system for fast random access to any line in multi-GB files, build viewport management with lazy loading to render only visible lines, and integrate with Zed's rendering APIs. Add automatic activation for files >4MB with language server disabling.

3. **Search & Replace Features**: Implement multithreaded search using Rayon for parallel chunk processing with support for literal and regex patterns, build search result navigation interface, and add multithreaded replace with safe in-place editing for same-length replacements and copy-on-write strategy for size-changing operations. Include preview, confirmation, and undo/redo support.

4. **Polish & Cross-Platform Testing**: Test thoroughly on macOS and Linux, conduct performance optimization and profiling, add comprehensive error handling, implement user-facing features (file statistics, position indicators, settings panel), write complete documentation, and produce demonstration video with benchmark comparisons.

### Technology Stack

- **Core Language**: Rust (100%)
- **Memory-Mapped I/O**: `memmap2` crate
- **Parallelism**: `rayon` for multithreading (with potential `tokio` for async operations if needed)
- **Regular Expressions**: `regex` crate
- **Zed Extension API**: Official Zed extension SDK (`zed_extension_api`)
- **Testing**: `cargo test`, integration tests with generated large files, `criterion` for benchmarking
- **CI/CD**: GitHub Actions for automated testing, linting, and cross-platform builds

### Development Approach

As a solo developer project, the development approach will include:

- **Iterative Development**: Build features incrementally with working prototypes at each stage
- **Test-Driven Development**: Write tests alongside implementation to ensure code quality
- **Continuous Benchmarking**: Regular performance measurements to track progress toward goals
- **Documentation as Code**: Maintain up-to-date documentation throughout development
- **Version Control Best Practices**: Frequent, atomic commits with clear messages
- **Self Code Review**: Careful review of each change before committing, using linting tools

### Risk Mitigation

- **Risk**: Zed extension API may have limitations
  - **Mitigation**: Early research into API capabilities; fallback to contributing to Zed core or standalone software if extension approach is insufficient

- **Risk**: Cross-platform compatibility issues
  - **Mitigation**: Early testing on all platforms; use platform-abstraction crates where necessary

- **Risk**: Performance targets not met
  - **Mitigation**: Continuous benchmarking; iterative optimization; scope reduction if necessary

### Success Criteria

The project will be considered successful if:
1. The extension can open and scroll through a 10GB text file with <100ms response time
2. Search operations on 1GB files complete in <5 seconds on modern hardware
3. The extension works reliably on macOS, and Linux
4. Memory usage remains under 500MB regardless of file size
5. All CI/CD checks pass with >80% code coverage

---

**Repository**: https://github.com/acejarvis/zed-large-text-file  
**Project Duration**: 8 weeks  
**Estimated Lines of Code**: 1500-2000 lines (excluding tests and dependencies)
