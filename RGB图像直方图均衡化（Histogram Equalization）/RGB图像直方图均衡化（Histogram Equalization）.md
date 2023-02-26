---
title: RGB图像直方图均衡化（Histogram Equalization）
author: Portgas·D·Asce
categories:
  - [Image Processing]
tags:
  - Image Processing
date: 2021-02-20 17:48:26
---
原图：图像偏暗
{% asset_img 1.png %}

## 1 直方图均衡化方法
方法：
- $RGB$ 均衡化（一）：分别对 $R$, $G$, $B$ 三波段进行均衡化，然后将三个波段的结果进行合并；
- $RGB$ 均衡化（二）：直接对 $R$, $G$, $B$ 三波段一块进行均衡化；
- $HSV$ 均衡化：先将 $RGB$ 转为 $HSV$, 然后对 $V$ 波段进行均衡化，最后再将 $HSV$ 转回 $RGB$；
- $HSI$ 均衡化：先将 $RGB$ 转为 $HSI$, 然后对 $I$ 波段进行均衡化，最后再将 $HSI$ 转回 $RGB$；

## 2 $RGB$ 均衡化（一）
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <png.h>
#include <zlib.h>

void histogram_equalization_rgb1(png_bytep buffer, int w, int h, int c, int (*st)[256]) {
    for(int i = 0; i < h; ++i) {
        int base = i * w * c;
        for(int j = 0; j < w; ++j) {
            int cur = base + j * c;
            for(int k = 0; k < c - 1; ++k) {
                ++st[k][buffer[cur + k]];
            }
        }
    }

    for(int i = 0; i < c - 1; ++i) {
        for(int j = 1; j < 256; ++j) {
            st[i][j] += st[i][j - 1];
        }
    }

    double temp = 255.0 / (w * h);
    for(int i = 0; i < h; ++i) {
        int base = i * w * c;
        for(int j = 0; j < w; ++j) {
            int cur = base + j * c;
            for(int k = 0; k < c - 1; ++k) {
                buffer[cur + k] = st[k][buffer[cur + k]] * temp;
            }
        }
    }
}

int main() {
    char path_src[512] = "./data/src.png";
    char path_dst[512] = "./res/rgb1.png";

    png_image image;
    memset(&image, 0, sizeof(image));
    image.version = PNG_IMAGE_VERSION;

    if(png_image_begin_read_from_file(&image, path_src) == 0) {
        fprintf(stderr, "error: %s\n", image.message);
        return 0;
    }

    int w = image.width, h = image.height;
    //波段数
    int channels = PNG_IMAGE_PIXEL_CHANNELS(image.format);
    //像素深度
    //int depth = PNG_IMAGE_PIXEL_COMPONENT_SIZE((image).format);

    //申请内存
    png_bytep buffer = malloc(PNG_IMAGE_SIZE(image));
    if(buffer == NULL) {
        png_image_free(&image);
        return 0;
    }

    //缓冲到内存
    if(png_image_finish_read(&image, NULL, buffer, 0, NULL) == 0) {
        fprintf(stderr, "error: %s\n", image.message);
    } else {
        int st[3][256] = {0};
        histogram_equalization_rgb1(buffer, w, h, channels, st);
        if(png_image_write_to_file(&image, path_dst, 0, buffer, 0, NULL) == 0) {
            fprintf(stderr, "error: %s\n", image.message);
        } else {
            printf("histogram equalization successfully\n");
        }
    }

    free(buffer);
    png_image_free(&image);

    return 0;
}
```

效果：
{% asset_img 2.png %}

## 3 $RGB$ 均衡化（二）
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <png.h>
#include <zlib.h>

void histogram_equalization_rgb2(png_bytep buffer, int w, int h, int c, int *st) {
    for(int i = 0; i < h; ++i) {
        int base = i * w * c;
        for(int j = 0; j < w; ++j) {
            int cur = base + j * c;
            for(int k = 0; k < c - 1; ++k) {
                ++st[buffer[cur + k]];
            }
        }
    }

    for(int i = 1; i < 256; ++i) {
        st[i] += st[i - 1];
    }

    double temp = 255.0 / (w * h * 3);
    for(int i = 0; i < h; ++i) {
        int base = i * w * c;
        for(int j = 0; j < w; ++j) {
            int cur = base + j * c;
            for(int k = 0; k < c - 1; ++k) {
                buffer[cur + k] = st[buffer[cur + k]] * temp;
            }
        }
    }
}

int main() {
    char path_src[512] = "./data/src.png";
    char path_dst[512] = "./res/rgb2.png";

    png_image image;
    memset(&image, 0, sizeof(image));
    image.version = PNG_IMAGE_VERSION;

    if(png_image_begin_read_from_file(&image, path_src) == 0) {
        fprintf(stderr, "error: %s\n", image.message);
        return 0;
    }

    int w = image.width, h = image.height;
    //波段数
    int channels = PNG_IMAGE_PIXEL_CHANNELS(image.format);
    //像素深度
    //int depth = PNG_IMAGE_PIXEL_COMPONENT_SIZE((image).format);

    //申请内存
    png_bytep buffer = malloc(PNG_IMAGE_SIZE(image));
    if(buffer == NULL) {
        png_image_free(&image);
        return 0;
    }

    //缓冲到内存
    if(png_image_finish_read(&image, NULL, buffer, 0, NULL) == 0) {
        fprintf(stderr, "error: %s\n", image.message);
    } else {
        int st[256] = {0};
        histogram_equalization_rgb2(buffer, w, h, channels, st);
        if(png_image_write_to_file(&image, path_dst, 0, buffer, 0, NULL) == 0) {
            fprintf(stderr, "error: %s\n", image.message);
        } else {
            printf("histogram equalization successfully\n");
        }
    }

    free(buffer);
    png_image_free(&image);

    return 0;
}
```
效果：
{% asset_img 3.png %}

