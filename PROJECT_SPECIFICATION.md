## Project Specification: `elevenlabs-sfx` - Core Python Library for ElevenLabs Sound Effects Generation

### 1. Project Goal & Overview

The primary goal is to develop `elevenlabs-sfx`, a reusable and robust Python library. This library serves as a dedicated interface for generating sound effects using the ElevenLabs API. It encapsulates all direct API interaction logic (utilizing the official `elevenlabs` Python SDK), provides clear parameter validation, custom error handling with a retry mechanism for transient errors, and is designed for easy integration into other Python applications. The library abstracts the complexities of the ElevenLabs API for sound effect generation into a simple, well-documented Python package.

### 2. Final Project Structure

The project is organized as follows:

```
elevenlabs-sfx/
├── .coverage               # Coverage report data (generated)
├── LICENSE                 # MIT License file
├── README.md               # Project documentation
├── PROJECT_SPECIFICATION.md # This file
├── pyproject.toml          # Build system and package configuration
├── src/
│   └── elevenlabs_sfx/
│       ├── __init__.py     # Makes the library a package and exports main components
│       ├── client.py       # Contains the core ElevenLabsSFXClient class
│       └── exceptions.py   # Defines custom exceptions for the library
└── tests/
    ├── __init__.py         # Makes the tests directory a package
    └── test_client.py      # Unit tests for the client.py module
```

### 3. Core Library Implementation (`src/elevenlabs_sfx/`)

#### 3.1. `exceptions.py`

This module defines custom exceptions for the library to provide specific error information to consumers.

*   **`ElevenLabsSFXError(Exception)`**: Base exception for all library-specific errors.
*   **`ElevenLabsParameterError(ValueError, ElevenLabsSFXError)`**: Raised for invalid parameters passed to library functions (e.g., out-of-range `duration_seconds` or empty API key). Inherits from `ValueError`.
*   **`ElevenLabsAPIError(ElevenLabsSFXError)`**: Base exception for errors originating from the ElevenLabs API or SDK.
    *   `__init__(self, message: str, status_code: int | None = None, original_error: Exception | None = None)`
    *   `__str__` method prepends status code to the message if available.
*   **`ElevenLabsAPIKeyError(ElevenLabsAPIError)`**: Raised for API authentication errors (HTTP 401).
    *   Default message: "Invalid API key or authentication failed."
    *   Default `status_code`: 401.
*   **`ElevenLabsRateLimitError(ElevenLabsAPIError)`**: Raised when API rate limits are exceeded (HTTP 429).
    *   Default message: "API rate limit exceeded."
    *   Default `status_code`: 429.
*   **`ElevenLabsPermissionError(ElevenLabsAPIError)`**: Raised for permission-related errors (HTTP 403).
    *   Default message: "Permission denied. The API key may lack necessary permissions."
    *   Default `status_code`: 403.
*   **`ElevenLabsGenerationError(ElevenLabsAPIError)`**: Raised for general errors during sound generation not covered by other specific exceptions (e.g., HTTP 400, HTTP 5xx).

#### 3.2. `client.py` - `ElevenLabsSFXClient` Class

This class is the main interface for the library.

*   **Class Constants**:
    *   `DEFAULT_MAX_RETRIES: ClassVar[int] = 3`
    *   `DEFAULT_BACKOFF_FACTOR: ClassVar[float] = 1.0` (seconds)
    *   `DEFAULT_DURATION_SECONDS: ClassVar[float] = 5.0`
    *   `DEFAULT_PROMPT_INFLUENCE: ClassVar[float] = 0.3`
    *   `DEFAULT_OUTPUT_FORMAT: ClassVar[str] = "mp3_44100_128"`
    *   `MIN_DURATION: ClassVar[float] = 0.5`
    *   `MAX_DURATION: ClassVar[float] = 22.0`
    *   `MIN_INFLUENCE: ClassVar[float] = 0.0`
    *   `MAX_INFLUENCE: ClassVar[float] = 1.0`

*   **`__init__(self, api_key: str, max_retries: int = DEFAULT_MAX_RETRIES, backoff_factor: float = DEFAULT_BACKOFF_FACTOR)`**:
    *   Initializes the client.
    *   Parameters:
        *   `api_key: str`: The ElevenLabs API key. Raises `ElevenLabsParameterError` if empty.
        *   `max_retries: int`: Maximum number of retries for transient API errors.
        *   `backoff_factor: float`: Base factor for exponential backoff delay.
    *   Instantiates the underlying SDK client: `self.client = elevenlabs.client.ElevenLabs(api_key=api_key)`.
    *   Sets up a logger (`logging.getLogger(__name__)`) with a `logging.NullHandler()` to be silent by default.

