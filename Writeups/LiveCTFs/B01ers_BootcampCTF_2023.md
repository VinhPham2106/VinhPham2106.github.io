# B01lers Bootcamp CTF 2023
## Context
B01lers Bootcamp CTF is an Purdue internal one, beginner friendly. Our team finished 1st place. I contributed 5 solves: 3 cryptos (all first blood), 2 miscs (1 first blood). I also went back to solve the 6/7 web challenges, so I'll also include them here. Beginner ones are explained briefly only.
## Table of Contents
* [Crypto](#crypto)
* [Misc](#misc)
* [Web](#web)
## Crypto Challenges<a name="crypto"></a>:
[The good crypto here](#good-crypto)
These challenges are pretty easy for me, but I think they're well made because attention to details and good mathematical understanding does the job, rather than relying on papers and scripts like nowaday's crypto challenges.
### drm_pad: 402 points, 50 points IMO
Description:
```
We are offering a one-time deal -- get a free sample today! But don't get greedy.

nc bootcamp.b01lers.com 13969

Author: enigcryptist
```
Attached product.py:
```
from Crypto.Random import get_random_bytes
from Crypto.Util.strxor import strxor
import random

NUM_PRODUCTS = 5
manufacturer_key = None
product_keys = []
serial_nums = []

with open("flag.txt") as f:
    flag = f.read()

class AcmeCorp:
    num_sold: int

    def __init__(self):
        self.num_sold = 0
        manufacturer_key = get_random_bytes(64)
        for item in range(NUM_PRODUCTS):
            product_keys.append(get_random_bytes(64))
            serial_nums.append(strxor(manufacturer_key, product_keys[item]))

    def browse_product(self, item: int):
        if item < 0 or item >= NUM_PRODUCTS:
            print("Are you just going to window shop all day...?")
            return

        print("Serial Number %i:" % item, serial_nums[item].hex())

    def redeem_product(self, prod_key: str):
        try:
            item = product_keys.index(bytes.fromhex(prod_key))
        except:
            print("That's not a valid product key! We have our eyes on you...")
            return
        if item < 0 or item >= NUM_PRODUCTS:
            print("We dont have unlimited products! Do you think we're made of money?")
            return
        elif product_keys[item] == None:
            print("Sorry, we are sold out of this product.")
            return

        res = random.random()
        if res == 0:
            print("Nope, sorry. Nothing.")
        elif res < pow(2, -276709):
            print("By some infitesimally small probability, the Infinite Improbability Drive apparates in front of you!")
        elif res < 0.20:
            print("You have obtained a thingamabob!")
        elif res < 0.40:
            print("You have obtained a whatchamacallit!")
        elif res < 0.60:
            print("You have obtained a gizmo!")
        elif res < 0.80:
            print("You have obtained a schmiblick!")
        elif res < 0.93:
            print("You have obtained a frob!")
        elif res < 0.9999999:
            print("You have obtained a MacGuffin!")
        elif res < 1.00:
            print("You have obtained the magical sampo!")
        else:
            print("You have obtained a unobtainium-reinforced wishalloy weapon!")
        
        #print("Product Key %i:" % item, product_keys[item].hex())
        # Product key has expired
        product_keys[item] = None
        self.num_sold += 1

def main():

    in_stock = NUM_PRODUCTS
    sample_given = False
    shop = AcmeCorp()
    while shop.num_sold < NUM_PRODUCTS:
        response = int(input("\nWelcome to AcmeCorp! We have %i products currently in stock! Would you like to (1) buy, (2) browse, (3) obtain a free sample, (4) redeem a product, or (5) leave?  " % (NUM_PRODUCTS-shop.num_sold)))
        if response == 1:
            print("You don't have any money.", end=' ')
            if not sample_given:
                print("Why don't you try out our free sample instead?")
            else:
                print("We can't just give away our entire stock for free!")

        elif response == 2:
            item = int(input("Which item on the shelf would you like to look at? (0-%i)  " % (NUM_PRODUCTS-1)))
            shop.browse_product(item)

        elif response == 3:
            if sample_given:
                print("The free sample was just for you to try out... If you want more, you gotta buy!")
            else:
                print("Here's your free sample!")
                print("Product Key 0:", product_keys[0].hex())
                shop.num_sold += 1
                sample_given = True

        elif response == 4:
            prod_key = input("Please enter your product key:  ")
            shop.redeem_product(prod_key)

        elif response == 5:
            print("Thank you for your patronage!")
            exit(1)

        else:
            print("Sorry, I don't quite understand. Please respond with 1-5.")

    print("Little do they know, but you'have redeemed their entire stock of products out from under them.")
    print(flag)

if __name__ == "__main__":
    main()
```
So we're interacting with a shop then. The basic idea here is that serial number = random_key XOR product_key, and we need to know all the product keys, use them to redeem all products to get the flag. The vulnerability here is that they give you a sample product key, which is actually the product key of item 1. We can use that and its serial number to get the random key by using XOR, and then proceed to recover all other product keys.
### Tag_check: 482 points, 100 points IMO
Description:
```
I found someone's baggage claim tag on the ground. I wonder if I can do something with this...

nc ctf.b01lers.com 46414

Author: enigcryptist
```
Attachment:
```
#!/usr/bin/env python
import math
import os
import sys
import time
from Crypto.Util.number import getPrime, bytes_to_long

with open('flag.txt') as f:
    flag = f.read()

def setup_RSA_tagging():
    p = getPrime(512)
    q = getPrime(512)
    N = p*q
    e = 65537
    phi = (p-1)*(q-1)
    # d = e^{-1} mod phi(N); use Python 3.8+
    d = pow(e, -1, phi)
    return (N, e, d)

def tag_bag(N, e, d, bag_id):
    tag = pow(bag_id, d, N)
    return tag

def verify_tag(N, e, bag_id, tag):
    return pow(tag, e, N) == (bag_id % N)

def suspense():
    for _ in range(4):
        time.sleep(1)
        print(".", end=" ")
        sys.stdout.flush()
    print()

def main():
    (N, e, d) = setup_RSA_tagging()
    lost_bag_id = 0xB0B5F01DAB1E0FF1CECA5E
    lost_tag = tag_bag(N, e, d, lost_bag_id)

    print('Looks like someone lost their baggage claim tag. It reads:\nBag ID:\t%X\nFlight:\t%X\nTag:\t%X\n' % (lost_bag_id, N, lost_tag) )

    print("Maybe if I put this tag on my own bag, I can take it on the plane for free. Should I give it a new ID first, in case they get suspicious?")
    new_bag_id = int(input("New Bag ID: "), 16)
    print("Oh no, I guess they're checking bags after all... Hope they don't notice!")
    suspense()

    assert(verify_tag(N, e, lost_bag_id+N, lost_tag))
    if not verify_tag(N, e, new_bag_id, lost_tag):
        print("\nSecurity: Announcement, suspicious baggage claim at Terminal B007. The tag on this bag looks like it was forged!")
        exit(1)
    elif new_bag_id == lost_bag_id:
        print("\nSecurity: Announcement, suspicious baggage claim at Terminal B007. The owner of this bag, \"Bob\", was not a passenger on the plane!")
        exit(1)
    else:
        print("\nWhew, that was a close one... wait, what's this scribbled on the back of the tag?")
        print(flag)


if __name__ == "__main__":
    main()
```
If you have firm grasp of RSA, and noting that the check is checking bag_id % N, you just need to put in real_id + N. The conversation basically gives you all RSA informations you need.
### Greenhouse: 498 Points, 200 points IMO <a name="good-crypto"></a>
Description:
```
It's a long day in my green house.
I heard that there is a new magic agriculture code that can help me organize my plants.

nc ctf.b01lers.com 9002

Author: bronson113
```
Attachment:
```
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from secret import flag
import os


# CBCMAC stands for Critically Secured Common Magic Agriculture Code
def mymac(key, message):
    padded_message = pad(message, 16)
    cipher = AES.new(key, AES.MODE_CBC, b"\0"*16).encrypt(padded_message)
    last_block = cipher[-16:]
    return last_block


def prompt():
    print("Welcome to my green house, I'm trying to decorate the spacious field with some unique plants.")

def main():
    prompt()

    # Generate Key
    key = os.urandom(16)

    # First Plant
    plant1 = bytes.fromhex(input("enter plant 1 in hex (00112233aabbccdd): "))
    print("The first plant: ", pad(plant1, 16).hex())

    # Plant it :)
    mac1 = mymac(key, plant1)
    print("It should be planted at: ", mac1.hex())
    
    # Second Plant
    plant2 = bytes.fromhex(input("enter plant 2 in hex (00112233aabbccdd): "))
    print("The second plant: ", pad(plant2, 16).hex())

    # No!!! I want different plants
    if plant1 == plant2:
        print("I want to have different plants in my collection!!")
        exit(1)

    # Plant it :)
    mac2 = mymac(key, plant2)
    print("It should be planted at: ", mac2.hex())

    if mac1 == mac2:
        print("No!!!! Why do they occupy the same space ><")
        print(flag)
    else:
        print("Another day, another two plants planted.")


if __name__ == "__main__":
    main()
```
The main idea is that you enter 1 hex, it gives you the last 16-byte block of its AES-CBC encrypted message. Then you enter a second hex, only if the encrypted last block matches with the previous does it give you the flag.
If you search of CBC-MAC, you'll get to explore about the [length extension attacks](https://security.stackexchange.com/questions/123793/why-is-cbc-mac-insecure-for-variable-length-input), [defense mechanisms](https://en.wikipedia.org/wiki/CBC-MAC).

Knowing that the IV is 0 and it's CBC, I was looking for a XORing technique, but the annoying thing was the padding. Here's my final solution:
* Because IV = 0, I enter plaintext1 (P), which will get padded to (P1), then encrypted to E(P1). This is also the last block, which the server will give us
* I calculate P2 = E(P1) XOR P1. Repeat until P2 has the last byte to be 0x01, I'll send P1 + P2[:-1] as the second plant and get the flag
* Why? Because when the server encrypts P1 + P2[:-1], the first encrypted block is E(P1), then it gets XOR with P2, which will give P1 and the encrypted last block will be the same. The reason to wait for the last byte to be 0x01 is the deal with the padding. So we only sent 15 bytes of P2, the last 0x01 will automatically get padded.

It was 5 minutes until the end, and I need help from my teammate to get the pwntool scripts:
```
from pwn import *

while True:
    s = remote("ctf.b01lers.com", 9002)
    s.recvuntil(b":")
    s.sendline(b"11")
    pt = bytes.fromhex("110f0f0f0f0f0f0f0f0f0f0f0f0f0f0f")
    s.recvuntil(b"at: ")
    mac1 = bytes.fromhex(s.recvline().decode())
    b = xor(pt, mac1)
    
    if b[-1] == 1:
        b = b[:-1]
        print(s.recvuntil(b": "))
        s.sendline((pt + b).hex())
        print(s.recvall())
        break
    s.close()
```
Overall I feel like this is the only brain demanding crypto, but still pretty easy :)

## Misc Challenges: <a name="misc"></a>
### Sourdough Secret: 372 points, 50 points IMO
Basically you're given an image and exiftool will give you the flag!!!!
### Read Between The Lines: 498 points, 200 points IMO
Description:
```
Look at this awesome artwork I created! I included the line data in case you wanted to recreate it. I might have used some sensitive information in the process, but Python's random module is safe so I should be okay.

Author: Bilbin
```
You're given the output.txt and the awesome-artwork.png, along with the generator script:
```
from PIL import Image, ImageDraw
import random

image_width = 64
image_height = 64

with open("flag.txt", "r") as f:
    flag = f.read()

image = Image.new("RGB", (image_width, image_height), "white")
draw = ImageDraw.Draw(image)

initial_lines = 700

def get_coords(sample):
    x1 = (sample & 0xfc000000) >> 26
    y1 = (sample & 0x3f00000) >> 20
    x2 = (sample & 0xfc000) >> 14
    y2 = (sample & 0x3f00) >> 8
    return (x1, y1, x2, y2)

# I LOVE BIT MATH
with open("output.txt", "w") as f:
    for _ in range(initial_lines):
        sample = random.getrandbits(32)
        x1, y1, x2, y2 = get_coords(sample)

        r = (sample & 0xc0)
        g = (sample & 0x38) << 2
        b = (sample & 0x7) << 5
        line_color = (r, g, b)

        draw.line([(x1, y1), (x2, y2)], fill=line_color, width=1)
        f.write(f"{x1} {y1} {x2} {y2} {r} {g} {b}\n")

    for i in range(len(flag)):
        sample = random.getrandbits(32)
        x1, y1, x2, y2 = get_coords(sample)

        flag_color = ord(flag[i]) ^ (sample & 0xff)
        r = (flag_color & 0xc0)
        g = (flag_color & 0x38) << 2
        b = (flag_color & 0x7) << 5
        line_color = (r, g, b)

        draw.line([(x1, y1), (x2, y2)], fill=line_color, width=1)
        f.write(f"{x1} {y1} {x2} {y2} {r} {g} {b}\n")

image.save("awesome_artwork.png")
```
The output image is just LMAO, we don't need to care about it, just the output file. It's easy to understand to bitwise operations being done if you paid attention in your discrete math and programming classes. The only thing problem is that for the last 25 lines of the output.txt, we need to know the generated random sample to be able to recover the flag. How so?


I was struggling until I reread the description: "**I might have used some sensitive information in the process, but Python's random module is safe so I should be okay.**". So Python random modules is not secure, Mersenne Twister as a core generator is no good, more explanation [here](https://docs.python.org/3/library/random.html). We can use [randcrack](https://github.com/tna0y/Python-random-module-cracker) recover the random generator state, as the scripts generated 700 samples, more than the 624 limit we need. Here's the **solution**:
```
from randcrack import RandCrack
import random

rc = RandCrack()
with open("output.txt", "r") as file:
    for i in range(700):
        numbers = [int(x) for x in file.readline().strip().split(" ")]
        feed = 0
        for x in range(4):
            feed += numbers[x] << (26 - 6 * x)
        feed += numbers[4]
        feed += numbers[5] >> 2
        feed += numbers[6] >> 5
        if (i in range(76, 700)):
            rc.submit(feed)
    flag = ""
    for i in range(25):
        numbers = [int(x) for x in file.readline().strip().split(" ")]
        sample = rc.predict_randrange(0, 4294967295)
        s8 = sample % 256
        char = (numbers[-3] + (numbers[-2] >> 2) + (numbers[-1] >> 5)) ^ s8
        flag += chr(char)
print(flag)
```
## Web Challenges <a name="web"></a>
Web was my strongest point but my team has a very fast teammate and nobody doing crypto so I took care of cryptos and some mathy challenges instead. Here's the summary of some of the ideas behind the web challenges:
* A Web App Game in which the **score update API can be fooled**, by Postman or BurpSuite to get an impossibly highscore and get the flag
* An **enumeration** challenge where endpoints you find will give you parts of the flag
* An **IDOR** one where you can access other's private post
* A **php curl systemescapecmd** where you can either send files to your remote server or do file:/// for LFI
* A clone of the IDOR ones, except this time IDOR doesn't work. We are given the adminbot, so we can do **XSS to steal the admin cookie** and read private contents. I discovered [webhook.site](https://webhook.site/) as a pretty neat tool to host a remote server rather than my old method which is local listener + cloudflared tunnel.
* A **SQL injection** one, but it's pretty tough because even sqlmap can't solve it. We have to craft the payload manually.
* A **harder web one with source code given**, which I haven't taken a look to see what's the vulnerabilities.
