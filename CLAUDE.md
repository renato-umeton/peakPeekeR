# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

peakPeekeR is an R/Shiny bioinformatics package for peak calling parameter optimization on ChIP-seq, ATAC-seq, CUTNRUN, or other signal pile-up genomic data. It provides an interactive web interface for testing multiple peak callers with varying parameters on genomic region subsets.

## Development Commands

### Package Installation
```R
devtools::install_github("stjude-biohackathon/peakPeakeR")
```

### Testing
```R
library("peakPeekeR")
test()  # Tests all conda environments, takes several minutes on first run
```

### Development
```R
devtools::load_all()  # Load package during development
devtools::document()  # Update documentation from roxygen2 comments
devtools::check()     # R CMD check
```

### Running the App
```R
peakPeekeR(trt_bam = "path/to/treatment.bam",
           ctrl_bam = "path/to/control.bam")  # optional control
```

## Architecture

### Shiny Module Pattern
Each peak caller is implemented as a Shiny module with:
- `{caller}UI(id)` - reactive UI function
- `{caller}Server(id, ...)` - server logic with `moduleServer()`
- Private `.{caller}_calling()` - actual peak calling implementation

### Peak Callers Supported
1. **MACS2** (Python 3.8) - narrow and broad peaks
2. **MACS** (Python 2.7) - legacy MACS
3. **SICER2** (Python 3.8) - broad domains, requires bedtools
4. **Genrich** (Python 3.8) - replicated samples support

### Conda Environment Management
- Uses Basilisk package for isolated Python environments
- Environments defined in `R/basilisk.R`
- Auto-created on first package load via `configure` script
- Peak calling functions use `basiliskRun()` for execution

### Key Data Flow
1. BAM subsetting by genomic coordinates (`R/file_utils.R`)
2. Format conversion (BAM to BED for SICER2, query-name sorting for Genrich)
3. Peak calling via basilisk in temporary directories
4. Visualization with ggbio tracks (treatment, control, peaks)

## File Structure

```
R/
├── peakPeekeR.R         # Main Shiny app (246 lines)
├── basilisk.R           # Python environment definitions
├── *_module.R           # Peak caller modules (macs2, macs, sicer2, genrich)
├── file_utils.R         # BAM processing utilities
├── module_utils.R       # Shiny module helpers
├── test.R               # Environment testing function
└── zzz.R                # Package load hooks
```

## Development Conventions

### Code Organization
- **Private functions**: Prefixed with `.` (e.g., `.subset_bams()`)
- **Module IDs**: Generated with `UUIDgenerate()` for dynamic insertion
- **Outputs**: All peak calling results written to `tempdir()`
- **Documentation**: Roxygen2 for all exported functions

### R Package Standards
- **NAMESPACE**: Auto-generated, only exports `peakPeekeR()` and `test()`
- **Imports**: Explicitly declared in DESCRIPTION, use `@importFrom`
- **Assets**: Package images registered in `.onLoad()` hook

### Adding New Peak Callers
1. Create `R/{caller}_module.R` with UI/Server functions
2. Implement private `.{caller}_calling()` function
3. Add basilisk environment in `R/basilisk.R` if needed
4. Update caller selection in main app (`peakPeekeR.R`)
5. Update NAMESPACE and DESCRIPTION imports

## Important Technical Details

- **Genome Reference**: Hardcoded to hg38 (BSgenome.Hsapiens.UCSC.hg38)
- **BAM Requirements**: Indexed BAM files, supports optional control samples
- **Python Dependencies**: Managed via conda, isolated from system Python
- **Temporary Files**: All outputs ephemeral, not persisted between sessions
- **UI Dynamics**: Peak caller modules inserted/removed with `insertUI()`/`removeUI()`

## Testing Requirements

The `test()` function verifies:
- All basilisk environments are functional
- Peak callers can be executed (`--version` commands)
- Returns version strings for verification

Testing requires no BAM files - only tests environment setup.