# eDefender
✨The paper has been accepted to TrustCom 2024✨

## Dataset Description
This dataset starts with **12,000** source files across five file types: **AVI, JPEG, MP3, PDF, and TXT**. For each file type, five transformations are applied: **original**, **compressed(_rar_)**, **encrypted(_enc_)**, **encrypted-then-compressed(_enc_rar_)**, and **compressed-then-encrypted(_rar_enc_)**. This process yields a total of **300,000** Byte Frequency Distribution (BFD) samples in the final dataset.<br />
<br />
### Source File Collection
For each file type, 12,000 source files were collected from various dataset sources:<br />
- **AVI**: UCF101 dataset (https://www.crcv.ucf.edu/data/UCF101.php)
- **JPG**: Open Images Dataset v5 (https://www.figure-eight.com/dataset/)
- **MP3**: FMA medium dataset (https://github.com/mdeff/fma)
- **PDF**: Papers from arXiv (https://arxiv.org/) and Govdocs1 (https://digitalcorpora.org/corpora/file-corpora/files/)
- **TXT**: Govdocs1 (https://digitalcorpora.org/corpora/file-corpora/files/)

**JPG, MP3, and some of PDF files were downloaded directly from [De Gaspari(2022)](https://link.springer.com/article/10.1007/s00521-022-07586-7)'s collected dataset (https://drive.google.com/file/d/1IDNv3U1hRILXblwT9fI3G-D8hJquiequ)**

### File Transformations
After the collection of the source files for each file type, follow these instructions to perform file transformations:
- **Original files**: source files
- **Compressed files**: [WinRAR](https://www.win-rar.com/create-rar-archive.html?&L=0) was utilized to perform file compressions. RAR was selected as the format of the new archive, and compression level, dictionary size, and other archiving parameters were set as default.
- **Encrypted files**: encrypting file types utilized the AES implementation provided by the [PyCryptodome Python library](https://pycryptodome.readthedocs.io/en/latest/src/cipher/aes.html).

```
import os
import sys
from hashlib import md5
from Crypto.Cipher import AES
from os import urandom

def derive_key_and_iv(password, salt, key_length, iv_length): #derive key and IV from password and salt.
    d = d_i = b''
    while len(d) < key_length + iv_length:
        d_i = md5(d_i + str.encode(password) + salt).digest() #obtain the md5 hash value
        d += d_i
    return d[:key_length], d[key_length:key_length+iv_length]

def encrypt(in_file, out_file, password, key_length=32):
    bs = AES.block_size #16 bytes
    salt = urandom(bs) #return a string of random bytes
    key, iv = derive_key_and_iv(password, salt, key_length, bs)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    out_file.write(salt)
    finished = False

    while not finished:
        chunk = in_file.read(1024 * bs) 
        if len(chunk) == 0 or len(chunk) % bs != 0:#final block/chunk is padded before encryption
            padding_length = (bs - len(chunk) % bs) or bs
            chunk += str.encode(padding_length * chr(padding_length))
            finished = True
        out_file.write(cipher.encrypt(chunk))

password = '(create your own password)' # At least 12 characters long

current_directory = os.getcwd()
print(current_directory)
folder_path = os.listdir("./MP3/mp3_original")
print(len(folder_path))

for file in folder_path:
    base = os.path.splitext(file)[0]
    orig_file = './MP3/mp3_original/'+ base + '.mp3' # Change this according to the input source file format
    enc_file = './MP3/mp3_enc/' + base + '.enc'
    with open(orig_file, 'rb') as in_file, open(enc_file, 'wb') as out_file:
        encrypt(in_file, out_file, password)
```
- **Encrypted-then-Compressed**: first encrypt the source file with procedures provided above then perform file compression with WinRAR
- **Compressed-then-Encrypted**: first perform file compression with WinRAR then encrypt the files with procedures provided above

### Byte Frequency Distributions(BFD) Generations
Our collected samples have a wide range of byte sizes:
- AVI: 36,710 bytes - 6,746,802 bytes
- JPG: 6,982 bytes - 1,203,783 bytes
- MP3: 1,061 bytes - 1,203,783 bytes
- PDF: 1,271 bytes - 67,135,442 bytes
- TXT: 23 bytes - 770,758,388 bytes
  
Each byte contain 8 bits of information. The BFDs, which count the occurrences of each byte value in a file, were extracted from the binary information of each file using MATLAB.
```
clearvars
close all
clc

projectdir = '(path_to_source_files)';
dinfo = dir(fullfile(projectdir));
dinfo([dinfo.isdir]) = [];
nfiles = length(dinfo);

for j = 1 : nfiles
  filename = fullfile(projectdir, dinfo(j).name);
  fid = fopen(filename, 'r');
  bytes = fread(fid);

  for i = 1 : 256
      bfd(j, i) = sum(bytes == i-1);
  end
  fclose(fid);
  
  if rem(j,10) == 0
      fprintf('number %d  \n', j)
  end
end
```
You may download the BFD dataset that we used in the paper within this [link](https://utsacloud-my.sharepoint.com/:u:/g/personal/wenjian_huang_my_utsa_edu/EfmTAPLyGfFPuGpy4Re8sf0BhbjbRbX3sPvDNkM4QC3O4Q?e=D5SfKA).

## Addressing Research Questions (RQs)

### _RQ1: Are there discernible patterns in the BFDs that can characterize the file types?_
Follow the below steps to replicate our results from Table I in the paper:

**1. Dataset Preparation:**
- Start with a dataset of 300,000 transformed files, each with 256-byte features.

**2. Exploratory Factor Analysis(EFA) in SPSS:**
- Perform factor analysis with Principal Component Analysis (PCA) and Varimax rotation in SPSS Statistics.
- Remove features with low factor loadings; retain only features with loadings ≥ 0.4 in the rotated component matrix.

**3. Select Features by Factor:**
- Identify the features corresponding to each factor (component) based on the rotated component matrix.

**4. Reliability Statistics:**
- Calculate Cronbach's Alpha to assess internal consistency using the selected features from each factor.
- Review Cronbach's Alpha results to confirm underlying consistency in byte distribution patterns across file types.

Useful Tutorials:
[SPSS Factor Analysis – Intermediate Tutorial](https://www.spss-tutorials.com/spss-factor-analysis-intermediate-tutorial/) , 
[Cronbach's Alpha (α) using SPSS Statistics](https://statistics.laerd.com/spss-tutorials/cronbachs-alpha-using-spss-statistics.php#interpreting)

## If you use our dataset or model in your paper, please cite:
```
@inproceedings{Hooker2024,
author       = {Hooker, A. and Huang, W. and Kurumathur, SK. and Vishwamitra, N. and Choo, KKR},
title        = {Towards Understanding and Detecting File Types in Encrypted Files for Law Enforcement Applications},
booktitle    = {23rd {IEEE} International Conference on Trust, Security and Privacy in Computing and Communications, TrustCom 2024},
publisher    = {{IEEE}},
year         = {2024}
}
```
