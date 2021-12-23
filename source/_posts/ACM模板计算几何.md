---
title: ACM模板计算几何
date: 2021/07/07 15:46:35
categories: 
- 模板
tags: 
- 模板
- 计算几何
---

<!--more-->

## 点

```cpp
const double eps = 1e-8;
const double PI = acos(-1.0);

int sgn(double x) {  //判断一个数的正负
  if (fabs(x) < eps) return 0;  //x = 0
  if (x < 0) return -1;  //x < 0
  return 1;  //x > 0
}

//弧度转角度
double R_to_D(double rad) { return 180 / PI * rad; }
//角度转弧度
double D_to_R(double D) { return PI / 180 * D; }

struct Point {  //点
  double x, y;
  Point(double _x = 0, double _y = 0): x(_x), y(_y) {}
};
typedef Point Vector;  //向量

Vector operator + (Vector A, Vector B) { return Vector(A.x - B.x, A.y - B.y); }
Vector operator - (Point A, Point B) { return Vector(A.x - B.x, A.y - B.y); }
Vector operator * (Vector A, double p) { return Vector(A.x * p, A.y * p); }
Vector operator / (Vector A, double p) { return Vector(A.x / p, A.y / p); }
bool operator < (Point A, Point B) { return A.x < B.x || (A.x == B.x && A.y < B.y); }

//向量点积(对加法满足交换律)
double Dot(Vector A, Vector B) { return A.x * B.x + A.y * B.y; }
//向量叉积(不满足交换律)
//等于两向量有向面积的二倍(从v的方向看, w在左边, 叉积 > 0, w在右边, 叉积 < 0, 共线, 叉积 = 0)
//Cross(x, y) = -Cross(y, x)
double Cross(Vector A, Vector B) { return A.x * B.y - B.x * A.y; }
//求极角,在极坐标系中,平面上任何一点到极点的连线和极轴的夹角叫做极角。单位弧度制rad
double Polar_angle(Vector A) { return atan2(A.y, A.x); }
//判断向量BC是否在AB的逆时针方向,可以看作点C是否在向量AB左边
bool ToLeftTest(Point A, Point B, Point C) { return Cross(B - A, C - B) > 0; }
//向量的模长
double Length(Vector A) { return sqrt(Dot(A, A)); }
//返回弧度制(rad)下的夹角
double Angle(Vector A, Vector B) { return acos(Dot(A, B) / Length(A) / Length(B)); }
//三个点确定两个向量(交点为A的两个向量AB和AC),然后求这两个向量的叉积(叉乘)
double Area2(Point A, Point B, Point C) { return Cross(B - A, C - A); }
//!向量的单位法线实际上就是将该向量向左旋转90°
//因为是单位法线所以长度归一化调用前请确保A不是零向量
Vector Normal(Vector A) {
  double L = Length(A);
  return Vector(-A.y / L, A.x / L);
}
//一个向量(或点)绕原点逆时针旋转rad弧度之后的新向量
Vector Rotate(Vector A, double rad) {
  return Vector(A.x * cos(rad) - A.y * sin(rad), A.x * sin(rad) + A.y * cos(rad));
}
//点A绕点B顺时针旋转rad
Point turn_PP(Point A, Point B, double rad) {
  double x = (A.x - B.x) * cos(rad) + (A.y - B.y) * sin(rad) + B.x;
  double y = -(A.x - B.x) * sin(rad) + (A.y - B.y) * cos(rad) + B.y;
  return Point(x, y);
}
```

## 线

```cpp
struct Line {  //线
  Vector v;
  Point p;
  Line(Vector _v, Point _p): V(_v), P(_p) {}
}
//点(C)与直线(AB)关系,1在左侧,-1在右侧,0在直线上
int relation(Point A, Point B, Point C) {
  int c = sgn(Cross(B - A, C - A));
  return c < 0 ? 1 : (c > 0 ? -1 : 0);
}
```