## 4 $HSV$ 均衡化
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>

#include <png.h>
#include <zlib.h>

#define min(a, b, c) ((a) < (b) ? ((a) < (c) ? (a) : (c)) : ((b) < (c) ? (b) : (c)))
#define max(a, b, c) ((a) > (b) ? ((a) > (c) ? (a) : (c)) : ((b) > (c) ? (b) : (c)))

#define epsl 0.0000001

void rgb_to_hsv(png_bytep buffer, int w, int h, int c, double *hsv) {
    for(int i = 0; i < h; ++i) {
        int base = i * w * c;
        for(int j = 0; j < w; ++j) {
            int cur = base + j * c;
            double rr = buffer[cur + 0] / 255.0;
            double gg = buffer[cur + 1] / 255.0;
            double bb = buffer[cur + 2] / 255.0;
            double mn = min(rr, gg, bb);
            double mx = max(rr, gg, bb);
            double delta = mx - mn;

            if(fabs(delta) < epsl) {
                hsv[cur + 0] = 0.0;
            } else if(fabs(mx - rr) < epsl) {
                double temp = (gg - bb) / delta;
                if(temp < 0) temp += 6.0;
                hsv[cur + 0] = 60.0 * temp;
            } else if(fabs(mx - gg) < epsl) {
                hsv[cur + 0] = 60.0 * ((bb - rr) / delta + 2);
            } else {
                hsv[cur + 0] = 60.0 * ((rr - gg) / delta + 4);
            }

            if(fabs(mx) < epsl) {
                hsv[cur + 1] = 0.0;
            } else {
                hsv[cur + 1] = delta / mx;
            }

            hsv[cur + 2] = mx;
        }
    }
}

void histogram_equalization_hsv(double *hsv, int w, int h, int c, int st[]) {
    for(int i = 0; i < h; ++i) {
        int base = i * w * c;
        for(int j = 0; j < w; ++j) {
            int cur = base + j * c;
            ++st[(int)(hsv[cur + 2] * 255)];
        }
    }
    for(int i = 1; i < 256; ++i) {
        st[i] += st[i - 1];
    }
    double temp = 255.0 / (w * h);
    for(int i = 0; i < h; ++i) {
        int base = i * w * c;
        for(int j = 0; j < w; ++j) {
            int cur = base + j * c;
            hsv[cur + 2] = st[(int)(hsv[cur + 2] * 255)] * temp / 255;
        }
    }
}

