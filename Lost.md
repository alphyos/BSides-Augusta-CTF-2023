# 🤷Lost - 400pts

```
💡 Just kinda cryptography
```

### Required files

[output.txt](/assets/output.txt)

[challenge.py](/assets/challenge.py)

### Challenge.py Explanation

```python
#!/usr/bin/python3
from Crypto.Util.number import getPrime, long_to_bytes, inverse
flag = open('flag.txt', 'r').read().strip().encode()

class RSA:
    def init(self):
        self.p = getPrime(512)
        self.q = getPrime(512)
        self.e = 3
        self.n = self.p self.q
        self.d = inverse(self.e, (self.p-1)*(self.q-1))
    def encrypt(self, data: bytes) -> bytes:
        pt = int(data.hex(), 16)
        ct = pow(pt, self.e, self.n)
        return long_to_bytes(ct)
    def decrypt(self, data: bytes) -> bytes:
        ct = int(data.hex(), 16)
        pt = pow(ct, self.d, self.n)
        return long_to_bytes(pt)

def main():
    crypto = RSA()
    print ('Flag:', crypto.encrypt(flag).hex())

if name == 'main':
    main()
```

Basically, the flag is encoded and they give you a function to decrypt it in a way that you have to `repeatedly` or `recursively` decode the message until it is `human-readable` or at least `flag legible`

- `output.txt` that they provide you is the encoded flag you must decode
- It is encoded with `hex`
- You seriously do not have to understand the majority of these lines

You could write a program to accomplish this:
