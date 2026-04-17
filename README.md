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

```text
~/.magicguard/
```

Docker uses `/data`, `/logs`, and `/scan`.

## Development

Code style:

- PEP 8  
- Black (100 char line length)  
- Type hints required  

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
