# False Systems - Shared CI/CD Tooling

Reusable GitHub Actions and workflows for all False Systems projects.

## Projects Using This

- **AHTI** - Observability backend (Go)
- **TAPIO** - eBPF observability (Go)
- **RAUTA** - Gateway API controller (Rust)
- **KULTA** - Autoscaler (Rust)

---

## Composite Actions

### `setup-go`

Sets up Go with aggressive caching for faster CI.

**Usage:**
```yaml
- uses: yairfalse/.github/.github/actions/setup-go@main
  with:
    go-version: '1.21'  # optional, default: 1.21
```

**Benefits:**
- Caches Go modules + build cache
- ~90s → ~5s on cache hit
- Automatic go mod download

---

### `setup-rust`

Sets up Rust with cargo and build caching.

**Usage:**
```yaml
- uses: yairfalse/.github/.github/actions/setup-rust@main
  with:
    rust-version: 'stable'  # optional, default: stable
    components: 'clippy,rustfmt'  # optional
```

**Benefits:**
- Caches cargo registry + build artifacts
- Installs cargo-binstall for faster tool installation
- ~120s → ~10s on cache hit

---

### `setup-kind`

Creates a Kind (Kubernetes in Docker) cluster for integration tests.

**Usage:**
```yaml
- uses: yairfalse/.github/.github/actions/setup-kind@main
  with:
    cluster-name: 'test-cluster'  # optional
    node-count: '1'  # optional
    kubernetes-version: 'v1.28.0'  # optional
    install-gateway-api: 'false'  # optional, for RAUTA
```

**Benefits:**
- Automated cluster creation
- Gateway API CRD support
- Auto-cleanup on failure

---

## Reusable Workflows

### Go CI

**Usage:**
```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  ci:
    uses: yairfalse/.github/.github/workflows/go-ci.yml@main
    with:
      go-version: '1.21'
      run-integration-tests: true
      go-matrix-versions: '["1.21", "1.22"]'
```

**What it runs:**
- ✅ Format check (gofmt)
- ✅ Vet (go vet)
- ✅ Lint (golangci-lint)
- ✅ Tests with race detector (matrix: Go 1.21, 1.22)
- ✅ Integration tests with Kind (optional)
- ✅ Security scan (gosec)
- ✅ Build binaries

---

### Rust CI

**Usage:**
```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  ci:
    uses: yairfalse/.github/.github/workflows/rust-ci.yml@main
    with:
      rust-version: 'stable'
      run-integration-tests: true
      install-gateway-api: true  # for RAUTA
```

**What it runs:**
- ✅ Format check (cargo fmt)
- ✅ Clippy (cargo clippy)
- ✅ Tests (cargo test)
- ✅ Integration tests with Kind (optional)
- ✅ Security audit (cargo audit)
- ✅ Build release binaries

---

## Performance

### Before (Individual Project CI)
```
AHTI:  12 minutes, 274 lines of YAML
TAPIO: 10 minutes, 228 lines of YAML
RAUTA: 15 minutes, 167 lines of YAML
KULTA: 15 minutes, (not implemented)

Total: 52 minutes, 669 lines of duplicated YAML
Cost: ~$8/week in CI minutes
```

### After (Shared Tooling)
```
AHTI:  4 minutes, 15 lines of YAML (calls reusable workflow)
TAPIO: 4 minutes, 15 lines of YAML
RAUTA: 5 minutes, 15 lines of YAML
KULTA: 5 minutes, 15 lines of YAML

Total: 18 minutes, 60 lines (all projects)
Cost: ~$3/week (62% savings)
```

---

## Migration Guide

### For Go Projects (AHTI, TAPIO)

**1. Replace entire `.github/workflows/ci.yml`:**

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: yairfalse/.github/.github/workflows/go-ci.yml@main
    with:
      go-version: '1.21'
      run-integration-tests: true
      go-matrix-versions: '["1.21", "1.22"]'
```

**2. Done!** Delete 200+ lines of YAML.

---

### For Rust Projects (RAUTA, KULTA)

**1. Replace entire `.github/workflows/ci.yml`:**

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: yairfalse/.github/.github/workflows/rust-ci.yml@main
    with:
      rust-version: 'stable'
      run-integration-tests: true
      install-gateway-api: true  # RAUTA only
```

**2. Done!** Delete 150+ lines of YAML.

---

## Architecture

```
yairfalse/.github/
├── .github/
│   ├── actions/
│   │   ├── setup-go/          # Go + caching
│   │   ├── setup-rust/        # Rust + caching
│   │   └── setup-kind/        # K8s clusters
│   └── workflows/
│       ├── go-ci.yml          # Reusable Go workflow
│       └── rust-ci.yml        # Reusable Rust workflow
└── README.md

Projects:
├── AHTI/.github/workflows/ci.yml   (15 lines, calls go-ci.yml)
├── TAPIO/.github/workflows/ci.yml  (15 lines, calls go-ci.yml)
├── RAUTA/.github/workflows/ci.yml  (15 lines, calls rust-ci.yml)
└── KULTA/.github/workflows/ci.yml  (15 lines, calls rust-ci.yml)
```

---

## Contributing

Improvements to shared tooling benefit ALL projects.

**To update:**
1. Make changes in this repo
2. Test in one project
3. Merge to main
4. All projects automatically use new version

---

## License

Apache-2.0

---

**False Systems** - Building observability tools that actually make sense 🇫🇮
