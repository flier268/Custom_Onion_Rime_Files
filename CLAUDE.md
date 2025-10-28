# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the "Onion Rime Files" (洋蔥 Rime 方案) repository, a comprehensive collection of Rime IME (Input Method Engine) schemas for Traditional Chinese and multilingual input on Linux/macOS/Windows. The project focuses on Zhuyin (Bopomofo/注音), Pinyin, Array, and shape-based input methods with extensive foreign language support (Japanese, Korean, Latin, Greek, Cyrillic).

**Key Characteristics:**
- Multi-platform desktop Rime configurations (not mobile)
- Schema files (.schema.yaml) define input method behavior
- Dictionary files (.dict.yaml) contain character/word mappings
- Lua scripts (rimefiles/lua/) provide advanced features and filters
- OpenCC configurations handle Traditional/Simplified Chinese conversion
- Extensive multilingual support integrated into primary schemas

## Repository Structure

### Core Directories

- **`rimefiles/`**: Central repository of all Rime configuration files (not organized by schema)
  - All schema files, dictionaries, and Lua scripts live here
  - Files are shared across multiple input method variants
  - **`rimefiles/lua/`**: Lua processors, filters, and translators
    - `processor_*.lua`: Input processing (key handling, auto-switching)
    - `filter_*.lua`: Candidate filtering and display customization
    - `translator_*.lua`: Custom translation logic
    - `f_components/`: Utility functions (date/time, calculations, encoding)
  - **`rimefiles/opencc/`**: OpenCC conversion tables for Traditional/Simplified Chinese

- **`電腦RIME方案_YYYYMMDD/`**: Generated distribution folders (created by sort scripts)
  - Each subfolder is a complete, ready-to-deploy input method package
  - Examples: `注音洋蔥plus版_YYYYMMDD/`, `注音洋蔥mixin版_YYYYMMDD/`, `洋蔥行列30_YYYYMMDD/`

- **Root Python scripts**: Distribution preparation tools
  - `sort_rime_file.py`: Main script to organize files into deployable schema packages
  - `sort_rime_file_by_bpin1.py`, `sort_rime_file_by_allin1.py`: Alternative organization scripts

### File Types and Naming Conventions

**Schema Files (`.schema.yaml`)**:
- Define input method configuration: keys, engine components, switches
- Example: `bopomo_onionplus.schema.yaml` (Zhuyin plus variant)
- Custom overrides: `.custom.yaml` files (per-platform settings in subdirectories)

**Dictionary Files (`.dict.yaml`, `.extended.dict.yaml`)**:
- `.dict.yaml`: Character/word lists with pronunciation codes
- `.extended.dict.yaml`: "Main dictionary" that imports other dictionaries
- Naming patterns:
  - `bopomo_onion*.dict.yaml`: Zhuyin-related
  - `*_jp*.dict.yaml`: Japanese
  - `*_kr*.dict.yaml`: Korean
  - `*_la*.dict.yaml`: Latin/phonetic
  - `phrases.*.dict.yaml`: Multi-word phrases

**Lua Scripts (`.lua`)**:
- `rime.lua`: Entry point that imports all other Lua modules
- Component-based architecture: processors → translators → filters

**Other Files**:
- `essay-*.txt`: Language model/frequency data (八股文)
- `punct_*.yaml`: Punctuation definitions
- `element_*.yaml`: Reusable configuration fragments
- `*.txt` (in schema folders): User phrase files (custom shortcuts)

## Primary Input Methods (Schemas)

1. **Zhuyin/Bopomofo (注音) Series**:
   - `bopomo_onion.schema.yaml`: Pure Zhuyin (純注音版) - minimal features
   - `bopomo_onionplus.schema.yaml`: Plus variant (plus版) - adds foreign language support
   - `bo_mixin1-4.schema.yaml`: Mix-in variants (mixin版) - multilingual simultaneous input (4 keyboard layouts)
   - `bopomo_onion_double.schema.yaml`: Shuangpin Zhuyin (雙拼版)

2. **Pinyin (拼音)**:
   - `terra_pinyin_onion.schema.yaml`: Terra Pinyin mix-in variant

3. **Shape-based (形碼)**:
   - Array 30/10 (行列): `onion-array30.schema.yaml`, `onion-array10.schema.yaml`
   - OCM variants: `ocm_mixin.schema.yaml`, etc.

4. **Auxiliary Schemas** (mounted/掛載):
   - English: `easy_en_lower`, `easy_en_upper`
   - Japanese: `jpnin1`
   - Korean: `hangeul_hnc` (HNC layout)
   - Latin phonetics: `latinin1`
   - Greek: `greek`
   - Cyrillic: `cyrillic`
   - Symbols: `symbols_bpmf`, `fullshape`

## Common Development Tasks

### Deploying Schemas to Rime User Directory

