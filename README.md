# ElevenLabs SFX Python Library (`elevenlabs-sfx`)

[![PyPI version](https://badge.fury.io/py/elevenlabs-sfx.svg)](https://badge.fury.io/py/elevenlabs-sfx) <!-- Placeholder, will work once published -->
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

`elevenlabs-sfx` is a Python library designed to simplify the generation of sound effects using the [ElevenLabs API](https://elevenlabs.io/). It provides a robust, easy-to-use interface, encapsulating direct API interactions, error handling, and retry mechanisms.

## Features

*   Simple, class-based client for API interactions (`ElevenLabsSFXClient`).
*   Straightforward function to generate sound effects (`generate_sound_effect`).
*   Validation for parameters like `duration_seconds` and `prompt_influence`.
*   Automatic retries with exponential backoff for transient API errors (e.g., rate limits, server issues).
*   Specific custom exceptions for fine-grained error handling (e.g., `ElevenLabsAPIKeyError`, `ElevenLabsRateLimitError`, `ElevenLabsGenerationError`).
*   Configurable retry attempts and backoff factor.
*   Internal logging using the standard `logging` module (silent by default).
*   Built with Python 3.8+ and the official `elevenlabs` Python SDK.

## Installation

You will be able to install `elevenlabs-sfx` from PyPI once it's published:

```bash
pip install elevenlabs-sfx
```

For now, to install it locally or from a Git repository:

```bash
pip install .
# or
pip install git+https://github.com/your-username/elevenlabs-sfx.git # Replace with actual URL
```

You'll also need an API key from [ElevenLabs](https://elevenlabs.io/).

## Quickstart / Usage Examples

First, ensure you have your ElevenLabs API key.

```python
import os
import logging
from elevenlabs_sfx import (
    ElevenLabsSFXClient,
    ElevenLabsParameterError,
    ElevenLabsAPIKeyError,
    ElevenLabsRateLimitError,
    ElevenLabsGenerationError,
    ElevenLabsSFXError
)

# Optional: Configure logging to see library's internal logs
# logging.basicConfig(level=logging.INFO) # For general info
# logging.basicConfig(level=logging.DEBUG) # For detailed debug messages

# --- Basic Usage ---
try:
    # It's recommended to get the API key from an environment variable or secure store
    api_key = os.environ.get("ELEVENLABS_API_KEY")
    if not api_key:
        raise ValueError("ELEVENLABS_API_KEY environment variable not set.")

    client = ElevenLabsSFXClient(api_key=api_key)

    prompt_text = "A futuristic spaceship door opening with a hiss."
    
    print(f"Generating sound for: '{prompt_text}'")
    audio_data = client.generate_sound_effect(text=prompt_text)

    # Save the audio data to a file
    with open("spaceship_door.mp3", "wb") as f:
        f.write(audio_data)
    print("Sound effect 'spaceship_door.mp3' saved successfully.")

except ElevenLabsParameterError as e:
    print(f"Parameter Error: {e}")
except ElevenLabsAPIKeyError as e:
    print(f"API Key Error: {e}")
except ElevenLabsRateLimitError as e:
    print(f"Rate Limit Error: {e}")
except ElevenLabsGenerationError as e:
    print(f"Generation Error: {e}")
except ElevenLabsSFXError as e: # Catch-all for other library-specific errors
    print(f"SFX Library Error: {e}")
except ValueError as e: # For the API key check in this example
    print(f"Configuration error: {e}")
except Exception as e:
    print(f"An unexpected error occurred: {e}")


# --- Usage with Custom Parameters ---
try:
    api_key = os.environ.get("ELEVENLABS_API_KEY")
    if not api_key:
        raise ValueError("ELEVENLABS_API_KEY environment variable not set.")

    # Configure client with custom retry settings
    custom_client = ElevenLabsSFXClient(
        api_key=api_key,
        max_retries=5,
        backoff_factor=0.5
    )

    short_sound_prompt = "a quick button click"
    print(f"\nGenerating sound for: '{short_sound_prompt}' with custom settings")
    audio_data_custom = custom_client.generate_sound_effect(
        text=short_sound_prompt,
        duration_seconds=1.5,       # Shorter duration
        prompt_influence=0.8,       # Higher influence
        output_format="mp3_44100_192" # Higher quality MP3
    )

    with open("button_click_custom.mp3", "wb") as f:
        f.write(audio_data_custom)
    print("Sound effect 'button_click_custom.mp3' saved successfully.")

except Exception as e:
    print(f"An error occurred in custom example: {e}")

```

## API Key Management

An ElevenLabs API key is required to use this library. You should pass your API key when initializing the `ElevenLabsSFXClient`:

```python
client = ElevenLabsSFXClient(api_key="YOUR_ELEVENLABS_API_KEY")
```

It is highly recommended to manage your API key securely, for example, by using environment variables or a secrets management system, rather than hardcoding it into your application.

## Error Handling

The library raises specific exceptions to allow for robust error handling:

*   `ElevenLabsSFXError`: Base exception for all errors specific to this library.
*   `ElevenLabsParameterError(ValueError, ElevenLabsSFXError)`: For invalid parameters passed to library functions (e.g., `duration_seconds` out of range).
*   `ElevenLabsAPIError(ElevenLabsSFXError)`: Base for API-related errors.
    *   `ElevenLabsAPIKeyError(ElevenLabsAPIError)`: For authentication issues (e.g., HTTP 401 - invalid API key).
    *   `ElevenLabsRateLimitError(ElevenLabsAPIError)`: For rate limiting errors (e.g., HTTP 429).
    *   `ElevenLabsPermissionError(ElevenLabsAPIError)`: For permission issues (e.g., HTTP 403 - API key lacks permissions).
    *   `ElevenLabsGenerationError(ElevenLabsAPIError)`: For general issues during sound generation not covered by other specific exceptions (e.g., HTTP 400 - bad prompt, HTTP 5xx - server errors).

Refer to the Quickstart section for an example of how to catch these exceptions.

## Logging

The library uses Python's standard `logging` module for internal logging. By default, these logs are not output to the console. If you want to see them (e.g., for debugging), you can configure the logger in your application:

```python
import logging

# To see informational messages and errors from the library:
logging.basicConfig(level=logging.INFO)

# For more detailed debug messages:
# logging.getLogger("elevenlabs_sfx").setLevel(logging.DEBUG)
# logging.basicConfig(level=logging.DEBUG) # This would also work if you want all debug logs
```

## Contributing (Optional)

Contributions are welcome! If you'd like to contribute, please feel free to fork the repository, make your changes, and submit a pull request. For major changes, please open an issue first to discuss what you would like to change.

(Further details on development setup, running tests, etc., can be added here if desired.)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
