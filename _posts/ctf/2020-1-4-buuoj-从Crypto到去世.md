---
layout: post
title:  "buuoj-从Crypto到去世"
date:   2020-1-4
desc: ""
keywords: ""
categories: [Ctf]
tags: [CTF,buuoj,Pwn]
icon: icon-html
---



# buuoj-从Crypto到去世

### [NCTF2019]Keyboard

qwertyuiop对应1234567890

然后九宫格键盘



### [NCTF2019]childRSA

Pollard's p-1

### [NCTF2019]babyRSA

ed白给，没n

ed-1=k*phin

phin≈n，d<n，所以k<e，爆破phin

pq很接近，可以用phin开方找p-1

```python
def fuck_n(e,d):
	kphi = e*d - 1
	for k in range(1, e):
		if kphi % k == 0:
			phi = kphi // k
			root = gmpy2.iroot(phi, 2)[0]
			for p in range(root - 2000, root + 2000):
				if phi % (p-1) == 0:
					q = phi//(p-1) + 1 
					return p,q
```

### [NCTF2019]Sore

```
https://guballa.de/vigenere-solver
```



### [AFCTF2018]Vigenère

```
https://guballa.de/vigenere-solver
```

### [AFCTF2018]Morse

```
-.... .---- -.... -.... -.... ...-- --... ....- -.... -.... --... -... ...-- .---- --... ...-- ..--- --... --... ....- ..... ..-. --... ...-- ...-- ----- ..... ..-. ...-- ...-- ...-- ....- ...-- ..... --... ----. --... -..
61666374667b317327745f73305f333435797d
```

### [AFCTF2018]Single

词频分析

```
https://quipqiup.com/
```

### [AFCTF2018]你能看出这是什么加密么

RSA

```python
p1=0x928fb6aa9d813b6c3270131818a7c54edb18e3806942b88670106c1821e0326364194a8c49392849432b37632f0abe3f3c52e909b939c91c50e41a7b8cd00c67d6743b4f
q1=0xec301417ccdffa679a8dcc4027dd0d75baf9d441625ed8930472165717f4732884c33f25d4ee6a6c9ae6c44aedad039b0b72cf42cab7f80d32b74061
n1=p1*q1
e1=0x10001
c1=0x70c9133e1647e95c3cb99bd998a9028b5bf492929725a9e8e6d2e277fa0f37205580b196e5f121a2e83bc80a8204c99f5036a07c8cf6f96c420369b4161d2654a7eccbdaf583204b645e137b3bd15c5ce865298416fd5831cba0d947113ed5be5426b708b89451934d11f9aed9085b48b729449e461ff0863552149b965e22b6

phin=(p-1)*(q-1)

d1 = gmpy2.invert(e,phin)
print(hex(pow(c, d, n)))
```

### 可怜的RSA

直接分解

### [AFCTF2018]One Secret, Two encryption

模不互素

### [AFCTF2018]BASE

多次base

```python
from base64 import *
f = open('flag_encode.txt', 'r')
fuck = f.read()
while(1):
	if '{' in fuck:
		print(fuck)
		break
	try:
		fuck = b16decode(fuck)
		continue
	except:
		pass
	try:
		fuck = b32decode(fuck)
		continue
	except:
		pass
	try:
		fuck = b64decode(fuck)
		continue
	except:
		pass
```



### [AFCTF2018]MagicNum

```
72065910510177138000000000000000.000000
71863209670811371000000.000000
18489682625412760000000000000000.000000
72723257588050687000000.000000
4674659167469766200000000.000000
19061698837499292000000000000000000000.000000

```

char*转float

### [AFCTF2018]你听过一次一密么？

Many-Time-Pad Attack

