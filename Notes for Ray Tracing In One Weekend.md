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

注意by convention, each of the red/green/blue components are represented internally by real-valued variables that range from 0.0 to 1.0. These must be scaled to integer values between 0 and 255 before we print them out 因为PPM 要求 RGB 是整数，所以要把 0.0~1.0 变成 0~255。

# 暂时乱序

乱序就是说，我就是说我，暂时不知道怎么归类，但总有一天会知道）

RGB是三维空间中的一个点。

##### Hue

Hue (色相): 表示颜色的种类，单位是°，取值范围通常是0°-360°，表示颜色在color wheel (色相圆)上的角度位置。

Hue是一种感知/模型量，中间是非线性映射。

此外，当saturation=0或value=0时，hue是没有意义的。前者是灰色，后者是黑色，都不算是颜色（准确来说是没有色相，**hue undefined**）。

*为什么灰色和黑色不算颜色？saturation表示离灰有多远是什么意思？lightness表示离黑白中点有多远又是什么意思呢？*

在颜色模型里，“颜色”特指“有色相的颜色”。灰色、白色、黑色没有色相，所以叫“无彩色”。这三种颜色的共同点是R=G=B：

黑：`(0, 0, 0)`

灰：`(128, 128, 128)`

白：`(255, 255, 255)`

意味着它们没有**偏向**任何一个方向。而Hue本质上是表示RGB向量偏向哪一个方向。

| 角度 | 颜色          |
| ---- | ------------- |
| 0°   | 红            |
| 60°  | 黄            |
| 120° | 绿            |
| 180° | 青            |
| 240° | 蓝            |
| 300° | 紫            |
| 360° | 红（等同 0°） |

##### Saturation

*saturation表示离灰有多远是什么意思*

想象一个三维坐标系，在这个空间里，所有 R=G=B 的点构成一条线，这条线叫**灰轴**。

有一个颜色点 `(R, G, B)`：

- 如果它 **正好在灰轴上**
  - R = G = B
  - Saturation = 0
- 如果它 **偏离灰轴**
  - Saturation > 0
- 偏得越远
  - 颜色越“纯”
  - Saturation 越高

Saturation 本质上是 颜色点到“灰轴”的距离。

##### Lightness

*lightness表示离黑白中点有多远又是什么意思呢？*

在 HSL 里：

- Lightness 范围是 0% ~ 100%
- 中点是 **50%**。这时候颜色最像正常的颜色。

对应的是：

```
黑 (0%)
↓
灰 (50%)
↓
白 (100%)
```

这是一条表示明暗的轴。在 RGB 空间里，Lightness ≈ `(max + min) / 2`。



##### Value

Value表示一个颜色中最亮的那一部分有多亮，通常表示为0-1或0%-100%。

在 HSV 模型中：
$$
V = \max(R, G, B)
$$

##### 饱和度

简单来说，描述的是一个颜色中“纯色成分”相对于“无色成分（白/灰）”的比例。

HSV视角下的定义：

$$
S_{H S V}= \begin{cases}0, & V=0 \\ 1-\frac{\min (R, G, B)}{V}, & V \neq 0\end{cases}
$$

其中：
－$V=\max (R, G, B)$
含义：
在当前亮度下，
灰色成分占了多少？

-  $\min =\max \rightarrow$ 灰色 $\rightarrow \mathrm{S}=0$
-  $\min =0 \rightarrow$ 纯色 $\rightarrow \mathrm{S}=1$



总结：

RGB 是基础数据，是颜色的物理表示方式；HSV 是解释视角。前者表示R/G/B三个光源的强度，后者中H表示偏向哪个RGB方向、S表示偏离灰轴的程度、V表示RGB中的最大值。

存储 / 渲染 / 设备：RGB

理解 / 调色 / 描述：HSV / HSL

暂时还没遇到到实践上的问题。只是今天想要精准地描述出我的睛蓝色袜子的颜色，本来想要用“饱和度比较高的蓝色”来描述，但想想又发现自己并不知道饱和度的准确定义，所以查了一下。







