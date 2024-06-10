# DebugTools

## Check the content of ptr

- 查看指针所指内容（Visual Studio）
  - 选择需要查看的指针，添加到快速监视；
  - `ptr,10    (查看指针所指内存前10个内容)`
- 查看指针所指内容（Visual Studio Code）
  - 选择需要查看的指针，添加到监视；
  - `ptr@10  (查看指针所指内存前10个内容)`

## ImageWatch

- @band(img, number): extract channel number  (UINT32) from img. Note: this operator preserves the input channel type.

- @thresh(img, threshold): threshold pixels in img: return 1 if >= threshold (FLOAT32) and 0 otherwise

- @clamp(img, min, max): clamp pixel values in img to lie between min (FLOAT32) and max (FLOAT32).

- @abs(img): take absolute value of pixels in img

- @scale(img, factor): scale pixel values in img by factor (FLOAT32)

- @norm8(img): scale pixels values in img by 1/255

- @norm16(img): scale pixels values in img by 1/65535

- @fliph(img), @flipv(img), @flipd(img): flip img horizontally, vertically, and diagonally (matrix transpose), respectively. Note: this operator preserves the input channel type.

- @rot90(img), @rot180(img), @rot270(img): rotate img by 90, 180, 270 degrees clockwise, respectively. Note: this operator preserves the input channel type.

- @diff(img0, img1): return pixel-wise difference: img0 – img1

- @file(path): load image from path (string). Example: @file(“d:\temp\debug.png”)

- @mem(address, type, channels, width, height, stride): interpret raw memory as pixels, starting at address (UINT64), with channel type (see Pixel Formats), number of channels (UINT32), width (UINT32), height (UINT32), and stride (UINT32). Example: @mem(myimg.data, UINT8, 1, 320, 240, 320)
