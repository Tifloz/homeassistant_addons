# Patch Notes for I2C OLED Add-on

## fix-default-params.patch

### Issue
The application was crashing with `OSError: [Errno 5] I/O error` during module import because:

1. The `BaseScreen.__init__` method in `bin/Screens.py` had default parameters:
   ```python
   def __init__(self, duration, display = Display(), utils = Utils(), config = None)
   ```

2. In Python, default parameter values are evaluated **once** when the function is defined (at module import time), not each time the function is called.

3. The `Display()` constructor immediately attempts to initialize I2C hardware by calling:
   - `SSD1306_128_64.__init__()` 
   - `self.clear()` which calls `self.display.begin()`
   - This ultimately calls `self._bus.write_byte_data()` to communicate with the I2C device

4. If the I2C device is not available during module import, the application crashes before it can even check for hardware availability.

### Solution
Changed the default parameters from instantiated objects to `None`:

```python
def __init__(self, duration, display = None, utils = None, config = None):
    self.display = display if display is not None else Display()
    self.duration = duration
    self.utils = utils if utils is not None else Utils()
    self.config = config
```

This defers the instantiation of `Display()` and `Utils()` until the `BaseScreen` constructor is actually called, not when the module is imported.

### Impact
- **Backward Compatible**: All existing code that passes explicit `display` and `utils` parameters (like `Config.screen_factory()` does) continues to work unchanged.
- **Fixes Import Error**: The module can now be imported successfully even when I2C hardware is not available.
- **Best Practice**: Follows Python best practices by avoiding mutable defaults and side effects in default parameters.

### References
- Python Documentation: [Default Parameter Values](https://docs.python.org/3/tutorial/controlflow.html#default-argument-values)
- Related upstream repository: https://github.com/Tifloz/rpi_i2c_oled