**Linux (fcitx5-rime)**:
```bash
# Copy schema files to user directory
cp -r 電腦RIME方案_*/注音洋蔥plus版_*/* ~/.local/share/fcitx5/rime/

# Rebuild and restart Rime
rm -rf ~/.local/share/fcitx5/rime/build/
fcitx5 -r  # or: pkill fcitx5 && fcitx5 -d
```

**Other platforms**:
- Windows (Weasel/小狼毫): `%APPDATA%\Rime`
- macOS (Squirrel/鼠鬚管): `~/Library/Rime`
- Linux (ibus-rime): `~/.config/ibus/rime`

**Important**: Never overwrite the entire `opencc/` folder - merge files inside instead.

### Generating Distribution Packages

```bash
# Run from repository root
python sort_rime_file.py
# Creates: 電腦RIME方案_YYYYMMDD/ with organized schema folders
```

### Testing Lua Changes

After modifying Lua files in `rimefiles/lua/`:
```bash
# For fcitx5 on Linux
cp /path/to/rimefiles/lua/processor_*.lua ~/.local/share/fcitx5/rime/lua/
rm -rf ~/.local/share/fcitx5/rime/build/
fcitx5 -r
```

Lua modules are loaded via `rime.lua`, which uses `require()` to import from subdirectories.

### Checking Deployment Status

```bash
# View Rime logs (fcitx5)
journalctl -u fcitx5 --user -n 50

# Check if schemas are recognized
ls ~/.local/share/fcitx5/rime/*.schema.yaml

# Verify build output
ls ~/.local/share/fcitx5/rime/build/*.bin
```

## Version Compatibility Notes

**Critical requirements** (see README.md lines 66-86):
- **librime**: ≥1.11.2 (Nightly Build recommended)
- **librime-lua**: ≥#200
- Platform-specific:
  - Linux: Ensure librime-lua is installed (often missing in distros)
  - macOS: Squirrel ≥1.0.3 (or 0.16.2 with updated librime for older macOS)
  - Windows: Weasel 0.17.4 recommended (0.16.3+ with updated librime)

Older Rime versions **will not work** due to modern librime-lua features used throughout.

## Schema Architecture Patterns

### Modular Configuration

Schemas use YAML `__include` and `__patch` directives to share configuration:
```yaml
# In bopomo_onionplus.schema.yaml
__include: element_bopomo.yaml  # Import common settings
```

### Lua Integration

Schemas reference Lua functions in `engine:` sections:
```yaml
engine:
  processors:
    - lua_processor@mix_apc_s2rm_ltk  # Custom ASCII/punct handling
  filters:
    - lua_filter@comment_filter_unicode  # Add Unicode annotations
  translators:
    - lua_translator@multifunction_translator  # Date/time/calc functions
```

Corresponding Lua files: `processor_mix_apc_s2rm_ltk.lua`, `filter_comment_filter_unicode.lua`, etc.

### Dictionary Hierarchies

Extended dictionaries import multiple sources:
```yaml
# In bopomo_onionplus.extended.dict.yaml
import_tables:
  - terra_pinyin_onion          # Base Chinese
  - terra_pinyin_onion_add      # Additional pronunciations
  - phrases.chtp                # Phrases with pinyin
  - phrases.cht_en_w            # Chinese-English mixed
  # ... many more
```

## Key Lua Subsystems

1. **Auto-switching processors**: `processor_mix_apc_s2rm_ltk.lua` - handles ASCII/Chinese mode switching
2. **Charset filters**: `filter_mix_cf2_miss_filter.lua` - removes invalid characters, handles OpenCC artifacts
3. **Comment filters**: `filter_comment_filter_*.lua` - adds pronunciation/Unicode/debug info
4. **Multifunction translator**: `translator_multifunction_translator.lua` - uses `f_components/` for date/time/calculations
5. **Japanese conversion**: `filter_convert_japan_filter.lua` + `convert_kana/convert_kana.lua`
6. **Array30 processors**: `processor_array30new_mix.lua` - custom logic for Array input method

## Important Constraints

- **No commercial use**: Repository explicitly prohibits commercial usage (README lines 3-4)
- **Platform differences**: Schemas contain platform-specific workarounds (Windows crash prevention, macOS fcitx5 display modes)
- **OpenCC encoding issues**: Lua filters clean up "᰼᰼" artifacts from OpenCC conversions
- **Performance**: Multiple Lua filters can impact speed; some are combined (e.g., `mix_cf2_miss_filter`)

## Testing Practices

When making changes:
1. Always test deployment with `rm -rf build/ && fcitx5 -r`
2. Check logs for Lua errors: `journalctl` or console output during deployment
3. Verify all dependent schemas still load (e.g., auxiliary schemas for mixin variants)
4. Test on the target platform - Windows/macOS/Linux behaviors differ

## Documentation and Resources

- Main README: `/README.md` (installation, version requirements, donation info)
- File structure guide: `/rimefiles/README.md` (detailed file type descriptions)
- Official Wiki: https://github.com/oniondelta/Onion_Rime_Files/wiki (feature documentation in Chinese)
- Dictionary source notes: `/README_TC_Dictionary.md`