void hsv_to_rgb(png_bytep buffer, int w, int h, int c, double *hsv) {
    for(int i = 0; i < h; ++i) {
        int base = i * w * c;
        for(int j = 0; j < w; ++j) {
            int cur = base + j * c;
            double hh = hsv[cur + 0];
            double ss = hsv[cur + 1];
            double vv = hsv[cur + 2];

            double cc = vv * ss;
            double temp = hh / 60.0;
            double xx = cc * (1 - fabs(temp - (int)(temp / 2.0) * 2.0 - 1));
            double mm = vv - cc;
            double rr = 0.0, gg = 0.0, bb = 0.0;
            if(hh > 0 && hh < 60) {
                rr = cc;
                gg = xx;
                bb = 0.0;
            } else if(hh < 120) {
                rr = xx;
                gg = cc;
                bb = 0.0;
            } else if(hh < 180) {
                rr = 0.0;
                gg = cc;
                bb = xx;
            } else if(hh < 240) {
                rr = 0.0;
                gg = xx;
                bb = cc;
            } else if(hh < 300) {
                rr = xx;
                gg = 0.0;
                bb = cc;
            } else if(hh < 360) {
                rr = cc;
                gg = 0.0;
                bb = xx;
            }

            buffer[cur + 0] = (rr + mm) * 255;
            buffer[cur + 1] = (gg + mm) * 255;
            buffer[cur + 2] = (bb + mm) * 255;
        }
    }
}

