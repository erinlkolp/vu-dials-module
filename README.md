# vudials_client

[![PyPI version](https://badge.fury.io/py/vudials_client.svg)](https://badge.fury.io/py/vudials_client)
[![CI](https://github.com/erinlkolp/vu1-dial-python-module/actions/workflows/ci.yml/badge.svg)](https://github.com/erinlkolp/vu1-dial-python-module/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)

A Python client library for Streacom's VU1 Dial development hardware. Provides a simple, tested interface for controlling multiple dials and managing the VU1 server.

## Features

- **Full Dial API**: Change a dial's value, backlight color, name, background image, and easing parameters
- **Full Admin API**: VU1 Dial Server API key management and dial provisioning
- **Type-annotated**: All public methods carry full parameter and return-type annotations
- **Well tested**: Comprehensive test suite using `pytest` and `responses` — no live hardware required
- **CI tested**: Every supported Python version (3.11+) is exercised in GitHub Actions

## Installation

```bash
pip install vudials-client
```

For development:

```bash
git clone git@github.com:erinlkolp/vu1-dial-python-module.git
cd vu1-dial-python-module
pip install -e ".[dev]"
```

## Quick Start

```python
import os
from vudials_client import vudialsclient

server_address = os.environ["VU1_SERVER_ADDRESS"]
server_port    = int(os.environ["VU1_SERVER_PORT"])
server_key     = os.environ["API_KEY"]
admin_key      = os.environ["ADMIN_API_KEY"]

vu_meter  = vudialsclient.VUDial(server_address, server_port, server_key)
admin_api = vudialsclient.VUAdmin(server_address, server_port, admin_key)

# List connected dials
dial_list = vu_meter.list_dials()
print(dial_list.json())

# Set first dial to 75% with a blue backlight
uid = dial_list.json()[0]["uid"]
vu_meter.set_dial_value(uid, 75)
vu_meter.set_dial_color(uid, 0, 0, 100)

# List API keys
api_key_list = admin_api.list_api_keys()
print(api_key_list.json())
```

> **Note:** The VU1 server communicates over plain HTTP on your local network. Keep the server on a trusted interface and treat the API keys as secrets. Both keys are passed as URL query parameters and will appear in server access logs.

## API Reference

### `VUDial`

Controls dial hardware. All methods return a `requests.Response`; HTTP 4xx/5xx errors raise `requests.exceptions.HTTPError`.

```python
vu_meter = vudialsclient.VUDial(server_address, server_port, api_key)
```

| Parameter | Type | Description |
|---|---|---|
| `server_address` | `str` | VU1 server hostname or IP (e.g. `"localhost"`) |
| `server_port` | `int` | VU1 server port (e.g. `5340`) |
| `api_key` | `str` | A valid API key for the VU1 server |

**Methods:**

| Method | Description |
|---|---|
| `list_dials()` | List all connected dials |
| `get_dial_info(uid)` | Get status/info for a specific dial |
| `set_dial_value(uid, value)` | Set the dial position (0–100) |
| `set_dial_color(uid, red, green, blue)` | Set the backlight color (0–100 each channel) |
| `set_dial_background(uid, file)` | Upload a background image file |
| `get_dial_image_crc(uid)` | Get the CRC of the current background image |
| `set_dial_name(uid, name)` | Assign a name to a dial |
| `reload_hw_info(uid)` | Reload hardware information for a dial |
| `set_dial_easing(uid, period, step)` | Configure dial movement easing |
| `set_backlight_easing(uid, period, step)` | Configure backlight easing |
| `get_easing_config(uid)` | Retrieve current easing configuration |

### `VUAdmin`

Manages the VU1 server. All methods return a `requests.Response`; HTTP 4xx/5xx errors raise `requests.exceptions.HTTPError`.

```python
admin_api = vudialsclient.VUAdmin(server_address, server_port, admin_key)
```

| Parameter | Type | Description |
|---|---|---|
| `server_address` | `str` | VU1 server hostname or IP |
| `server_port` | `int` | VU1 server port |
| `admin_key` | `str` | A valid Admin API key for the VU1 server |

**Methods:**

| Method | Description |
|---|---|
| `provision_dials()` | Provision newly connected dial hardware |
| `list_api_keys()` | List all configured API keys |
| `create_api_key(name, dials)` | Create a new API key; the generated key is in the response |
| `update_api_key(name, target_key, dials)` | Update an existing API key |
| `remove_api_key(target_key)` | Remove an API key |

## Testing

The test suite uses [`responses`](https://github.com/getsentry/responses) to mock all HTTP calls — no VU1 hardware or running server is required.

```bash
# Run all tests with coverage report
pytest tests/ -v --cov=vudials_client --cov-report=term-missing

# Run a specific test class
pytest tests/test_vudialsclient.py::TestVUDialSetDialColor -v
```

CI runs automatically on every push and pull request via GitHub Actions, across the full supported Python range (3.11+).

## Documentation

Full API documentation can be regenerated from docstrings:

```bash
pip install pydoc-markdown
pydoc-markdown
```

This writes `docs/api.md` using the configuration in `pydoc-markdown.yml`.

## Contributing

Contributions are welcome! Please open a pull request.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Make your changes and add tests for any new or modified behaviour
4. Ensure the test suite passes: `pytest tests/ -v`
5. Open a pull request against `main`

Please follow the existing code style: type-annotate all public methods, URL-encode user-supplied path segments with `quote(value, safe="")`, and keep library code free of `logging.basicConfig()` calls.

## License

MIT — see [LICENSE](LICENSE).

## Author

Erin L. Kolp (<erinlkolpfoss@gmail.com>)

Many thanks to Aaron D. and Christopher K.!
