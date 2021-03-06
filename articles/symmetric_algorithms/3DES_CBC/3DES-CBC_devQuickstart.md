---
layout: quickstart
title: "Developer's QuickStart"
type: 3DES-CBC
image: /static_files/NewDevLogo.png
note: "Are you a developer? Get started with crucial implementation details above."
col: col-md-4 col-sm-4 col-xs-4 infoBlocks
alerts:
  - id: 1
    type: warning
    description: "Background Reading: Understanding Different Types of Problems in Crypto."
    link: "/flaw-categories.html"

further-reading:

related-articles:

attacks:
---
<p id="cbcintro">

  <h2> <img src="/static_files/configuration.jpg " style="width:110px;height:100px;" /> 3DES CBC Introduction </h2>

<strong>How it works:</strong><br />
Each block of plaintext is xor'ed with the previous block of ciphertext before being transformed, ensuring that identical plaintext blocks don't result in identical ciphertext blocks when in sequence. For the first block of plaintext (which doesn't have a preceding block) we use an initialization vector instead. This value should be unique per message per key, to ensure that identical messages don't result in identical ciphertexts.  <br /> <br />

<strong>Where is it commonly used:</strong><br />
CBC is used in many of the SSL/TLS cipher suites. <br /> <br />

<strong>Why CBC Mode?</strong><br />
CBC - Cipher Block Chaining. This mode is very common, and is considered to be reasonably secure.  <br /> <br />

<strong>How secure?</strong><br />
Unfortunately, there are attacks against CBC when it is not implemented alongside a set of strong integrity and authenticity checks. One property it has is block-level malleability, which means that an attacker can alter the plaintext of the message in a meaningful way without knowing the key, if he can mess with the ciphertext. As such, implementations usually include a HMAC-based authenticity record. But we are going to save that for a later post. <br /> <br />

<strong>Encryption Methods:</strong><br /><br />
INPUT: plaintext string, key, IV.<br />
<ul>
<li>convert string to bytes</li>
<li>pad</li>
<li>encrypt bytes</li>
</ul>
OUTPUT: ciphertext bytes.<br /><br />

<strong>Decryption Methods:</strong><br /><br />
INPUT: ciphertext bytes, key, IV.<br />
<ul>
<li>decrypt bytes</li>
<li>unpad (or return error if invalid)</li>
<li>convert bytes to string</li>
OUTPUT: plaintext string or "decryption error".<br /><br />

<span class="green">Good points:</span>  Secure when used properly, parallel decryption. <br />
<span class="red">Bad points:</span>  No parallel encryption, susceptible to malleability attacks when authenticity checks are bad / missing. But when done right, it's very good.

<p id="nocryptoroll">
  <div class="col-md-12 col-sm-12 col-xs-12">

        <h2> <img src="/static_files/implementation.png " style="width:100px;height:100px;" /> 3DES Implementation</h2>

<font size="4"><strong>Concept:</strong></font>  DO NOT roll your own Crypto! Use standard services and libraries. <br />

It is NOT advisable in any circumstances to develop any sort of cryptography on your own. Instead , there are a few options for standard libraries that can be used.
These libraries offer better stability as they are usually a product of several years of experience in implementing cryptography by an active development community who are
dedicated towards efforts in implementation. It is therefore considered to be reliable and robust. <br /> <br />

To use openssl:
Reference <a href="https://www.cryptosys.net/encrypt3des_ex.html">here</a>. <br /> <br />

<p id="usagelibrary">
<h2>Usage of Cryptography in Programming Languages</h2>

<font size="4"><strong>Concept:</strong></font> It is again advised to not roll out your own cryptography while developing software. There are popular libraries in almost all programming
languages that can readily be used to perform cryptographic operations.
<br /> <br />
<font size="4"><strong>Examples:</strong></font> <br />
<span class="green">3DES Encrypt Bytes and Hex in C:</span> <br />
This example uses Triple DES in CBC mode. The three examples show how to encrypt an ANSI string in Byte mode, in Hex mode, and how you could encrypt a string that consists of Unicode `wide` characters. <br /> <br />

Example:
<pre>
<code>
/* $Id: EncryptBytesAndHex.c $ */

/* Examples showing how to encrypt a arbitrary-length 'text' string
   with CryptoSys API using both 'Bytes' and `Hex' modes.
*/

/*
    Copyright (C) 2006 DI Management Services Pty Limited.
    All rights reserved. <www.di-mgt.com.au> <www.cryptosys.net>
    Last updated:
    $Date: 2006-08-16 06:06:00 $
    Use at your own risk.
*/

/*
    NOTES:
    We use standard ANSI C here. Upgrade/downgrade to suit your own tastes.
    Because we are dealing with data of unknown length, we allocate memory as we go.
    This tends to obscure the main cryptography a bit.
    We use assert statements to catch memory allocation problems and (shock! horror!)
    goto statements to handle errors and avoid memory leaks.
    You should make your own improvements on this if you use it in production code.

    ENCRYPTION SCHEME.
    INPUT:  SZ, KY, IV
    OUTPUT: CT
    E.1. PT <- TOBYTES(SZ)  // convert `text' to plaintext bytes
    E.2. IB <- PT || PAD        // pad the PT to make an input block so |IB| mod n = 0
    E.3. CT <- ENCRYPT(KY, IV, IB)  // Encrypt the input block

    DECRYPTION SCHEME.
    INPUT:  CT, KY, IV
    OUTPUT: SZ
    D.1. OB <- DECRYPT(K, IV, CT)   // decrypt the ciphertext to an output block
    D.2a. PT || PAD <- OB   // decompose the OB into plaintext and padding
    D.2b. If PAD is invalid, return "DECRYPTION ERROR"
    D.3. SZ <- TOSTRING(PT) // convert 'bytes' into 'text'

    NOTATION:
    KY: key
    IV: Initialization Vector
    SZ: a zero-terminated string containing text
    PT: plaintext bytes
    PAD: padding
    IB: input block to the encryption function
    CT: ciphertext bytes
    OB: output block from the decryption function
    || denotes concatenation
    |x| denotes the length of x

    OTHER POINTS TO NOTE:
    Note how we carefully distinguish between 'text' and 'bytes'.
    With ANSI characters in C you can get away with all sorts of fudging, but
    look what happens when we start using Unicode wide characters.
    'Text' is stored in a zero-terminated sequence of `char' or `wchar_t' types.
    A text string requires an extra character to be allocated for the zero terminating
    character and cannot contain a zero character inside it.
    'Bytes' are stored in an array of `unsigned char' types (often typedef'd as `BYTE').
    A byte array _can_ contain a zero byte inside it but always needs a separate variable
    to define its length.

    Because we are using CBC mode we have to pad the input before encryption. We use the
    PKCS7 method of padding and we _always_ add padding even if the original data is already
    a multiple of the block length. The padding can be used to detect a decryption error
    but this is not 100% guaranteed.

    This example uses a hard-coded key which should _never_ be done in practice.
    In practice, too, a fresh IV should be generated at random each time a message is encrypted.
    The IV needs to be transmitted to the receiver along with the ciphertext. This can be done
    in the clear.

*/

#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <assert.h>
#include "diCryptoSys.h"

/* Link to CryptoSys API library */
#pragma comment(lib, ".\\diCryptoSys.lib")

static void pr_bytesmsg(const char *msg, const unsigned char *bytes, int nbytes)
/* Print a message followed by hex-encoded byte array */
{
    int i;
    printf("%s", msg);
    for (i = 0; i < nbytes; i++)
        printf("%02X", bytes[i]);
    printf("\n");
}

void encrypt_with_bytes(void)
{
    /* Example encrypting plaintext string using Triple DES in CBC mode
       passing data in `Byte' format */
    char szPlain[] = "Hello world!";
    unsigned char key[] = {
        0xF0, 0xE1, 0xD2, 0xC3, 0xB4, 0xA5, 0x96, 0x87,
        0x78, 0x69, 0x5A, 0x4B, 0x3C, 0x2D, 0x1E, 0x0F,
        0x00, 0x11, 0x22, 0x33, 0x44, 0x55, 0x66, 0x77,
    };
    /* The IV should be generated at random each time */
    unsigned char iv[] = {
        0xFE, 0xDC, 0xBA, 0x98, 0x76, 0x54, 0x32, 0x10,
    };
    char *lpszCheck = NULL;
    unsigned char *lpPlain = NULL;
    unsigned char *lpBlock = NULL;
    unsigned char *lpCipher = NULL;
    long nPlain, nBlock, nCipher;
    long nRet;

    printf("USING `BYTE' PARAMETERS...\n");

    /* ENCRYPTION.
       INPUT:   szPlain, a zero-terminated ANSI string of any length;
                key of exactly 24 bytes (to be kept secret);
                iv of exactly 8 bytes.
       OUTPUT:  lpCipher, the ciphertext in an array of bytes;
                nCipher, the length of the ciphertext in bytes.
    */

    /* E.1 Convert the plaintext string into byte format
       PT <- TOBYTES(SZ) */
    /* (In C we can just use a typecasted pointer) */
    lpPlain = (unsigned char *)szPlain;

    /* Compute its length in bytes - assumed `narrow' ANSI chars */
    nPlain = strlen(szPlain);

    pr_bytesmsg("KY=", key, sizeof(key));
    pr_bytesmsg("IV=", iv, sizeof(iv));
    printf("SZ='%s'\n", szPlain);
    pr_bytesmsg("PT=", lpPlain, nPlain);

    /* How long is the padded block? */
    nBlock = PAD_BytesBlock(NULL, 0, lpPlain, nPlain, API_BLK_TDEA_BYTES, 0);
    if (nBlock <= 0) return;

    /* Allocate storage for padded plaintext block */
    lpBlock = malloc(nBlock);
    assert(lpBlock);

    /* E.2 Pad the plaintext to an exact multiple of the block size */
    /* IB <- PT || PAD */
    nBlock = PAD_BytesBlock(lpBlock, nBlock, lpPlain, nPlain, API_BLK_TDEA_BYTES, 0);
    if (nBlock <= 0) return;
    pr_bytesmsg("IB=", lpBlock, nBlock);

    /* Allocate storage for ciphertext output = same as input block */
    lpCipher = malloc(nBlock);
    assert(lpCipher);

    /* E.3 Encrypt the padded input block using Triple DES in CBC mode */
    /* CT <- ENCRYPT(K, IV, IB) */
    nRet = TDEA_BytesMode(lpCipher, lpBlock, nBlock, key, ENCRYPT, "CBC", iv);
    /* Deal with error (e.g. invalid key length) */
    if (nRet != 0) goto clean_up;
    nCipher = nBlock;

    /* Return the result, CT */
    pr_bytesmsg("CT=", lpCipher, nCipher);

    free(lpBlock);

    /* END OF ENCRYPTION ... NOW WE DECRYPT ... */

    /* DECRYPTION.
       INPUT:   lpCipher, the ciphertext in an array of bytes;
                nCipher, the length of the ciphertext in bytes;
                key; iv.
       OUTPUT:  szPlain, a zero-terminated string.
    */

    /* Allocate storage for padded decrypted plaintext block = same as ciphertext */
    nBlock = nCipher;
    lpBlock = malloc(nBlock);
    assert(lpBlock);

    /* D.1 Decrypt the ciphertext bytes
       OB <- DECRYPT(K, IV, CT) */
    nRet = TDEA_BytesMode(lpBlock, lpCipher, nCipher, key, DECRYPT, "CBC", iv);

    /* Deal with error */
    if (nRet != 0) goto clean_up;
    pr_bytesmsg("OB=", lpBlock, nBlock);

    /* D.2. Remove the padding bytes, or return an error if they are invalid
       PT || PAD <- OB */
    nPlain = PAD_UnpadBytes(lpBlock, nBlock, lpBlock, nBlock, API_BLK_TDEA_BYTES, 0);
    /* D.2b. If padding is invalid, return "DECRYPTION ERROR" */
    if (nPlain < 0)
    {
        printf("DECRYPTION ERROR\n");
        goto clean_up;
    }

    /* D.3. Convert the new plaintext bytes into a string
       SZ <- TOSTRING(PT) */

    /* You could just do
            lpBlock[nPlain] = '\0';
            lpszCheck = (char*)lpBlock;
       but we create a separate string to emphasise the difference
       between `bytes' and 'string' type.
    */
    lpszCheck = malloc(nPlain + 1);
    memcpy(lpszCheck, lpBlock, nPlain);
    lpszCheck[nPlain] = '\0';

    /* Display the results */
    pr_bytesmsg("PT=", lpBlock, nPlain);
    printf("P'='%s'\n", lpszCheck);

clean_up:
    /* Note that lpPlain is not allocated */
    if (lpBlock) free(lpBlock);
    if (lpCipher) free(lpCipher);
    if (lpszCheck) free(lpszCheck);
}

void encrypt_with_hex(void)
{
    /* Same example encrypting plaintext string using Triple DES in CBC mode
       using Hex encoded parameters */
    /*
       INPUT:   szPlain, a zero-terminated ANSI string of any length;
                szKeyHex, hex-encoded key representing exactly 24 bytes;
                szIvHex, hex-encoded IV representing exactly 8 bytes.
       OUTPUT:  lpszCipherHex, hex-encoded ciphertext string.
    */
    char szPlain[]  = "Hello world!";
    char szKeyHex[] = "F0E1D2C3B4A5968778695A4B3C2D1E0F0011223344556677";
    char szIvHex[]  = "FEDCBA9876543210";

    char *lpszPlainHex = NULL;
    char *lpszBlockHex = NULL;
    char *lpszCipherHex = NULL;
    char *lpszCheck = NULL;
    long ptchars, blkchars;
    long nRet;

    printf("USING `HEX' PARAMETERS...\n");

    /* E.1. Encode the plaintext string into hex format
       PT <- HEX(TOBYTES(SZ)) */
    ptchars = CNV_HexStrFromBytes(NULL, 0, (unsigned char*)szPlain, strlen(szPlain));
    if (ptchars <= 0) return;
    lpszPlainHex = malloc(ptchars + 1); /* NB extra one */
    ptchars = CNV_HexStrFromBytes(lpszPlainHex, ptchars, (unsigned char*)szPlain, strlen(szPlain));

    printf("KY=%s\n", szKeyHex);
    printf("IV=%s\n", szIvHex);
    printf("PT='%s'\n", szPlain);
    printf("PT=%s\n", lpszPlainHex);

    /* E.2. Pad the hex string directly, creating a new string
       IB <- PT || PAD */
    blkchars = PAD_HexBlock(NULL, 0, lpszPlainHex, API_BLK_TDEA_BYTES, 0);
    if (blkchars <= 0) return;
    lpszBlockHex = malloc(blkchars + 1);    /* NB extra one */
    assert(lpszBlockHex);
    blkchars = PAD_HexBlock(lpszBlockHex, blkchars, lpszPlainHex, API_BLK_TDEA_BYTES, 0);
    if (blkchars <= 0) goto clean_up;

    printf("IB=%s\n", lpszBlockHex);

    /* Allocate storage for output = same size as Input Block */
    lpszCipherHex = malloc(blkchars + 1);   /* NB extra one */
    if (NULL == lpszCipherHex) goto clean_up;

    /* E.3. Encrypt the plaintext data using Triple DES in CBC mode
       CT <- ENCRYPT(K, IV, IB) */
    nRet = TDEA_HexMode(lpszCipherHex, lpszBlockHex, szKeyHex, ENCRYPT, "CBC", szIvHex);
    /* Deal with error */
    if (nRet != 0) goto clean_up;

    /* Display the results */
    printf("CT=%s\n", lpszCipherHex);


    /* END OF ENCRYPTION ... NOW WE DECRYPT ... */

    /* DECRYPTION.
       INPUT:   ciphertext in a hex-encoded string;
                key as a hex-encoded string;
                iv used by sender as hex-encoded string.
       OUTPUT:  szPlain, a zero-terminated string or "DECRYPTION ERROR".
    */

    /* D.1 Decrypt the ciphertext:
       OB <- DECRYPT(K, IV, CT) */
    nRet = TDEA_HexMode(lpszBlockHex, lpszCipherHex, szKeyHex, DECRYPT, "CBC", szIvHex);
    /* Deal with error */
    if (nRet != 0) goto clean_up;
    printf("OB=%s\n", lpszBlockHex);

    /* D.2. Unpad the output block or return error if padding is invalid
       PT || PAD <- OB
       Note that the final output will ALWAYS be shorter than the block
       so we can use the same memory */
    nRet = PAD_UnpadHex(lpszBlockHex, blkchars, lpszBlockHex, API_BLK_TDEA_BYTES, 0);
    if (nRet < 0)
    {
        printf("DECRYPTION ERROR\n");
        goto clean_up;
    }

    /* D.3. Decode the unpadded plaintext hex into a zero-terminated string
       SZ <- TOSTRING(PT) */
    ptchars = CNV_BytesFromHexStr(NULL, 0, lpszBlockHex);
    lpszCheck = malloc(ptchars + 1);    /* NB one extra */
    assert(NULL != lpszCheck);
    ptchars = CNV_BytesFromHexStr(lpszCheck, ptchars, lpszBlockHex);
    lpszCheck[ptchars] = '\0';  /* Don't forget this */

    /* Display the results */
    printf("P'=%s\n", lpszBlockHex);
    printf("P'='%s'\n", lpszCheck);

clean_up:
    if (lpszPlainHex) free(lpszPlainHex);
    if (lpszBlockHex) free(lpszBlockHex);
    if (lpszCipherHex) free(lpszCipherHex);
}

void encrypt_wide_chars(void)
{
    /* To add some spice, let's do the `Bytes' method again
       but this time with the input as Unicode wide chars */
    wchar_t szPlain[] = L"Hello world!";
    unsigned char key[] = {
        0xF0, 0xE1, 0xD2, 0xC3, 0xB4, 0xA5, 0x96, 0x87,
        0x78, 0x69, 0x5A, 0x4B, 0x3C, 0x2D, 0x1E, 0x0F,
        0x00, 0x11, 0x22, 0x33, 0x44, 0x55, 0x66, 0x77,
    };
    unsigned char iv[] = {
        0xFE, 0xDC, 0xBA, 0x98, 0x76, 0x54, 0x32, 0x10,
    };
    wchar_t *lpszCheck = NULL;
    unsigned char *lpPlain = NULL;
    unsigned char *lpBlock = NULL;
    unsigned char *lpCipher = NULL;
    long nPlain, nBlock, nCipher, nwchars;
    long nRet;

    printf("WITH WIDE CHARACTERS...\n");

    /* E.1 Convert the plaintext string into byte format
       PT <- TOBYTES(SZ) */
    /* This time, with wide chars, we can still use a cast for the byte array ptr
       but we compute the length differently */
    lpPlain = (unsigned char *)szPlain;
    nwchars = wcslen(szPlain);
    nPlain = nwchars * sizeof(wchar_t);

    pr_bytesmsg("KY=", key, sizeof(key));
    pr_bytesmsg("IV=", iv, sizeof(iv));
    wprintf(L"SZ='%s'\n", szPlain);
    pr_bytesmsg("PT=", lpPlain, nPlain);

    /* How long is the padded block? */
    nBlock = PAD_BytesBlock(NULL, 0, lpPlain, nPlain, API_BLK_TDEA_BYTES, 0);
    if (nBlock <= 0) return;

    /* Allocate storage for padded plaintext block */
    lpBlock = malloc(nBlock);
    assert(lpBlock);

    /* E.2 Pad the plaintext to an exact multiple of the block size */
    /* IB <- PT || PAD */
    nBlock = PAD_BytesBlock(lpBlock, nBlock, lpPlain, nPlain, API_BLK_TDEA_BYTES, 0);
    if (nBlock <= 0) return;
    pr_bytesmsg("IB=", lpBlock, nBlock);

    /* Allocate storage for ciphertext output = same as input block */
    lpCipher = malloc(nBlock);
    assert(lpCipher);

    /* E.3 Encrypt the padded input block using Triple DES in CBC mode */
    /* CT <- ENCRYPT(K, IV, IB) */
    nRet = TDEA_BytesMode(lpCipher, lpBlock, nBlock, key, ENCRYPT, "CBC", iv);
    /* Deal with error (e.g. invalid key length) */
    if (nRet != 0) goto clean_up;
    nCipher = nBlock;

    /* Return the result, CT */
    pr_bytesmsg("CT=", lpCipher, nCipher);

    free(lpBlock);

    /* END OF ENCRYPTION ... NOW WE DECRYPT ... */

    /* Allocate storage for padded decrypted plaintext block = same as ciphertext */
    nBlock = nCipher;
    lpBlock = malloc(nBlock);
    assert(lpBlock);

    /* D.1 Decrypt the ciphertext bytes
       OB <- DECRYPT(K, IV, CT) */
    nRet = TDEA_BytesMode(lpBlock, lpCipher, nCipher, key, DECRYPT, "CBC", iv);

    /* Deal with error */
    if (nRet != 0) goto clean_up;
    pr_bytesmsg("OB=", lpBlock, nBlock);

    /* D.2. Remove the padding bytes, or return an error if they are invalid
       PT || PAD <- OB */
    nPlain = PAD_UnpadBytes(lpBlock, nBlock, lpBlock, nBlock, API_BLK_TDEA_BYTES, 0);
    /* D.2b. If padding is invalid, return "DECRYPTION ERROR" */
    if (nPlain < 0)
    {
        printf("DECRYPTION ERROR\n");
        goto clean_up;
    }

    /* D.3. Convert the new plaintext bytes into a string
       SZ <- TOSTRING(PT) */
    /* Be careful with wide characters... */
    nwchars = nPlain / sizeof(wchar_t);
    lpszCheck = malloc((nwchars + 1) * sizeof(wchar_t));
    memcpy(lpszCheck, lpBlock, nPlain);
    lpszCheck[nwchars] = L'\0';

    /* Display the results */
    pr_bytesmsg("PT=", lpBlock, nPlain);
    wprintf(L"SZ='%s'\n", lpszCheck);

clean_up:
    /* Note that lpPlain is not allocated */
    if (lpBlock) free(lpBlock);
    if (lpCipher) free(lpCipher);
    if (lpszCheck) free(lpszCheck);
}


int main()
{
    encrypt_with_bytes();
    encrypt_with_hex();
    encrypt_wide_chars();
    return 0;
}