int main() {
    char path_src[512] = "./data/src.png";
    char path_dst[512] = "./res/hsv.png";

    png_image image;
    memset(&image, 0, sizeof(image));
    image.version = PNG_IMAGE_VERSION;

    if(png_image_begin_read_from_file(&image, path_src) == 0) {
        fprintf(stderr, "error: %s\n", image.message);
        return 0;
    }

    int w = image.width, h = image.height;
    //波段数
    int channels = PNG_IMAGE_PIXEL_CHANNELS(image.format);
    //像素深度
    //int depth = PNG_IMAGE_PIXEL_COMPONENT_SIZE((image).format);

    //申请内存
    png_bytep buffer = malloc(PNG_IMAGE_SIZE(image));
    if(buffer == NULL) {
        png_image_free(&image);
        return 0;
    }

    //缓冲到内存
    if(png_image_finish_read(&image, NULL, buffer, 0, NULL) == 0) {
        fprintf(stderr, "error: %s\n", image.message);
    } else {
        int st[256] = {0};
        double *hsv = (double *)malloc(w * h * channels * sizeof(double));
        rgb_to_hsv(buffer, w, h, channels, hsv);
        histogram_equalization_hsv(hsv, w, h, channels, st);
        hsv_to_rgb(buffer, w, h, channels, hsv);
        free(hsv);

        if(png_image_write_to_file(&image, path_dst, 0, buffer, 0, NULL) == 0) {
            fprintf(stderr, "error: %s\n", image.message);
        } else {
            printf("histogram equalization successfully\n");
        }
    }

    free(buffer);
    png_image_free(&image);

    return 0;
}
```
效果：
{% asset_img 4.png %}

## 5 $HSI$ 均衡化
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>

#include <png.h>
#include <zlib.h>

#define min(a, b, c) ((a) < (b) ? ((a) < (c) ? (a) : (c)) : ((b) < (c) ? (b) : (c)))
#define max(a, b, c) ((a) > (b) ? ((a) > (c) ? (a) : (c)) : ((b) > (c) ? (b) : (c)))

#define epsl 0.0000001

void rgb_to_hsi(png_bytep buffer, int w, int h, int c, double *hsi) {
    const double pi = acos(-1);
    for(int i = 0; i < h; ++i) {
        int base = i * w * c;
        for(int j = 0; j < w; ++j) {
            int cur = base + j * c;
            int r = buffer[cur + 0];
            int g = buffer[cur + 1];
            int b = buffer[cur + 2];
            
            if(r == g && g == b) {
                hsi[cur + 0] = 0;
            } else {
                int x = (r - g + r - b);
                double y = 2.0 * sqrt((r - g) * (r - g) + (r - b) * (g - b));
                double z = x / y;
                
                hsi[cur + 0] = acos(x / y);
                if(g < b) {
                    hsi[cur + 0] = 2 * pi - hsi[cur + 0];
                }
            }
            
            hsi[cur + 1] = 1.0 - 3.0 * min(r, g, b) / (r + g + b);
            hsi[cur + 2] = (r + g + b) / 3.0;
            //printf("%lf %lf %lf\n", hsi[cur + 0] / pi * 180, hsi[cur + 1], hsi[cur + 2]);
        }
    }
}

void histogram_equalization_hsi(double *hsi, int w, int h, int c, int st[]) {
    for(int i = 0; i < h; ++i) {
        int base = i * w * c;
        for(int j = 0; j < w; ++j) {
            int cur = base + j * c;
            ++st[(int)hsi[cur + 2]];
        }
    }
    for(int i = 1; i < 256; ++i) {
        st[i] += st[i - 1];
    }
    double temp = 255.0 / (w * h);
    for(int i = 0; i < h; ++i) {
        int base = i * w * c;
        for(int j = 0; j < w; ++j) {
            int cur = base + j * c;
            hsi[cur + 2] = st[(int)hsi[cur + 2]] * temp;
        }
    }
}

void hsi_to_rgb(png_bytep buffer, int w, int h, int c, double *hsi) {
    const double pi = acos(-1);
    for(int i = 0; i < h; ++i) {
        int base = i * w * c;
        for(int j = 0; j < w; ++j) {
            int cur = base + j * c;
            double hh = hsi[cur + 0];
            double ss = hsi[cur + 1];
            double ii = hsi[cur + 2];
            double rr = 1 / 3.0, gg = 1 / 3.0, bb = 1 / 3.0;
            if(fabs(ss) > epsl) {
                if(hh > 0 && hh < 2 * pi / 3) {
                    bb = (1.0 - ss) / 3.0;
                    rr = (1 + ss * cos(hh) / cos(pi / 3 - hh)) / 3.0;
                    gg = 1 - rr - bb;
                } else if(hh < 4 * pi / 3) {
                    hh = hh - 2 * pi / 3;
                    rr = (1.0 - ss) / 3.0;
                    gg = (1 + ss * cos(hh) / cos(pi / 3 - hh)) / 3.0;
                    bb = 1 - rr - gg;
                } else if(hh  < 2 * pi) {
                    hh = hh - 4 * pi / 3;
                    gg = (1.0 - ss) / 3.0;
                    bb = (1 + ss * cos(hh) / cos(pi / 3 - hh)) / 3.0;
                    rr = 1 - bb - gg;
                } else {
                    printf("\n H out of range: %lf.\n", hh);
                }
            }
            if(rr < 0 || gg < 0 || bb < 0 || rr > 1 || gg > 1 || bb > 1) {
                printf("\n RGB out of range: R = %lf G = %lf B = %lf H = %lf S = %lf I = %lf\n", rr, gg, bb, hh, ss, ii);
            }
            
            buffer[cur + 0] = 3 * ii * rr;
            buffer[cur + 1] = 3 * ii * gg;
            buffer[cur + 2] = 3 * ii * bb;
        }
    }
}

int main() {
    char path_src[512] = "./data/src.png";
    char path_dst[512] = "./res/hsi.png";

    png_image image;
    memset(&image, 0, sizeof(image));
    image.version = PNG_IMAGE_VERSION;

    if(png_image_begin_read_from_file(&image, path_src) == 0) {
        fprintf(stderr, "error: %s\n", image.message);
        return 0;
    }

    int w = image.width, h = image.height;
    //波段数
    int channels = PNG_IMAGE_PIXEL_CHANNELS(image.format);
    //像素深度
    //int depth = PNG_IMAGE_PIXEL_COMPONENT_SIZE((image).format);

    //申请内存
    png_bytep buffer = malloc(PNG_IMAGE_SIZE(image));
    if(buffer == NULL) {
        png_image_free(&image);
        return 0;
    }

    //缓冲到内存
    if(png_image_finish_read(&image, NULL, buffer, 0, NULL) == 0) {
        fprintf(stderr, "error: %s\n", image.message);
    } else {
        int st[256] = {0};
        double *hsi = (double *)malloc(w * h * channels * sizeof(double));
        rgb_to_hsi(buffer, w, h, channels, hsi);
        histogram_equalization_hsi(hsi, w, h, channels, st);
        hsi_to_rgb(buffer, w, h, channels, hsi);
        free(hsi);

        if(png_image_write_to_file(&image, path_dst, 0, buffer, 0, NULL) == 0) {
            fprintf(stderr, "error: %s\n", image.message);
        } else {
            printf("histogram equalization successfully\n");
        }
    }

    free(buffer);
    png_image_free(&image);

    return 0;
}
```

效果：
{% asset_img 5.png %}

## 结论

$RGB$ 均衡化（二） 和 $HSV$ 均衡化 效果较好。
