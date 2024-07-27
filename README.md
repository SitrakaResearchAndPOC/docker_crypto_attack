# docker_crypto_attack
## GETTING OS
```  
docker pull ubuntu:20.04
```
```  
docker run -itd --privileged   --name cryptographyattack ubuntu:20.04 
```  
```  
docker exec -it cryptographyattack bash
```
## INSTALLING LENGTH EXTENSION ATTACK ON PYTHON
```  
apt update 
```  
```  
apt install pip git wget nano
```  
```  
pip install hexdump
```  
```  
git clone https://github.com/leesangmin144/length-extension-attack
```  
```  
cd length-extension-attack/
```  
```  
python3 length_extension_sha256/length_extension_sha256.py
```  
```  
cd ..
```  
## INSTALLING LENGTH EXTENSION ATTACK ON C

```  
apt install openssl openssl-dev
```  
```  
git clone https://github.com/iagox86/hash_extender
```  
```  
cd hash_extender/
```  
Open Makefile 
```  
nano Makefile 
```  
```  
https://github.com/iagox86/hash_extender/issues/9
```  
remove : -Wno-deprecated
```  
apt-get install libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev
```  
```  
make 
```  
```  
cd hash_extender/
```
```  
echo '
#include <stdio.h>
#include <openssl/md5.h>
int main(int argc, const char *argv[])
{
    MD5_CTX c;
    unsigned char buffer[MD5_DIGEST_LENGTH];
    int i;
    MD5_Init(&c);
    MD5_Update(&c, "secret", 6);
    MD5_Update(&c, "data"
                   "\x80\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00"
                   "\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00"
                   "\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00"
                   "\x00\x00\x00\x00"
                   "\x50\x00\x00\x00\x00\x00\x00\x00"
                   "append", 64);
    MD5_Final(buffer, &c);
    for (i = 0; i < 16; i++) {
    printf("%02x", buffer[i]);
    }
    printf("\n");
    return 0;
   }' > hash_extension_1.c
```  
```  
ls
```  
```  
gcc -o hash_extension_1 hash_extension_1.c -lssl -lcrypto
```  
```  
./hash_extension_1
```  
```  
cd /
```  
## INSTALLING BIT FLIPPING PYTHON CODE
```  
pip3 install crypto
```  
```  
pip install PyCryptodome
```  
```  
git  clone https://github.com/Ectario/bitflipper
```  
```  
cd bitflipper/
```  
```  
chmod +x main.py
```  
```  
./main.py -H 618be3a451f64dd93551de33e18f444f4d50e6226cf2cf3bdf0d2928a6c560973bbb504ab87e94772b8d7ab768f01764 -t "hello my name is a small test :)" -g "hello my name is a small success" -v
```  
```  
cd ..
```
## INSTALLING ORACLE PADDING By NCC RESEARCH
```    
mkdir oraclepad
```  
```  
cd oraclepad
```  
```  
nano full_attack.py 
```  
#!/usr/bin/env python3

BLOCK_SIZE = 16


def single_block_attack(block, oracle):
    """Returns the decryption of the given ciphertext block"""

    # zeroing_iv starts out nulled. each iteration of the main loop will add
    # one byte to it, working from right to left, until it is fully populated,
    # at which point it contains the result of DEC(ct_block)
    zeroing_iv = [0] * BLOCK_SIZE

    for pad_val in range(1, BLOCK_SIZE+1):
        padding_iv = [pad_val ^ b for b in zeroing_iv]

        for candidate in range(256):
            padding_iv[-pad_val] = candidate
            iv = bytes(padding_iv)
            if oracle(iv, block):
                if pad_val == 1:
                    # make sure the padding really is of length 1 by changing
                    # the penultimate block and querying the oracle again
                    padding_iv[-2] ^= 1
                    iv = bytes(padding_iv)
                    if not oracle(iv, block):
                        continue  # false positive; keep searching
                break
        else:
            raise Exception("no valid padding byte found (is the oracle working correctly?)")

        zeroing_iv[-pad_val] = candidate ^ pad_val

    return zeroing_iv


def full_attack(iv, ct, oracle):
    """Given the iv, ciphertext, and a padding oracle, finds and returns the plaintext"""
    assert len(iv) == BLOCK_SIZE and len(ct) % BLOCK_SIZE == 0

    msg = iv + ct
    blocks = [msg[i:i+BLOCK_SIZE] for i in range(0, len(msg), BLOCK_SIZE)]
    result = b''

    # loop over pairs of consecutive blocks performing CBC decryption on them
    iv = blocks[0]
    for ct in blocks[1:]:
        dec = single_block_attack(ct, oracle)
        pt = bytes(iv_byte ^ dec_byte for iv_byte, dec_byte in zip(iv, dec))
        result += pt
        iv = ct

    return result
