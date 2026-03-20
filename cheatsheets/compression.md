# File Compression & Archiving

Tools for compressing and decompressing files on Linux systems.

## 1. Compression Tools
* `gzip -k <file>` - Compress file to `.gz` format. Use `-k` to keep original.
* `gzip -9 <file>` - Maximum compression level (1=fastest, 9=best compression).
* `xz -k <file>` - Compress file to `.xz` format. Better compression than gzip but slower.
* `xz -9 <file>` - Maximum xz compression level.
* `bzip2 -k <file>` - Compress file to `.bz2` format. May not be installed by default.

## 2. Decompression
* `gunzip <file.gz>` - Decompress `.gz` file.
* `xz -d <file.xz>` - Decompress `.xz` file.
* `bzip2 -d <file.bz2>` - Decompress `.bz2` file.

## 3. tar Archives
* `tar -cvf <archive.tar> <file>` - Create tar archive (no compression).
* `tar -czvf <archive.tar.gz> <file>` - Create tar archive with gzip compression.
* `tar -cJvf <archive.tar.xz> <file>` - Create tar archive with xz compression.
* `tar -xvf <archive.tar>` - Extract tar archive.
* `tar -xzvf <archive.tar.gz>` - Extract tar.gz archive.

## 4. Compression Tips
* Sorting file content before compression improves ratio - similar strings grouped together create more repeating patterns for the algorithm.
* Compression ratio comparison (best to worst): `xz` > `bzip2` > `gzip`
* Speed comparison (fastest to slowest): `gzip` > `bzip2` > `xz`