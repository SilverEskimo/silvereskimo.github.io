```python
from fireblocks_sdk import FireblocksSDK

api_secret = open('</path/to/fireblocks_secret.key>', 'r').read()
api_key = '<your-api-key-here>'
fireblocks = FireblocksSDK(api_secret, api_key)
# If you are working with a Sandbox environment, add the sandbox URL under api_base_u
```