```python
## OTP - Recovering the private key from a set of messages that were encrypted w/ the same private key (Many time pad attack) - crypto100-many_time_secret @ alexctf 2017
# Original code by jwomers: https://github.com/Jwomers/many-time-pad-attack/blob/master/attack.py)

import string
import collections
import sets, sys

# 11 unknown ciphertexts (in hex format), all encrpyted with the same key

c1='25030206463d3d393131555f7f1d061d4052111a19544e2e5d'
c2='0f020606150f203f307f5c0a7f24070747130e16545000035d'
c3='1203075429152a7020365c167f390f1013170b1006481e1314'
c4='0f4610170e1e2235787f7853372c0f065752111b15454e0e09'
c5='081543000e1e6f3f3a3348533a270d064a02111a1b5f4e0a18'
c6='0909075412132e247436425332281a1c561f04071d520f0b11'
c7='4116111b101e2170203011113a69001b475206011552050219'
c8='041006064612297020375453342c17545a01451811411a470e'
c9='021311114a5b0335207f7c167f22001b44520c15544801125d'
c10='06140611460c26243c7f5c167f3d015446010053005907145d'
c11='0f05110d160f263f3a7f4210372c03111313090415481d49'
ciphers = [c1, c2, c3, c4, c5, c6, c7, c8, c9, c10, c11]
# The target ciphertext we want to crack
#target_cipher = "0529242a631234122d2b36697f13272c207f2021283a6b0c7908"

# XORs two string
def strxor(a, b):     # xor two strings (trims the longer input)
    return "".join([chr(ord(x) ^ ord(y)) for (x, y) in zip(a, b)])

def target_fix(target_cipher):
	# To store the final key
	final_key = [None]*150
	# To store the positions we know are broken
	known_key_positions = set()

	# For each ciphertext
	for current_index, ciphertext in enumerate(ciphers):
		counter = collections.Counter()
		# for each other ciphertext
		for index, ciphertext2 in enumerate(ciphers):
			if current_index != index: # don't xor a ciphertext with itself
				for indexOfChar, char in enumerate(strxor(ciphertext.decode('hex'), ciphertext2.decode('hex'))): # Xor the two ciphertexts
					# If a character in the xored result is a alphanumeric character, it means there was probably a space character in one of the plaintexts (we don't know which one)
					if char in string.printable and char.isalpha(): counter[indexOfChar] += 1 # Increment the counter at this index
		knownSpaceIndexes = []

		# Loop through all positions where a space character was possible in the current_index cipher
		for ind, val in counter.items():
			# If a space was found at least 7 times at this index out of the 9 possible XORS, then the space character was likely from the current_index cipher!
			if val >= 7: knownSpaceIndexes.append(ind)
		#print knownSpaceIndexes # Shows all the positions where we now know the key!

		# Now Xor the current_index with spaces, and at the knownSpaceIndexes positions we get the key back!
		xor_with_spaces = strxor(ciphertext.decode('hex'),' '*150)
		for index in knownSpaceIndexes:
			# Store the key's value at the correct position
			final_key[index] = xor_with_spaces[index].encode('hex')
			# Record that we known the key at this position
			known_key_positions.add(index)

	# Construct a hex key from the currently known key, adding in '00' hex chars where we do not know (to make a complete hex string)
	final_key_hex = ''.join([val if val is not None else '00' for val in final_key])
	# Xor the currently known key with the target cipher
	output = strxor(target_cipher.decode('hex'),final_key_hex.decode('hex'))

	print "Fix this sentence:"
	print ''.join([char if index in known_key_positions else '*' for index, char in enumerate(output)])+"\n"

	# WAIT.. MANUAL STEP HERE 
	# This output are printing a * if that character is not known yet
	# fix the missing characters like this: "Let*M**k*ow if *o{*a" = "cure, Let Me know if you a"
	# if is too hard, change the target_cipher to another one and try again
	# and we have our key to fix the entire text!

	#sys.exit(0) #comment and continue if u got a good key

	target_plaintext = "cure, Let Me know if you a"
	print "Fixed:"
	print target_plaintext+"\n"

	key = strxor(target_cipher.decode('hex'),target_plaintext)

	print "Decrypted msg:"
	for cipher in ciphers:
		print strxor(cipher.decode('hex'),key)

	print "\nPrivate key recovered: "+key+"\n"
	
for i in ciphers:
	target_fix(i)
```

