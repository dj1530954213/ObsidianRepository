### 1. 仿射变换 (Affine Transform)

**描述**：仿射变换能够在保持图像中线的平行性和直线性的前提下，对图像执行旋转、缩放、平移和倾斜等操作。仿射变换可以通过一个3×3 的矩阵来表示，但其最后一行固定为0,0,1。

**使用场景**：

- 图像校正：矫正图像中的扭曲或歪斜。
- 图像配准：将两幅图像中的相同特征对齐。
- 图像变换：改变图像的尺寸、位置或方向。

**对应的案例及相关算子使用方法**：
```csharp
*仿射变换的案例
dev_close_window()
read_image (BrakeDiskBike01, 'C:/Users/Public/Documents/MVTec/HALCON-24.05-Progress/examples/images/brake_disk/brake_disk_bike_01.png')
get_image_size (BrakeDiskBike01, Width, Height)
dev_open_window (0, 0, Width, Height, 'black', WindowHandle)
rgb1_to_gray(BrakeDiskBike01,GrayImage)
threshold (GrayImage, Region, 77, 255)
*获取图像面积并获取中心点坐标
area_center (Region, Area, Row, Column)
*定义仿射变化矩阵
hom_mat2d_identity (HomMat2DIdentity)
*设置平移矩阵至中心点坐标，获得了变换后的矩阵
hom_mat2d_translate (HomMat2DIdentity, Height/2-Row, Width/2-Column, HomMat2DTranslate)
*通过仿射变换将三角形移动至中心点位置并显示图像
affine_trans_image (GrayImage, ImageAffineTrans, HomMat2DTranslate, 'constant', 'true')
*按图形的中心进行旋转，获得旋转矩阵
hom_mat2d_rotate (HomMat2DIdentity, 3.14/2, Height/2, Width/2, HomMat2DRotate)
*通过仿射变换使用得到的旋转矩阵进行旋转
affine_trans_image (ImageAffineTrans, ImageAffineTrans1, HomMat2DRotate, 'constant', 'false')
*将图形进行缩放得到缩放矩阵
hom_mat2d_scale (HomMat2DIdentity, 0.5, 0.5, Height/2, Width/2, HomMat2DScale)
*使用仿射变换及缩放矩阵获得对应的图像
affine_trans_image (ImageAffineTrans1, ImageAffineTrans2, HomMat2DScale, 'constant', 'false')
dev_display (ImageAffineTrans2)
```
总结起来有三个关键点，第一个是单位矩阵，第二个是获得变换矩阵第三个是使用仿射变化和变换矩阵操作图像

**对应的矩阵**：
单位矩阵
![[1727794032971.png]]
平移矩阵(偏移参数在1-3和2-3)
![[1727794108572.png]]
旋转矩阵(旋转角度在1-1   1-2   2-1   2-2)
![[1727794180043.png]]
缩放矩阵(缩放值在1-1   1-2   2-1   2-2)
![[1727794297990.png]]

### 2. 投影变换 (Projective Transform or Homography)

**描述**：投影变换或称为单应性变换，是仿射变换的扩展，允许在变换过程中图像平面的“透视”效果也被模拟出来。投影变换需要一个3×3 的变换矩阵，可以处理因视角变化导致的图像畸变。

**使用场景**：

- 视角校正：例如，从高角度拍摄的图像恢复到正面视图。
- 3D效果和增强现实：在增强现实应用中匹配和定位图像与物理世界的相对位置。
- 图像融合和拼接：在全景图像创建中对不同角度拍摄的图像进行精确对齐和拼接。
```csharp
read_image (Caltab, 'C:/Users/Public/Documents/MVTec/HALCON-24.05-Progress/examples/images/caltab.png')
rgb1_to_gray (Caltab, GrayImage)
get_image_size (GrayImage, Width, Height)
dev_open_window (0, 0, Width, Height, 'black', WindowHandle)
dev_display(GrayImage)
draw_polygon (PolygonRegion, WindowHandle)
shape_trans (PolygonRegion, Filled, 'convex')
reduce_domain (GrayImage, Filled, ImageReduced)
*通过上面对图像进行基础处理完之后开始进行投影处理
dev_clear_window()
dev_display (ImageReduced)
dev_set_color ('red')
dev_set_line_width(2)
X:=[71,148,492,468]
Y:=[3,376,454,4]

Z:=[0,400,400,0]
W:=[0,0,400,400]
*标注小十字，XY是对应的点位 6表示小十字的长度   0.78表示旋转角度
gen_cross_contour_xld (Cross, X, Y, 6, 0.785398)
dev_display (Cross)
*计算投影变化矩阵，其中X,Y是变化前的四个点(最少四个)W,Z是变化后的落点
hom_vector_to_proj_hom_mat2d(X,Y,[1,1,1,1],W,Z,[1,1,1,1], 'normalized_dlt', HomMat2D)
*使用投影变化获取图像
projective_trans_image(ImageReduced,Image_rectified,HomMat2D,'bilinear', 'false', 'false')
dev_clear_window()
dev_display (Image_rectified)
```

### 3. 相似变换 (Similarity Transform)

**描述**：相似变换是仿射变换的一个子集，除了平移和旋转外，它只允许图像进行等比例缩放。这意味着图像的形状不会被歪曲，只是大小、位置和方向发生变化。

**使用场景**：

- 图像标准化：在进行图像比较或模板匹配之前，将不同图像调整到相同的尺度和方向。
- 特征匹配：在特征提取和匹配过程中，保持形状不变，只考虑位置、大小和旋转变化。
- 图像旋转和缩放：进行简单的图像旋转和缩放操作，常用于图像预处理。

这三种变换在HALCON中都可以通过相应的函数实现，如`affine_trans_image`, `projective_trans_image`, 和`similarity_trans_image`等。正确选择和使用这些变换可以有效地解决图像处理中的各种问题。