# tpt-imaging TODO

Pure-Rust medical image analysis and neuroimaging engine (ITK + FreeSurfer replacement).
Dual-licensed MIT OR Apache-2.0. Copyright (c) 2026 TPT Solutions.

Rules for every task: no FFI, no Apache-only / copyleft dependencies, `cargo deny check` must stay green.

---

## Phase 0: Project setup and compliance

### Repository and licensing
- [ ] `git init`, add `.gitignore` (target/, *.nii, *.dcm test outputs, wheels)
- [ ] Add `LICENSE-MIT` (Copyright (c) 2026 TPT Solutions)
- [ ] Add `LICENSE-APACHE` (Apache-2.0 text, copyright 2026 TPT Solutions)
- [ ] Add README.md (overview, goals, license section, status)
- [ ] Add CONTRIBUTING.md (contributions licensed MIT OR Apache-2.0, dependency policy)
- [ ] Add SECURITY.md and CODE_OF_CONDUCT.md
- [ ] Add CHANGELOG.md
- [ ] Set `license = "MIT OR Apache-2.0"` and `authors`/`repository` in workspace `Cargo.toml`
- [ ] Add SPDX headers policy (decide: per-file or crate-level only)

### Cargo workspace
- [ ] Create workspace `Cargo.toml` with `[workspace.package]` and `[workspace.dependencies]`
- [ ] Scaffold `tpt-imaging-core` (no_std + alloc)
- [ ] Scaffold `tpt-imaging-io`
- [ ] Scaffold `tpt-imaging-process`
- [ ] Scaffold `tpt-imaging-mesh`
- [ ] Scaffold `tpt-imaging-cli` (binary `tptimg`)
- [ ] Scaffold `bindings/tpt-imaging-py`
- [ ] Pin MSRV in `rust-toolchain.toml` / `rust-version`
- [ ] Add `rustfmt.toml` and clippy config (`-D warnings` in CI)
- [ ] Add `#![forbid(unsafe_code)]` to core, process, mesh, cli
- [ ] Add `#![deny(missing_docs)]` to public crates

