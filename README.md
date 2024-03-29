# azure_keyvault_ethereum_py

![Build](https://travis-ci.org/davidp94/azure-keyvault-ethereum-py.svg?branch=master)

Let you sign your ethereum transaction using Azure Keyvault


```bash
pip install azure-keyvault-ethereum-py
```

## Example

See https://github.com/davidp94/signing-azure-keyvault-secp256k1-ethereum

```python
from azure_keyvault_ethereum_py import sign_keyvault

from azure.common.credentials import ServicePrincipalCredentials
from azure.keyvault import KeyVaultClient, KeyVaultAuthentication
from ethtoken.abi import EIP20_ABI
from web3 import Web3, HTTPProvider

from config import CLIENT_ID, PASSWORD, TENANT_ID, VAULT_URL, KEY_NAME, KEY_VERSION

def auth_callback(server, resource, scope):
    credentials = ServicePrincipalCredentials(
        client_id=CLIENT_ID,
        secret=PASSWORD,
        tenant=TENANT_ID,
        resource='https://vault.azure.net'
    )
    token = credentials.token
    return token['token_type'], token['access_token']


client = KeyVaultClient(KeyVaultAuthentication(auth_callback))

w3 = Web3(HTTPProvider('http://localhost:8545'))

unicorns = w3.eth.contract(address="0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359", abi=EIP20_ABI)

unicorn_txn = unicorns.functions.transfer(
    '0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359',
    1,
).buildTransaction({
    'gas': 70000,
    'gasPrice': w3.toWei('1', 'gwei'),
    'nonce': 0,
})

address_signer, signed_transaction = sign_keyvault(client, VAULT_URL, KEY_NAME, KEY_VERSION, unicorn_txn)

print("Signer", address_signer)
print("Signed Tx", signed_transaction.hex())

tx_hash = w3.eth.sendRawTransaction(signed_transaction.hex())

print("tx hash", tx_hash.hex())

```


### Rebuild and Distribute

```bash
python3 setup.py sdist bdist_wheel
```

Upload to PyPI

```bash
twine upload dist/*
```