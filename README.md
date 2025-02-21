RandSteg
========

Randsteg is a program that hides your encrypted password into a png file.
Its name is short for RANDomized STEGanography.

General idea: you can hide your data into a png file by using bit operations on to *random* pixels of an image. 


Warning
-------

!!: Although the idea of storing your password into a png is pretty neat, it is not recommended as an actual security layer


Dependencies
-----------

- openssl (only 3.0.2 has been tested yet)
- libpng (only 1.6.37 has been tested yet)


Installation
------------

After cloning repo, run following cmake command to make the build folder:

```console
$ cmake -B build -D CMAKE_BUILD_TYPE=Debug
```

Run the following make command to build binary:

```console
$ make -C build
```


Commands
--------

#### Initializing masterkey
```console
$ randsteg -i
```

#### Distributing bits to png file
```console
$ randsteg FILEPATH_TO_PNG
```

1. You will be asked to type a password id, which will be used to identify the password you are trying to display later
2. Type the actual password you want to hide
3. Type the masterkey and press ENTER

#### Display hidden password from png file
```console
$ randsteg -d PASSWORD_ID FILEPATH_TO_PNG
```

#### Remove password
```console
$ randsteg -r PASSWORD_ID
```

!!: This will only remove the log entry of your password, but NOT the image, where the bits were injected!
    -> Make sure you delete the png file afterwards!

#### List all password IDs
```console
$ randsteg -l
```

#### Display help
```console
$ randsteg -h
```


What is Steganography?
----------------------

Steganography is a method of hiding information into a data of a photo. It takes advantage of the Least Significant Bit (LSB) of a pixel.

Example: Suppose a pixel is represented by 8 bits. 
The last bit is the LSB and switching this bit into the opposite value wouldn't make as much of a difference so that it would be impossible for the human eye to detect any difference.
With this information in mind, one could convert a plain text data into its binary form and distribute each of the bits into pixels, which "hides the data in a photo".
Theoretically, one could even hide a whole malware into a photo, which could be a potential attack trajectory.


Why use Randsteg?
-----------------

1. hide your encrypted password (via AES-256-CBC with PBKDF2 hashed masterkey) and not the password itself
2. hide individual bits of the ciphertext into cryptographically secure randomized coordinates of pixels throughout the png file
3. only log the encrypted coordinates and not the ciphertext, which will be used for decryption (along side with an id of the password, length of ciphertext and more stuff for necessary logging)
4. turn your png of your puppy into a password storage (which is pretty neat)

The major feature of randsteg is point number 2. ~(and 4)~
  - This doesn't prevent the issue of target bits to be potentially detected.
  - However, it would be hard for a detection program to make sense out of the extracted bits, as they will be randomly distributed throughout the png file and the only way to keep the order of the bits is to crack the hash in the log file or bruteforcing combinations of the bits.


Detailed Procedure
------------------

#### Procedure: bits injection
Randsteg will
1. encrypt inputted password via AES-256-CBC with PBKDF2 hashed masterkey
2. use a CSPRNG (Cryptographically Secure Pseudorandom Number Generator) from Openssl lib for the randomized coordinates, which makes it hard to replicate results
    - a hashtable is used to confirm every coordinate is unique
3. encrypt coordinates via AES-256-CBC with PBKDF2 hashed masterkey and store it into a log file
4. store following additional information in the log file:
  - id for password so you can navigate through multiple passwords
  - encrypted coordinates of the pixels, where bits of the hashed password are distributed to (with its string length)
  - maximum amount of digits that the largest number between the width and height of the png file has (so if 500 x 1920px = 4 digits)
  - salts and ivs used to randomize ciphertext

#### Procedure: bits extraction
Randsteg will:
1. authenticate user via PBKDF2 hashed masterkey
2. try to decrypt coordinates from log file 
3. extract LSB of pixels from given png according to decrypted coordinates
4. combine extracted bits and reproduce the encrypted password
5. decrypt password with PBKDF2 hashed masterkey
