# Notes for Ray Tracing In One Weekend

## Image Format

图片的格式有很多种，其中许多都很复杂。让我们从纯文本的ppm格式开始^^.

### The PPM Image Format

PPM stands for Portable Pixmap Format，一种纯文本/二进制的RGB图片格式。特点是简单。可以理解成把每个像素的RGB数字按顺序写在一个文本文件里，没有压缩，也没有复杂头信息。一个示例：

```
P3
3 2
255
255 0 0     0 255 0     0 0 255
0 0 0       255 255 255 128 128 128
```



其中第一行的`P3`是magic number, 表示这是一个文本格式(ASCII)的PPM图像。如果是二进制格式，会是`P6`。

第二行表示宽度和高度。`3 2` 代表宽为3像素，高为2像素。PPM中的图像尺寸以像素计。

第三行表示RGB的取值范围是0~255。这也是最常见的范围（和主流图像格式一致，8-bit颜色对人眼已经足够自然），但是也可以是1，代表RGB只能是0或1，黑白图；也可以是31，比如某些GPU/老相机的5-bit颜色；也可以是65535，代表16-bit 的高精度颜色（0–65535）。总之这一行是最大颜色值，代表每个颜色通道的最大整数值。

接下来让我们练习用`C++`来画一张graphic "hello world":

![](https://raytracing.github.io/images/img-1.01-first-ppm-image.png)

```c++
#include <iostream>

int main() {

    // Image

    int image_width = 256;
    int image_height = 256;

    // Render

    std::cout << "P3\n" << image_width << ' ' << image_height << "\n255\n";

    for (int j = 0; j < image_height; j++) {
        for (int i = 0; i < image_width; i++) {
            auto r = double(i) / (image_width-1);
            auto g = double(j) / (image_height-1);
            auto b = 0.0;

            int ir = int(255.999 * r);
            int ig = int(255.999 * g);
            int ib = int(255.999 * b);

            std::cout << ir << ' ' << ig << ' ' << ib << '\n';
        }
    }
}
```

注意by convention, each of the red/green/blue components are represented internally by real-valued variables that range from 0.0 to 1.0. These must be scaled to integer values between 0 and 255 before we print them out 因为PPM 要求 RGB 是整数，所以要把 0.0~1.0 变成 0~255。.