*   **`generate_sound_effect(self, text: str, duration_seconds: float = DEFAULT_DURATION_SECONDS, prompt_influence: float = DEFAULT_PROMPT_INFLUENCE, output_format: str = DEFAULT_OUTPUT_FORMAT) -> bytes`**:
    *   **Parameter Validation**:
        *   `duration_seconds`: Checks if between `MIN_DURATION` and `MAX_DURATION`. Raises `ElevenLabsParameterError` if not.
        *   `prompt_influence`: Checks if between `MIN_INFLUENCE` and `MAX_INFLUENCE`. Raises `ElevenLabsParameterError` if not.
        *   `text`: Checks if not empty or whitespace only. Raises `ElevenLabsParameterError` if it is.
    *   **API Interaction**:
        *   Calls `self.client.text_to_sound_effects.convert(...)` with the provided parameters.
        *   The result from `convert()` is an iterator/generator. This iterator is consumed using `consumed_audio_bytes = b"".join(audio_bytes)` to get the full audio data as `bytes`.
    *   **Return Value**: Returns the concatenated audio data as `bytes`.
    *   **Error Handling and Retry Logic**:
        *   A `for` loop implements retries up to `self.max_retries`.
        *   Catches `elevenlabs.core.api_error.ApiError` (aliased as `ElevenLabsSDKAPIError`).
        *   Extracts `status_code` and `error_message` (from `e.body` if possible, otherwise `str(e)`).
        *   **Status Code Mapping to Custom Exceptions**:
            *   `401`: Raises `ElevenLabsAPIKeyError`.
            *   `403`: Raises `ElevenLabsPermissionError`.
            *   `400`: Raises `ElevenLabsGenerationError`.
            *   `429` (Rate Limit): If retries remain, sleeps using exponential backoff (`backoff_factor * (2 ** attempt) + jitter`) and continues. If retries are exhausted, raises `ElevenLabsRateLimitError`.
            *   `>= 500` (Server Error): If retries remain, sleeps using exponential backoff and continues. If retries are exhausted, raises `ElevenLabsGenerationError` (message includes original status code).
            *   Other SDK API errors: Raises a generic `ElevenLabsAPIError`.
        *   Catches any other `Exception` and wraps it in `ElevenLabsSFXError` to prevent leaking unrelated exceptions.
        *   If the loop completes without returning (shouldn't happen with proper error raising), raises `ElevenLabsGenerationError`.

#### 3.3. `__init__.py` (in `src/elevenlabs_sfx/`)

*   Defines `__version__ = "0.1.0"`.
*   Exports the main client and all custom exceptions via `__all__`:
    ```python
    __all__ = [
        "ElevenLabsSFXClient",
        "ElevenLabsSFXError",
        "ElevenLabsAPIError",
        "ElevenLabsAPIKeyError",
        "ElevenLabsRateLimitError",
        "ElevenLabsPermissionError",
        "ElevenLabsGenerationError",
        "ElevenLabsParameterError",
        "__version__",
    ]
    ```

### 4. Packaging (`pyproject.toml`)

*   **Build System**: Uses `hatchling` (`requires = ["hatchling"]`, `build-backend = "hatchling.build"`).
*   **Project Metadata (`[project]`)**:
    *   `name = "elevenlabs-sfx"`
    *   `version = "0.1.0"` (dynamically sourced from `src/elevenlabs_sfx/__init__.py` via `[tool.hatch.version]`)
    *   `description`, `readme`, `requires-python = ">=3.8"`, `license = {text = "MIT"}`, `authors`, `classifiers`.
*   **Dependencies (`[project.dependencies]`)**:
    *   `elevenlabs>=1.2.0,<2.0.0` (Note: SDK v1.58.1 was used during development and testing).
*   **Optional Dependencies for Testing (`[project.optional-dependencies]`)**:
    *   `test = ["pytest>=7.0", "pytest-cov>=3.0", "requests-mock>=1.9.0"]`
*   **Hatch Configuration (`[tool.hatch...]`)**:
    *   `[tool.hatch.version]`: `path = "src/elevenlabs_sfx/__init__.py"`
    *   `[tool.hatch.build.targets.sdist]` and `[tool.hatch.build.targets.wheel]` define included files/packages.
    *   `[tool.hatch.envs.default]`:
        *   `skip-install = false`
        *   Lists dependencies for the default test environment, including `elevenlabs`, `pytest`, `pytest-cov`, `requests-mock`.
    *   `[tool.hatch.envs.default.scripts]`:
        *   `test = "python -m pytest {args}"`
        *   `cov = "python -m pytest --cov-report=html --cov-report=term {args}"`
*   **Pytest Configuration (`[tool.pytest.ini_options]`)**:
    *   `addopts = "-ra -q --cov=src/elevenlabs_sfx --cov-report=term-missing --cache-clear"`
    *   `testpaths = ["tests"]`
*   **Coverage Configuration (`[tool.coverage.run]`, `[tool.coverage.report]`)**:
    *   `source = ["src/elevenlabs_sfx"]`
    *   `show_missing = true`
    *   `fail_under = 80` (Achieved 95.58% coverage)

### 5. Documentation

*   **`README.md`**:
    *   Provides an overview, features, installation instructions (PyPI and local/Git).
    *   Includes Quickstart/Usage examples for basic use and custom parameters, demonstrating error handling.
    *   Explains API key management.
    *   Lists and describes custom exceptions.
    *   Provides instructions for enabling library logging.
    *   Mentions MIT license.
*   **`LICENSE`**: Contains the full text of the MIT License.
*   **Docstrings**: PEP 257 compliant docstrings are provided for all public modules, classes, methods, and functions.

### 6. Testing (`tests/`)

*   **Framework**: `pytest` along with `pytest-cov` for coverage.
*   **`tests/test_client.py`**: Contains unit tests for `ElevenLabsSFXClient`.
    *   **Key Fixtures**:
        *   `mock_elevenlabs_sdk_client`: Patches `elevenlabs_sfx.client.ElevenLabs` (where `ElevenLabs` is looked up by the SUT) using `unittest.mock.patch` with `autospec=True`. The returned mock instance has `text_to_sound_effects` as a `MagicMock`, whose `convert` method is also a `MagicMock`.
        *   `sfx_client`: Provides an `ElevenLabsSFXClient` instance initialized with the `mock_elevenlabs_sdk_client`.
        *   `mock_time_sleep`: Patches `time.sleep` to prevent actual delays.
    *   **Test Categories**:
        *   **Client Initialization**: Success with default/custom retries, failure with empty API key.
        *   **Parameter Validation**: For `duration_seconds`, `prompt_influence`, and `text` in `generate_sound_effect`.
        *   **Successful Generation**:
            *   Default parameters: Mocks `convert` to return `[b"mock_audio_data"]` (an iterable).
            *   Custom parameters: Mocks `convert` similarly.
        *   **API Error Handling**:
            *   For each relevant HTTP status code (401, 403, 400, 418 for unhandled), an `elevenlabs.core.api_error.ApiError` instance is created with the appropriate `status_code` and `body`.
            *   The `mock_elevenlabs_sdk_client.text_to_sound_effects.convert.side_effect` is set to this error instance (so the mock `convert` call *raises* the error).
            *   Asserts that the correct custom exception from `elevenlabs_sfx.exceptions` is raised and contains the original SDK error and correct status code.
        *   **Retry Mechanism**:
            *   Tests successful generation after retrying on 429 or 5xx errors (using a list for `side_effect`: `[error_instance, success_iterable]`).
            *   Tests exhaustion of retries for persistent 429 or 5xx errors (using a list of error instances for `side_effect`).
            *   Asserts call counts for `convert` and `time.sleep`.
        *   **Unexpected Errors**: Tests that a non-SDK error (e.g., `ValueError`) during `convert` is wrapped in `ElevenLabsSFXError`.
        *   **SDK Error Body Parsing**: Verifies that messages are correctly extracted from the `detail` field of the SDK error's `body`.
        *   **No-Retry Scenarios**: Confirms that non-transient errors (401, 400, 403) do not trigger retries.
    *   **Test Coverage**: Achieved 95.58%, exceeding the 80% target.

### 7. Key Technical Decisions & Learnings During Development

*   **`ApiError` Import**: The correct import for the SDK's API error class (for `elevenlabs` SDK v1.58.1) was identified as `from elevenlabs.core.api_error import ApiError`.
*   **SDK `convert` Method Return**: The `elevenlabs.client.ElevenLabs().text_to_sound_effects.convert()` method returns an iterator. The library client consumes this using `b"".join()`.
*   **Mocking Strategy**:
    *   The `ElevenLabs` client is mocked by patching `elevenlabs_sfx.client.ElevenLabs`.
    *   For success cases, the mocked `convert` method's `return_value` is set to an iterable (e.g., a list containing a bytes object).
    *   For error cases, the mocked `convert` method's `side_effect` is set to an instance of the SDK's `ApiError` (or other exceptions like `ValueError`) to simulate the SDK raising an error.
*   **`ApiError` Constructor**: The `elevenlabs.core.api_error.ApiError` constructor uses keyword arguments `status_code` and `body`.
*   **Test Debugging**: Iterative debugging of tests revealed issues with mock targets, SDK method return types (iterator vs. direct bytes), and the exact constructor signature and string representation of the SDK's `ApiError`.

---
