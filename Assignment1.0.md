## TODO 

- MLAA
- jiltered sampling

## My reflection

画三角形的另一种实现
```C++
static bool point_in_triangle(float px, float py, float x0, float y0, float x1,
                              float y1, float x2, float y2) {
  const Vector2D AB{x1 - x0, y1 - y0}, BC{x2 - x1, y2 - y1},
      CA{x0 - x2, y0 - y2};
  const Vector2D AP{px - x0, py - y0}, BP{px - x1, py - y1},
      CP{px - x2, py - y2};
  const float A{(float)cross(AB, AP)}, B{(float)cross(BC, BP)},
      C{(float)cross(CA, CP)};
  const bool sa = A >= 0, sb = B >= 0, sc = C >= 0;
  return sa == sb && sb == sc;
}

static void sort_point(float &x0, float &y0, float &x1, float &y1, float &x2,
                       float &y2) {
  if (y1 > y0) {
    std::swap(y1, y0);
    std::swap(x1, x0);
  }
  if (y2 > y0) {
    std::swap(y2, y0);
    std::swap(x2, x0);
  }
  if (y2 > y1) {
    std::swap(y2, y1);
    std::swap(x2, x1);
  }
}

static float locate_xright(float righty, float x1, float y1, float x2,
                           float y2) {
  // x = my + n assert ：不存在斜率为0的情况
  const float m = (x2 - x1) / (y2 - y1);
  const float n = x1 - m * y1;
  return m * righty + n;
}

void SoftwareRendererImp::rasterize_triangle_A(float tx, float ty, float bx1,
                                               float bx2, float by,
                                               Color color) {
  if (bx1 > bx2)
    std::swap(bx1, bx2);
  const auto VST = [this, tx, ty, bx2, by, &color](float x, float y) {
    const float x_right = locate_xright(y, tx, ty, bx2, by);
    if (!(std::abs(y - ty) < EPS))
      rasterize_line(std::floor(x + 0.5f) + 0.5f, std::floor(y + 0.5f) + 0.5f,
                     std::floor(x_right + 0.5f) + 0.5,
                     std::floor(y + 0.5f) + 0.5f, color);
  };
  goAlongline(tx, ty, bx1, by, VST);
}

void SoftwareRendererImp::rasterize_triangle_V(float bx, float by, float tx1,
                                               float tx2, float ty,
                                               Color color) {
  if (tx1 > tx2)
    std::swap(tx1, tx2);
  const auto VST = [this, bx, by, tx2, ty, &color](float x, float y) {
    const float x_right = locate_xright(y, bx, by, tx2, ty);
    if (!(std::abs(y - by) < EPS))
      rasterize_line(std::floor(x + 0.5f) + 0.5f, std::floor(y + 0.5f) + 0.5f,
                     std::floor(x_right + 0.5f) + 0.5f,
                     std::floor(y + 0.5f) + 0.5f, color);
  };
  goAlongline(tx1, ty, bx, by, VST);
}
void SoftwareRendererImp::rasterize_triangle(float x0, float y0, float x1,
                                             float y1, float x2, float y2,
                                             Color color) {
  // Task 3:
  // Implement triangle rasterization
  // 边界情况，不构成三角形
  // printf("enter triangle \n");
  if (!is_triangle(x0, y0, x1, y1, x2, y2))
    return;

// 这种方法看似快，实则处理比较极端的三角形有缺陷，因为右边界的确定往往不是那么完美
  // 约定最高点y0， 最低点y2，按照y0, y1, y2进行排序
  // printf("before sort : %3f %3f %3f %3f %3f %3f\n", x0, y0, x1, y1, x2, y2);
  sort_point(x0, y0, x1, y1, x2, y2);

  // printf("after sort : %3f %3f %3f %3f %3f %3f\n", x0, y0, x1, y1, x2, y2);
  //  分割为两个A/V型的三角形
  if (std::abs(y1 - y2) < EPS) { // A
    rasterize_triangle_A(x0, y0, x1, x2, y1, color);
  } else if (std::abs(y1 - y0) < EPS) { // V
    rasterize_triangle_V(x2, y2, x0, x1, y0, color);
  } else { // A+V
    // 不存在(x0,y0)-(x2,y2)斜率为0的情况， 设为 x = m * y + n
    const float m = (x0 - x2) / (y0 - y2);
    const float n = x0 - m * y0;
    // 已经知道(x1, y1) , 知道y1坐标，求另一侧x
    const float sx = m * y1 + n;
    // printf("A : %3f %3f %3f %3f %3f \n", x0, y0, x1, sx, y1);
    //  三角形A
    rasterize_triangle_A(x0, y0, x1, sx, y1, color);
    // printf("V : %3f %3f %3f %3f %3f \n", x2, y2, x1, sx , y1);
    //  三角形V
    rasterize_triangle_V(x2, y2, x1, sx, y1, color);
  }
}
```