### Dependency compliance (cargo-deny)
- [ ] Write `deny.toml` using current cargo-deny schema (the spec's version is invalid: duplicate `name` keys, removed `unlicensed`/`copyleft` fields)
- [ ] `[licenses] allow` = MIT, BSD-2-Clause, BSD-3-Clause, Zlib, ISC, Unicode-3.0 (Apache-2.0 NOT allowed on its own, so dual `MIT OR Apache-2.0` crates pass via MIT)
- [ ] Explicitly deny GPL-2.0/3.0, AGPL-3.0, LGPL-2.1/3.0, MPL-2.0
- [ ] `[bans] deny` entries as separate tables: `gdcm`, `itk`, `vtkio`, `openblas-src`, `*-sys` FFI crates as discovered
- [ ] Add `[advisories]` and `[sources]` (crates.io only) sections
- [ ] Verify each planned dependency's license: `glam`, `faer`, `ndarray`, `rayon`, `crossbeam`, `dicom`, `nifti`, `byteorder`, `memmap2`, `clap`, `pyo3`, `tpt-math`, `tpt-simd`
- [ ] Verify the whole transitive tree (`cargo deny list`) has no Apache-only crates
- [ ] Add `cargo-about` or `cargo-license` output to release checklist (third-party notices)
- [ ] Document the dependency policy in `docs/DEPENDENCIES.md`

### CI and tooling
- [ ] GitHub Actions: build + test matrix on Linux, macOS, Windows (x86_64 and aarch64)
- [ ] CI job: stable + MSRV
- [ ] CI job: `cargo fmt --check` and `cargo clippy`
- [ ] CI job: `cargo deny check`
- [ ] CI job: `cargo doc` with warnings denied
- [ ] CI job: `tpt-imaging-core` builds for a bare-metal `no_std` target (e.g. `thumbv7em-none-eabihf`)
- [ ] CI job: `tpt-imaging-core` and `tpt-imaging-io` build for `wasm32-unknown-unknown`
- [ ] CI job: `cargo-audit` / advisories
- [ ] CI job: test coverage reporting (llvm-cov)
- [ ] CI job: Miri on `core` (and any unsafe-free crate that supports it)
- [ ] Verify `tpt-math` and `tpt-simd` on crates.io: record versions, MSRV, `no_std` support, features needed (PDE solver, B-splines, optimizers, SIMD kernels)
- [ ] Add upstream issues/notes for any gaps found in `tpt-math` / `tpt-simd`

### Architecture docs
- [ ] Convert spec ADR-001/002/003 into `docs/adr/` files
- [ ] Decide and document the N-D voxel grid design (const generics vs runtime dims, memory layout, strides)
- [ ] Decide and document error handling (`thiserror` license check, or hand-written errors for `no_std`)
- [ ] Decide and document the feature-flag scheme (`std`, `alloc`, `rayon`, `mmap`, `simd`)
- [ ] Decide and document the coordinate convention (RAS/LPS, voxel center vs corner)

### Test data and validation policy
- [ ] Create synthetic phantom generators (spheres, cubes, Shepp-Logan, tori for topology tests)
- [ ] Audit licenses of candidate public sample DICOM/NIfTI data; keep only CC0/public-domain/permissive items
- [ ] Audit MNI152 license terms; decide whether it may be bundled, downloaded on demand, or user-supplied
- [ ] Audit Desikan-Killiany atlas / label data license; keep out of repo unless clearly permitted
- [ ] Define how ITK/FreeSurfer/ANTs reference outputs are compared (external, not redistributed)
- [ ] Set up a test-data download script with checksums (nothing large committed to git)

---

## Phase 1: Core and I/O (spec months 1-3)

### tpt-imaging-core (no_std + alloc)
- [ ] N-dimensional voxel grid type (shape, strides, spacing, origin, direction)
- [ ] Generic pixel/voxel element trait (u8, i16, u16, i32, f32, f64, RGB)
- [ ] Owned image container and borrowed view / mutable view types
- [ ] Slicing, sub-volume views, iterators (linear, N-D index)
- [ ] Physical <-> voxel affine transforms (via `glam` and `tpt-math`)
- [ ] Orientation handling (RAS/LPS/LAS codes, reorientation, direction cosines)
- [ ] Resampling grid definition and interpolators (nearest, linear, cubic)
- [ ] Spatial indexing (bounding boxes, octree or grid index)
- [ ] Border/padding policies (zero, constant, mirror, clamp)
- [ ] Error types for `no_std`
- [ ] Unit tests and property tests (proptest license check) for transforms and indexing
- [ ] Confirm `cargo build --no-default-features --features alloc` works
- [ ] Rustdoc examples for every public type

### tpt-imaging-io: NIfTI
- [ ] NIfTI-1 reader (`.nii`, `.nii.gz`) mapped into core types
- [ ] NIfTI-2 reader
- [ ] NIfTI-1/2 writer
- [ ] Header fields: sform/qform, scaling slope/intercept, datatype, units, extensions
- [ ] Gzip support via a pure-Rust, permissively licensed backend (verify `flate2` backend choice, e.g. `miniz_oxide`)
- [ ] Endianness handling (`byteorder`)
- [ ] Round-trip tests against synthetic data
- [ ] Optional memory-mapped zero-copy reader behind feature `mmap` (isolated module, `unsafe` audited and documented, `memmap2` license verified)

### tpt-imaging-io: DICOM
- [ ] DICOM file parsing via `dicom` crate ecosystem (confirm license of every sub-crate used)
- [ ] Transfer syntaxes: implicit VR LE, explicit VR LE/BE
- [ ] Encapsulated pixel data: RLE Lossless
- [ ] Encapsulated pixel data: JPEG baseline/extended and JPEG Lossless (pure Rust decoder, license verified)
- [ ] Evaluate pure-Rust JPEG2000 decoders against license and zero-FFI policy (decision recorded in ADR; if none qualify, defer to Backlog)
- [ ] Series discovery and grouping (SeriesInstanceUID, ImageOrientationPatient)
- [ ] Slice sorting by position, slice spacing computation, gantry tilt / non-uniform spacing handling
- [ ] Rescale slope/intercept, photometric interpretation, bits stored/allocated
- [ ] Multi-frame and enhanced MR/CT objects
- [ ] DICOM-to-NIfTI geometry conversion (orientation, affine) verified
- [ ] PHI handling: option to strip/anonymize tags; never log patient data
- [ ] Malformed-file robustness tests and fuzz targets (`cargo-fuzz`)

### tpt-imaging-io: other formats
- [ ] NRRD reader/writer (raw, gzip; detached headers)
- [ ] Analyze 7.5 reader/writer (`.hdr`/`.img`)
- [ ] Format auto-detection by magic bytes / extension
- [ ] Unified `read_image` / `write_image` API

### Streaming
- [ ] Chunked/slab reader for large volumes (bounded memory)
- [ ] Streaming writer
- [ ] Test with a multi-GB synthetic whole-body volume with a capped memory budget

### Phase 1 CLI (minimal)
- [ ] Minimal `tptimg convert` (DICOM series -> NIfTI) wired up
- [ ] `tptimg info` header dump

**Milestone 1:** `tptimg convert` reads a DICOM series and exports a correct NIfTI file; geometry validated against reference tools on permissive data.

---

## Phase 2: Processing and registration (spec months 4-6)

### Filtering
- [ ] Generic N-D separable convolution
- [ ] Gaussian smoothing and derivatives
- [ ] Mean, median, and bilateral filters
- [ ] Gradient magnitude, Laplacian
- [ ] Anisotropic diffusion (Perona-Malik, curvature flow)
- [ ] Morphology: erosion, dilation, opening, closing (binary and grayscale, N-D structuring elements)
- [ ] Connected components and hole filling
- [ ] Distance transforms
- [ ] Resampling, cropping, padding, reorientation
- [ ] Intensity ops: rescale, histogram equalization, thresholding (Otsu)
- [ ] Parallelize with `rayon` (chunked, no data races)
- [ ] Integrate `tpt-simd` kernels for hot loops
- [ ] Benchmarks (criterion) and correctness tests vs reference outputs

### Registration: rigid and affine
- [ ] Transform types: translation, rigid (6 DOF), similarity, affine (12 DOF)
- [ ] Interpolators for resampling during optimization
- [ ] Metrics: mean squared error, normalized cross-correlation
- [ ] Metrics: mutual information (Mattes) and normalized mutual information
- [ ] Optimizers via `tpt-math` (gradient descent, L-BFGS, Powell or similar)
- [ ] Multi-resolution pyramid with smoothing
- [ ] Center-of-mass / moments initialization
- [ ] Sampling strategies (random/regular subsample) for speed
- [ ] Parallel metric evaluation
- [ ] Save/load transforms (own format, ITK `.tfm` / `.mat` interop if license-safe)
- [ ] Accuracy tests with synthetic known transforms
- [ ] Performance target: align two brain scans with mutual information in seconds

### Segmentation (basic)
- [ ] Thresholding segmentation
- [ ] Region growing (connected threshold, confidence connected)
- [ ] Watershed (optional, decide scope)

**Milestone 2:** `tptimg register` aligns two brain scans using mutual information in seconds; benchmarked.

---

## Phase 3: Mesh and surface reconstruction (spec months 7-9)

### Mesh data structures
- [ ] Half-edge mesh data structure
- [ ] Triangle mesh type with vertex attributes (normals, scalars)
- [ ] Validated types: `ManifoldMesh` / `ClosedMesh` / `Genus0Mesh` as type-state (ADR-003)
- [ ] Manifold validation (edge-manifold, vertex-manifold, closed, orientable)
- [ ] Euler characteristic and genus computation
- [ ] Mesh I/O: STL, OBJ, PLY, FreeSurfer surface format, GIFTI (license-safe, no `vtkio`)
- [ ] Spatial acceleration (BVH) for intersection tests

### Isosurface extraction
- [ ] Marching Cubes (baseline)
- [ ] Topology-preserving variant (Custodio et al. / asymptotic-decider based; cite and study the paper)
- [ ] Parallelized extraction with `rayon`
- [ ] Vertex welding and consistent orientation
- [ ] Sub-voxel placement and voxel-to-world transform
- [ ] Tests: sphere, torus, nested shells, ambiguous-case cubes

### Topology repair
- [ ] Detect non-manifold edges and vertices
- [ ] Detect and fill holes
- [ ] Detect and remove handles / tunnels (voxel-domain topology correction)
- [ ] Self-intersection detection and resolution
- [ ] Guarantee genus-0 output or return a typed error
- [ ] Property/fuzz tests on random volumes

### Mesh processing
- [ ] Laplacian / Taubin smoothing
- [ ] Decimation (parallel)
- [ ] Remeshing and subdivision (optional)

### Spherical mapping
- [ ] Discrete Laplace-Beltrami operator (cotangent weights)
- [ ] Conformal mapping of genus-0 surface to the sphere
- [ ] Area/angle distortion metrics
- [ ] Inflated surface generation
- [ ] Sphere registration groundwork (for parcellation)

### Surface analysis (first pass)
- [ ] Vertex normals, mean/Gaussian curvature
- [ ] Surface area and volume

**Milestone 3:** `tptimg mesh extract` produces a genus-0 surface from a segmented brain volume with no topological defects.

---

## Phase 4: CLI, Python bindings and v0.1.0 release (spec months 10-12)

### tpt-imaging-cli (`tptimg`)
- [ ] `clap` command structure with subcommands
- [ ] `tptimg convert`
- [ ] `tptimg info`
- [ ] `tptimg register`
- [ ] `tptimg resample`
- [ ] `tptimg filter`
- [ ] `tptimg segment`
- [ ] `tptimg mesh extract`
- [ ] Progress bars, logging verbosity, consistent exit codes
- [ ] Shell completions and man pages
- [ ] CLI integration tests (`assert_cmd` license check)
- [ ] Prebuilt binaries for Linux, macOS, Windows (x86_64 + aarch64)

### tpt-imaging-py (PyO3)
- [ ] Set up `pyo3` + `maturin` (confirm `pyo3` licensing accepted under policy: MIT OR Apache-2.0)
- [ ] `abi3` wheels for supported Python versions
- [ ] NumPy interop (zero-copy where possible; `numpy` crate license check)
- [ ] Bindings: read/write, filters, registration, mesh extraction
- [ ] Type stubs (`.pyi`) and Python docs
- [ ] Python tests (pytest)
- [ ] Wheels for Linux, macOS, Windows (x86_64 + aarch64) via CI
- [ ] Optional SimpleITK-compatible convenience layer (decide scope)
- [ ] `unsafe` audit for the binding crate

### Documentation and examples
- [ ] mdBook or rustdoc site (user guide, architecture, ADRs)
- [ ] Examples for each crate
- [ ] Tutorial: DICOM -> NIfTI -> register -> mesh
- [ ] Migration guide from ITK/SimpleITK

### Benchmarks
- [ ] Criterion benchmark suite per crate
- [ ] Reproducible comparison scripts vs ITK and FreeSurfer (external tools, not bundled)
- [ ] Publish benchmark methodology and results

### Release v0.1.0
- [ ] Final dependency/license audit (`cargo deny`, `cargo about`), third-party notices
- [ ] Reserve names on crates.io and PyPI early (before release)
- [ ] Publish crates in dependency order: core, io, process, mesh, cli
- [ ] Publish Python package to PyPI
- [ ] Tag release, GitHub release notes, CHANGELOG
- [ ] Announce

**Milestone 4:** v0.1.0 on crates.io and PyPI with published benchmarks.

---

## Phase 5: Full cortical reconstruction pipeline

### Preprocessing
- [ ] N4 bias field correction (B-spline fitting via `tpt-math`)
- [ ] Intensity normalization (WM peak / histogram matching)
- [ ] Denoising (anisotropic diffusion, non-local means)

### Non-linear registration
- [ ] Demons / diffeomorphic Demons (ADR-002)
- [ ] LDDMM using `tpt-math` PDE solvers and `tpt-simd`
- [ ] Atlas registration to MNI152 (atlas supplied per license audit)
- [ ] Deformation field I/O and composition
- [ ] Validation on synthetic deformations

### Brain extraction and segmentation
- [ ] Atlas-based skull stripping (registration + morphology)
- [ ] Tissue segmentation: GM / WM / CSF with partial volume estimation
- [ ] Level-set segmentation
- [ ] Atlas-based tissue classification (priors)
- [ ] Subcortical structure segmentation (decide scope)

### Surface pipeline
- [ ] White matter surface extraction from segmentation (topology-preserving)
- [ ] Pial surface generation (surface deformation / expansion)
- [ ] Cortical thickness (white-to-pial distance)
- [ ] Curvature, sulcal depth, gyrification metrics
- [ ] Spherical inflation (conformal) and sphere registration to a template
- [ ] Parcellation (Desikan-Killiany or equivalent; atlas data per license audit)
- [ ] Surface-based statistics (per-parcel thickness, area, volume)

### Pipeline and integration
- [ ] `tptimg recon` end-to-end command (DICOM -> surfaces + stats)
- [ ] Python export of meshes and thickness maps
- [ ] Validation against FreeSurfer outputs (external reference, non-redistributed)
- [ ] Performance and memory profiling on full-resolution T1 scans
- [ ] Clinical-use disclaimer / regulatory notes in docs (research use only unless validated)

**Milestone 5:** Full cortical surface reconstruction from a T1 MRI, with parcellation and thickness maps, validated against FreeSurfer.

---

## Backlog / later
- [ ] JPEG2000 transfer syntaxes (if no pure-Rust permissive decoder was found in Phase 1)
- [ ] JPEG-LS and other DICOM compression syntaxes
- [ ] DICOM write support / DICOM-SEG / RT-STRUCT
- [ ] Additional formats (MGH/MGZ, MINC, ITK MetaImage)
- [ ] GPU acceleration (`wgpu` license check)
- [ ] Diffusion MRI (DTI) tools
- [ ] Functional MRI preprocessing
- [ ] Volume rendering / viewer integration
- [ ] Embedded-device example (no_std core on a real target)
- [ ] Packaging: Homebrew, winget, conda-forge
