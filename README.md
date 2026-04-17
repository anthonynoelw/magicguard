# MagicGuard 

MagicGuard checks whether a file really is what it claims to be. Instead of trusting file extensions, it looks at magic bytes (file signatures). This makes it useful for catching spoofed or potentially malicious files.

[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

## What it does

MagicGuard compares a file’s actual signature with its extension. This helps detect cases like an executable disguised as a PDF.

### Highlights

- Validates files using magic bytes  
- Extra checks for Office files (DOCX, XLSX, PPTX)  
- Optional SHA-256 hashing  
- Simple CLI interface  
- Modular design, easy to extend  
- Supports 25+ common file types  

## Installation

### Quick install (recommended)

**Linux / macOS**

```bash
git clone https://github.com/anthonynoelw/magicguard.git
cd magicguard

chmod +x install.sh
./install.sh

# dev setup
./install.sh --dev

# with docker checks
./install.sh --docker
```

**Windows (PowerShell)**

```powershell
git clone https://github.com/anthonynoelw/magicguard.git
cd magicguard

.\install.ps1
.\install.ps1 -Dev
.\install.ps1 -Docker
```

The script handles Python version checks, virtual environment setup, dependencies, and initializing the signature database.

### Manual setup

```bash
git clone https://github.com/anthonynoelw/magicguard.git
cd magicguard

python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

pip install -e .
pip install -e ".[dev]"
```

## Usage

### CLI

```bash
magicguard scan file.pdf
magicguard scan file.jpg --verbose
magicguard scan file.exe --hash

magicguard scan-dir ./folder
magicguard scan-dir ./folder --recursive
magicguard scan-dir ./folder -e pdf -e docx

magicguard list-signatures
magicguard status --verbose
```

### Python API

```python
from magicguard.core.validator import FileValidator

validator = FileValidator()

try:
    if validator.validate("document.pdf"):
        print("valid file")
    else:
        print("invalid file")
finally:
    validator.close()

print(validator.get_file_hash("document.pdf"))
```

## Project structure

```text
src/magicguard/
├── core/
├── cli/
└── utils/
```

The core logic is separated from the CLI, and components are loosely coupled to make testing and extension easier.

## Supported formats

**Documents**
- PDF, DOCX, XLSX, PPTX, XML  

**Images**
- PNG, JPEG, GIF, BMP, ICO, WebP  

**Archives**
- ZIP, RAR, 7Z, TAR, GZ  

**Executables**
- EXE, DLL, ELF  

**Media**
- MP3, MP4, AVI, MKV, WAV, FLAC  

**Databases**
- SQLite  

## Testing

```bash
pytest
pytest --cov=src/magicguard --cov-report=html
```

Coverage is currently around 82%, with higher coverage in core modules.

## Use cases

Detect renamed executables:

```bash
magicguard scan suspicious.pdf
```

Validate downloaded attachments before opening, or integrate checks into upload workflows.

## Docker

```bash
docker build -t magicguard:latest -f docker/Dockerfile .

docker run --rm \
  -v "$PWD/files:/scan:ro" \
  magicguard:latest scan /scan/file.pdf
```

For more advanced setups (compose, multi-arch builds, hardening), see `docker/README.md`.

## Configuration

Local data is stored in:

# Check status
docker-compose -f docker/docker-compose.yml run --rm status
```

### Multi-Architecture Support

Build for multiple platforms (amd64, arm64, arm/v7):

```bash
# Using the build script
./docker/build-multiarch.sh

# Build and push to registry
PUSH=true IMAGE_NAME=yourusername/magicguard ./docker/build-multiarch.sh
```

### Security Features

- **Multi-stage Alpine build** - Minimal 91MB image
- **Non-root user** (UID 1000) for enhanced security
- **4 security hardening levels** - From basic to maximum paranoid
- **Read-only filesystem** support with tmpfs
- **Capability dropping** - Remove all unnecessary Linux capabilities
- **Seccomp profiles** - Restrict system calls
- **Network isolation** - Optional no-network mode

For comprehensive Docker documentation, see **[docker/README.md](docker/README.md)** (400+ lines covering deployment, security, CI/CD, and troubleshooting).

## 🔧 Configuration

MagicGuard is highly configurable via environment variables, allowing flexible deployment across development, testing, and production environments.

### Configuration Files

MagicGuard supports environment-based configuration:

1. **`.env.example`** - Template with all available configuration options (safe to commit)
2. **`.env`** - Your local configuration (automatically ignored by git)

```bash
# Create your local configuration
cp .env.example .env

# Edit with your preferred values
vim .env  # or nano, code, etc.
```

### Environment Variables

All configuration variables are **optional** with sensible defaults:

#### Core Configuration

| Variable | Description | Default | Example |
|----------|-------------|---------|---------|
| `MAGICGUARD_DB_PATH` | SQLite database file location | `~/.magicguard/data/signatures.db` | `/var/lib/magicguard/signatures.db` |
| `MAGICGUARD_LOG_LEVEL` | Logging verbosity | `DEBUG` (dev), `INFO` (prod) | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` |
| `MAGICGUARD_LOG_DIR` | Directory for log files | `~/.magicguard/log` | `/var/log/magicguard` |
| `MAGICGUARD_DATA_DIR` | Application data directory | `~/.magicguard/data` | `/var/lib/magicguard/data` || `MAGICGUARD_MAX_FILE_SIZE` | Maximum file size in bytes | `104857600` (100MB) | `209715200` (200MB), `52428800` (50MB) |
#### Docker-Specific Variables

| Variable | Description | Default | Used In |
|----------|-------------|---------|---------|
| `SCAN_DIR` | Directory to mount as `/scan` | `./scan` | docker-compose.yml |
| `LOG_DIR` | Directory to mount as `/logs` | `./logs` | docker-compose.yml |
| `LOG_LEVEL` | Container log level | `INFO` | docker-compose.yml |

### Configuration Examples

#### Development Setup

```bash
# .env for local development
MAGICGUARD_LOG_LEVEL=DEBUG
MAGICGUARD_DB_PATH=/tmp/magicguard-dev/signatures.db
MAGICGUARD_LOG_DIR=/tmp/magicguard-dev/logs
MAGICGUARD_MAX_FILE_SIZE=52428800  # 50MB for testing
```

```bash
# Use development settings
magicguard scan document.pdf --verbose
```

#### Production Setup

```bash
# .env for production deployment
MAGICGUARD_LOG_LEVEL=WARNING
MAGICGUARD_DB_PATH=/var/lib/magicguard/signatures.db
MAGICGUARD_LOG_DIR=/var/log/magicguard
MAGICGUARD_DATA_DIR=/var/lib/magicguard/data
MAGICGUARD_MAX_FILE_SIZE=209715200  # 200MB for production
```

#### Testing/CI Setup

```bash
# Isolated testing environment
export MAGICGUARD_DB_PATH=/tmp/test-magicguard/signatures.db
export MAGICGUARD_LOG_DIR=/tmp/test-magicguard/logs
export MAGICGUARD_LOG_LEVEL=DEBUG

pytest tests/ --cov
```

#### Docker Configuration

```bash
# docker-compose configuration
SCAN_DIR="$PWD/samples"
LOG_DIR="$PWD/logs"
LOG_LEVEL=INFO

docker-compose -f docker/docker-compose.yml run --rm scanner scan /scan/file.pdf
```

### Application Limits

#### Configurable Limits

These can be overridden via environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `MAGICGUARD_MAX_FILE_SIZE` | 104857600 (100MB) | Maximum file size to process |

#### Fixed Limits

These constants are defined in [`src/magicguard/utils/config.py`](src/magicguard/utils/config.py) and **cannot** be overridden:

| Constant | Value | Description |
|----------|-------|-------------|
| `MAX_SIGNATURE_LENGTH` | 64 bytes | Maximum magic byte signature length |
| `MAX_LOG_FILES` | 30 files | Keep 30 days of daily log files |

### Directory Structure

#### Local Installation

MagicGuard stores data in `~/.magicguard/`:

```
~/.magicguard/
├── data/
│   └── signatures.db    # SQLite signature database
└── log/
    └── 2025-12-28.log   # Daily rotating logs (YYYY-MM-DD.log)
```

#### Docker Deployment

## Development

- **Database**: `/data/signatures.db` (use named volumes for persistence)
- **Logs**: `/logs/` (mount as volume or use tmpfs)
- **Scan files**: `/scan/` (mount read-only for security)

See [docker/README.md](docker/README.md) for comprehensive volume configuration.

### Code Usage

The configuration system is used throughout the codebase:

```python
from magicguard.utils.config import (
    get_database_path,  # Gets DB path (env or default)
    get_log_level,      # Gets log level (env or default)
    get_log_dir,        # Gets log directory (env or default)
    get_max_file_size,  # Gets max file size (env or default)
)

# Database automatically uses configured path
from magicguard.core.database import Database
db = Database()  # Uses MAGICGUARD_DB_PATH or default

# Logger respects MAGICGUARD_LOG_LEVEL
from magicguard.utils.logger import get_logger
logger = get_logger(__name__)  # Uses configured log level

# Validator respects MAGICGUARD_MAX_FILE_SIZE
from magicguard.core.validator import FileValidator
validator = FileValidator()  # Uses MAGICGUARD_MAX_FILE_SIZE or default
logger = get_logger(__name__)  # Uses configured log level
```

### Security Considerations

⚠️ **Important Security Notes:**

1. **Never commit `.env`** - Already in `.gitignore`
2. **Use restrictive permissions**: `chmod 600 .env`
3. **Use absolute paths** in production
4. **Validate user-provided paths** before using
5. **Avoid storing secrets** in environment variables when possible

Run checks:

```bash
black src/ tests/
ruff check src/ tests/
mypy src/
pre-commit run --all-files
```

## Contributing

Pull requests are welcome. Please include tests and make sure everything passes before submitting.

## License

GPL v3 — see `LICENSE`.

## Notes

MagicGuard helps detect suspicious files, but it’s not a complete security solution. Use it alongside other protections like antivirus or sandboxing.
