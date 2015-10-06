# kompressor

###### makes deceptively small images

What’s the biggest pixel size of a PNG image in the smallest number of bytes?

I wanted to try to create an image that could be downloaded but whose pixel buffer would be too big to store in the RAM of a PC. Here is a bzip2 file of 420 bytes that uncompresses to a PNG image of `6,132,534 bytes` (5.8 MB) and `225,000 × 225,000 pixels` (50.625 gigapixels), which, if represented as a pixel buffer of 3 bytes per pixel, takes about `141.4` GB.

PNG uses DEFLATE compression in a zlib wrapper. DEFLATE can asymptotically approach a compression ratio of 1032:1: each pair of bits can represent 258 zero bytes, and then there’s some constant overhead for headers and such.

The image is almost entirely zeroes, with a secret message in the center. We gain an extra factor of 8 pixels by using a 1-bit colorspace. Even with this maximum compression, the PNG file is basically a long string of zero bytes. bzip2 has a run-length preprocessing step that crunches these megabytes into a few hundred bytes.

Unfortunately the ratio of 1032 : 1 is really the best you can do. If you want an image with ten times the pixels, the PNG file has to be ten times the size. The above image is already close to the theoretical maximum (the extra divisor of 8 is because there are 8 pixels per uncompressed byte in the 1-bit colorspace):

```python
>>> 225000.*225000/8/6132534
1031.893993575902
```


There’s a constant cost of `109/1032/8 ≈ 121124` bytes per gigapixel. But if you are serving (or uploading) the image over HTTP, there’s another trick you can apply. Precompress the file and send it with Transfer-Coding: gzip to gain another factor of 1032 (gzip also uses DEFLATE) and a communication cost approaching ≈118 bytes per gigapixel. (This is like a two-layer zip bomb, except that both decompression steps are side effects of normal processing.)
