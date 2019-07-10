

```python
import os
import hashlib
import marshal
from pyeoskit import eosapi, wallet, db, config
db.reset()
config.main_token = 'SYS'

if os.path.exists('test.wallet'):
    os.remove('test.wallet')
psw = wallet.create('test')
wallet.import_key('test', '5KH8vwQkP4QoTwgBtCV5ZYhKmv8mx56WeNrw9AZuhNRXTrPzgYc')

def publish_contract(account_name, code, abi,):
    m = hashlib.sha256()
    code = compile(code, "contract", 'exec')
    code = marshal.dumps(code)
    m.update(code)
    code_hash = m.hexdigest()
    r = eosapi.get_code(account_name)
    if code_hash != r['code_hash']:
        eosapi.set_contract(account_name, code, abi, 1)
    return True

def publish_move_contract(account_name, code, abi,):
    m = hashlib.sha256()
    m.update(code)
    code_hash = m.hexdigest()
    r = eosapi.get_code(account_name)
    if code_hash != r['code_hash']:
        eosapi.set_contract(account_name, code, abi, 3)
    return True

#eosapi.set_nodes(['https://nodes.uuos.network:8443'])
eosapi.set_nodes(['http://127.0.0.1:8888'])


```


```python
eosapi.get_public_key('5KH8vwQkP4QoTwgBtCV5ZYhKmv8mx56WeNrw9AZuhNRXTrPzgYc')
```




    'EOS7ent7keWbVgvptfYaMYeF2cenMBiwYKcwEuc11uCbStsFKsrmV'




```python
eosapi.get_balance('eosio')
```




    10000000000000.002




```python
code = r'''
modules:
module Debug {
    native public print(data: bytearray);
}

script:
import 0x0.Debug;
main() {
    let input: bytearray;
    let r: bool;
    input = b"68656c6c6f2c776f726c64";
    Debug.print(copy(input));
    return;
}
'''
open('test.mvir', 'w').write(code)
%system /Users/newworld/dev/eos-1.8/externals/libra/target/release/compiler -o test.mvir.o test.mvir
```




    []




```python
code = open('test.mvir.o', 'rb').read()

abi = ''
account_name = 'uuos'
publish_move_contract(account_name, code, abi)
try:
    r = eosapi.push_action(account_name, 'sayhello', b'hello,world', {account_name:'active'})
    print(r['processed']['action_traces'][0]['console'])
    print(r['processed']['elapsed'])
except Exception as e:
    print(e)
```

    hello,world
    329