```
```  
nano oraclepad.py
```  
#!/usr/bin/env python3

import random
import os

from Crypto.Cipher import AES  # requires PyCryptodome
from Crypto.Util.Padding import pad, unpad


class Challenge:
    _strings = (
        b"MDAwMDAwTm93IHRoYXQgdGhlIHBhcnR5IGlzIGp1bXBpbmc=",
        b"MDAwMDAxV2l0aCB0aGUgYmFzcyBraWNrZWQgaW4gYW5kIHRoZSBWZWdhJ3MgYXJlIHB1bXBpbic=",
        b"MDAwMDAyUXVpY2sgdG8gdGhlIHBvaW50LCB0byB0aGUgcG9pbnQsIG5vIGZha2luZw==",
        b"MDAwMDAzQ29va2luZyBNQydzIGxpa2UgYSBwb3VuZCBvZiBiYWNvbg==",
        b"MDAwMDA0QnVybmluZyAnZW0sIGlmIHlvdSBhaW4ndCBxdWljayBhbmQgbmltYmxl",
        b"MDAwMDA1SSBnbyBjcmF6eSB3aGVuIEkgaGVhciBhIGN5bWJhbA==",
        b"MDAwMDA2QW5kIGEgaGlnaCBoYXQgd2l0aCBhIHNvdXBlZCB1cCB0ZW1wbw==",
        b"MDAwMDA3SSdtIG9uIGEgcm9sbCwgaXQncyB0aW1lIHRvIGdvIHNvbG8=",
        b"MDAwMDA4b2xsaW4nIGluIG15IGZpdmUgcG9pbnQgb2g=",
        b"MDAwMDA5aXRoIG15IHJhZy10b3AgZG93biBzbyBteSBoYWlyIGNhbiBibG93"
    )

    def __init__(self):
        self._key = os.urandom(16)

    def get_string(self):
        """This is the first function described by Challenge 17."""
        string = random.choice(self._strings)
        cipher = AES.new(self._key, AES.MODE_CBC)
        ct = cipher.encrypt(pad(string, AES.block_size))
        return cipher.iv, ct

    def check_padding(self, iv, ct):
        """This is the second function described by Challenge 17."""
        cipher = AES.new(self._key, AES.MODE_CBC, iv)
        pt = cipher.decrypt(ct)
        try:
            unpad(pt, AES.block_size)
        except ValueError:  # raised by unpad() if padding is invalid
            return False
        return True


if __name__ == "__main__":
    from full_attack import full_attack
    from base64 import b64decode

    service = Challenge()
    iv, ct = service.get_string()
    print("Ciphertext:", ct)
    print("Launching attack...")

    result = full_attack(iv, ct, service.check_padding)
    plaintext = unpad(result, AES.block_size)
    print("Recovered plaintext:", plaintext)
    print("Decoded:", b64decode(plaintext).decode('ascii'))
```  
if not yet replaced form oraclepad import full_attack ---> from full_attack import full_attack
```  
cd ..
```
## INSTALLING BIT FLIPPING SIMPLE PYTHON CODE
```  
apt install -y xxd
```  
```  
mkdir bitflip_simple
```  
```  
cd bitflip_simple
```  
```  
nano bitflipping.py
```
```  
nano bitflipping2.py
```  
```  
cd ..
```  
``` 
docker commit  cryptographyattack cryptographyattack:latest
```
```  
docker images
```  
```  
docker save cryptographyattack:latest -o cryptographyattack.tar.gz
```  
```  
chmod 777 cryptographyattack.tar.gz 
```  
# LOAD AND RUN
```  
docker load -i cryptographyattack.tar.gz
```  
```  
docker run -itd --privileged   --name cryptographyattack  cryptographyattack:latest
```  

# TEST 1 : length extension attack on python
```  
docker exec -it cryptographyattack bash
```  
```  
cd length-extension-attack/
```  
```  
python3 length_extension_sha256/length_extension_sha256.py
```  
```  
exit
``` 

# TEST 2 : length extension attack on C
```  
docker exec -it cryptographyattack bash
```  
```  
cd hash-length-extension/hash_extender/
```  
```  
./hash_extension_1
```  
```  
./hash_extender --data data --secret 6 --append append --signature 6036708eba0d11f6ef52ad44e8b74d5b --format md5
```  
```  
exit
```  
# TEST3 : bit flipping on python
```  
docker exec -it cryptographyattack bash
```  
```  
cd bitflipper/
```  
```  
./main.py -H 618be3a451f64dd93551de33e18f444f4d50e6226cf2cf3bdf0d2928a6c560973bbb504ab87e94772b8d7ab768f01764 -t "hello my name is a small test :)" -g "hello my name is a small success" -v
```  
```  
exit
```  


# TEST4 : oracle padding by ncc researchgroup
```  
docker exec -it cryptographyattack bash
```
```  
python3 oraclepad/oraclepad.py
```  
```  
exit
```  
# TEST5 : simple bit flipping on python
```  
docker exec -it cryptographyattack bash
```  
```  
python3 bitflip_simple/bitflipping.py 
```  
```  
python3 bitflip_simple/bitflipping2.py
```  
```  
exit
```  

