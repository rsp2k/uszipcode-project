# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**uszipcode** is a comprehensive Python library for US zipcode data analysis and search. It provides a rich API for querying zipcode information including demographics, geography, real estate, and census data from 2010-2020.

## Architecture

### Core Components

- **`uszipcode/search.py`**: Main SearchEngine class that provides the primary API for zipcode queries
- **`uszipcode/model.py`**: SQLAlchemy ORM models for `SimpleZipcode` and `ComprehensiveZipcode` data structures
- **`uszipcode/db.py`**: Database management utilities for downloading and managing SQLite databases
- **`uszipcode/state_abbr.py`**: State abbreviation mappings and utilities

### Database Architecture

The library uses two SQLite databases:
- **Simple DB** (`~/.uszipcode/simple_db.sqlite`): Basic zipcode, location, and geographic data
- **Comprehensive DB** (`~/.uszipcode/comprehensive_db.sqlite`): Includes detailed census demographics, income, housing, education data

Databases are automatically downloaded from GitHub releases on first use.

### Search Engine Pattern

The `SearchEngine` class supports multiple query patterns:
- By zipcode: `search.by_zipcode()`
- By coordinates: `search.by_coordinates()`
- By city/state: `search.by_city_and_state()`
- By value ranges: population, income, geographic bounds, etc.

## Development Commands

### Modern Development with uv
```bash
# Initialize project and install all dependencies
uv sync --dev

# Run tests with coverage
uv run pytest tests --cov=uszipcode

# Run specific test module
uv run pytest tests/test_search_1_helpers.py -v

# Run tests using the convenience script
uv run python tests/all.py
```

### Code Quality with Ruff
```bash
# Lint and auto-fix code
uv run ruff check uszipcode/ --fix

# Format code
uv run ruff format uszipcode/

# Check both linting and formatting
uv run ruff check uszipcode/ && uv run ruff format uszipcode/ --check
```

### Multi-Version Testing
```bash
# Test across Python versions with tox (3.8-3.12)
tox

# Test with modern uv approach
tox -e uv

# Test specific Python version
tox -e py312
```

### Legacy Development (Deprecated)
```bash
# Legacy testing approach
python -m pytest tests --cov=uszipcode
pip install -e .
pip install -r requirements.txt
pip install -r requirements-test.txt

# Legacy formatting (replaced by ruff)
python fix_code_style.py
```

### Database Management

The SearchEngine automatically handles database downloads, but for manual management:
- Simple DB URL: `https://github.com/MacHu-GWU/uszipcode-project/releases/download/1.0.1.db/simple_db.sqlite`
- Comprehensive DB URL: `https://github.com/MacHu-GWU/uszipcode-project/releases/download/1.0.1.db/comprehensive_db.sqlite`
- Default location: `~/.uszipcode/`

## Key Dependencies

- **SQLAlchemy 2.0+**: Core ORM and database operations
- **sqlalchemy_mate**: Extended SQLAlchemy utilities and compressed JSON types
- **fuzzywuzzy**: Fuzzy string matching for city names
- **python-Levenshtein**: C-accelerated string matching (70-100x faster than pure Python)
- **haversine**: Distance calculations between coordinates
- **requests**: HTTP client for database downloads
- **pathlib_mate**: Enhanced path operations

## Testing Strategy

Tests are organized by functionality:
- `test_search_1_helpers.py`: Utility functions and validation
- `test_search_2_by_zipcode.py`: Core zipcode lookup functionality
- `test_search_3_by_value_range.py`: Range-based queries
- `test_search_4_by_city_and_state.py`: Geographic search
- `test_search_5_by_coordinate.py`: Coordinate-based search
- `test_search_6_census_data.py`: Census data validation
- `test_search_9_custom_engine.py`: Custom database configurations

## Important Implementation Notes

### Data Models
- `SimpleZipcode`: Basic model with ~20 fields (location, basic demographics)
- `ComprehensiveZipcode`: Extended model with 200+ fields including detailed census data
- Both inherit from `AbstractSimpleZipcode` base class

### Performance Considerations
- Database files are downloaded and cached locally on first use
- SQLAlchemy session management handled internally by SearchEngine
- Spatial queries use lat/lng indexes for performance
- Large result sets support pagination and limiting

### State Handling
Uses comprehensive state abbreviation mappings supporting:
- Two-letter codes (CA, NY, TX)
- Full state names (California, New York, Texas)
- Fuzzy matching for common variations and misspellings