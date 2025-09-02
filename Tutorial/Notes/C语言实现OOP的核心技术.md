# C语言实现OOP的核心技术

C语言中实现面向对象编程（Object-Oriented Programming, OOP）是一个非常实用的技能，尤其在嵌入式系统、底层开发或需要与C++交互的场景中。虽然C语言本身并不原生支持OOP的三大特性（**封装、继承、多态**），但通过结构体（`struct`）、函数指针（`function pointer`）和指针操作等机制，可以有效地模拟这些特性。

## 封装（Encapsulation）

>   概念

封装是将数据（属性）和操作数据的函数（方法）绑定在一起，隐藏实现细节，只对外暴露接口。

>   C语言实现

在C语言中，使用 `struct` 来定义数据结构，使用函数指针来模拟方法。通过将结构体定义在 `.c` 文件中，外部只能通过头文件提供的函数接口访问对象。

>   示例：封装一个矩形类

`Rectangle.h`

```
#ifndef RECTANGLE_H
#define RECTANGLE_H

typedefstruct Rectangle Rectangle;

// 构造函数
Rectangle* rectangle_create(double width, double height);

// 析构函数
void rectangle_destroy(Rectangle* rect);

// 获取面积
double rectangle_area(const Rectangle* rect);

// 打印信息
void rectangle_print(const Rectangle* rect);

#endif // RECTANGLE_H
```

`Rectangle.c`

```
#include "Rectangle.h"
#include <stdio.h>
#include <stdlib.h>

// 真正的结构体定义
struct Rectangle {
    double width;
    double height;
};

Rectangle* rectangle_create(double width, double height) {
    Rectangle* rect = (Rectangle*)malloc(sizeof(Rectangle));
    if (rect) {
        rect->width = width;
        rect->height = height;
    }
    return rect;
}

void rectangle_destroy(Rectangle* rect) {
    free(rect);
}

double rectangle_area(const Rectangle* rect) {
    return rect->width * rect->height;
}

void rectangle_print(const Rectangle* rect) {
    printf("Rectangle: width = %.2f, height = %.2f, area = %.2f\n",
           rect->width, rect->height, rectangle_area(rect));
}
```

`main.c`

```
#include "Rectangle.h"

int main() {
    Rectangle* rect = rectangle_create(5.0, 3.0);
    rectangle_print(rect);
    rectangle_destroy(rect);
    return 0;
}
```

>   **封装效果**：外部无法直接访问 `width` 和 `height` 成员，只能通过 `rectangle_area()` 和 `rectangle_print()` 等接口操作。

## 继承（Inheritance）

>   概念

继承是面向对象中一个类（子类）继承另一个类（父类）的属性和方法。

>   C语言实现

C语言不支持继承语法，但可以通过**结构体嵌套**模拟继承。基类作为子类结构体的第一个成员，从而实现“is-a”关系。

>   示例：继承 Shape 类

`Shape.h`

```
#ifndef SHAPE_H
#define SHAPE_H

typedef struct Shape Shape;

// 构造函数
Shape* shape_create();
// 析构函数
void shape_destroy(Shape* shape);
// 虚函数：打印信息
void shape_print(const Shape* shape);

#endif // SHAPE_H
```

`Shape.c`

```
#include "Shape.h"
#include <stdio.h>
#include <stdlib.h>

struct Shape {
    void (*print)(const Shape*);
};

void shape_print(const Shape* shape) {
    printf("This is a Shape.\n");
}

Shape* shape_create() {
    Shape* shape = (Shape*)malloc(sizeof(Shape));
    if (shape) {
        shape->print = shape_print;
    }
    return shape;
}

void shape_destroy(Shape* shape) {
    free(shape);
}
```

`Rectangle.h`（继承 Shape）

```
#ifndef RECTANGLE_H
#define RECTANGLE_H

#include "Shape.h"

typedefstruct Rectangle {
    Shape base;  // 基类成员
    double width;
    double height;
} Rectangle;

Rectangle* rectangle_create(double width, double height);
void rectangle_destroy(Rectangle* rect);
void rectangle_print(const Rectangle* rect);

#endif // RECTANGLE_H
```

`Rectangle.c`

```
#include "Rectangle.h"
#include <stdio.h>
#include <stdlib.h>

static void rectangle_print_internal(const Shape* shape) {
    const Rectangle* rect = (const Rectangle*)shape;
    printf("Rectangle: width = %.2f, height = %.2f\n", rect->width, rect->height);
}

Rectangle* rectangle_create(double width, double height) {
    Rectangle* rect = (Rectangle*)malloc(sizeof(Rectangle));
    if (rect) {
        rect->base.print = rectangle_print_internal;
        rect->width = width;
        rect->height = height;
    }
    return rect;
}

void rectangle_destroy(Rectangle* rect) {
    free(rect);
}
```

`main.c`

```
#include "Rectangle.h"

int main() {
    Rectangle* rect = rectangle_create(5.0, 3.0);
    rect->base.print((Shape*)rect);  // 多态调用
    rectangle_destroy(rect);
    return 0;
}
```

>   **继承效果**：`Rectangle` 继承了 `Shape` 的 `print()` 方法，并重写了它。

## 多态（Polymorphism）

>   概念

多态是指同一个接口在不同对象上具有不同的行为。

>   C语言实现

通过**函数指针**和**虚函数表（vtable）**实现多态。每个对象维护一个指向函数指针数组的指针，该数组存储其所有方法的地址。

>   示例：多态实现（Shape、Rectangle、Circle）

`Shape.h`

```
#ifndef SHAPE_H
#define SHAPE_H

typedefstruct Shape Shape;

// 虚函数表
typedefstruct {
    void (*print)(const Shape*);
} ShapeVTable;

// 基类
struct Shape {
    const ShapeVTable* vtable;
};

// 构造函数
Shape* shape_create();
// 析构函数
void shape_destroy(Shape* shape);
// 虚函数调用
void shape_print(const Shape* shape);

#endif // SHAPE_H
```

`Shape.c`

```
#include "Shape.h"
#include <stdio.h>
#include <stdlib.h>

static void shape_print_default(const Shape* shape) {
    printf("This is a Shape.\n");
}

// 默认虚函数表
staticconst ShapeVTable shape_vtable = {
    .print = shape_print_default
};

Shape* shape_create() {
    Shape* shape = (Shape*)malloc(sizeof(Shape));
    if (shape) {
        shape->vtable = &shape_vtable;
    }
    return shape;
}

void shape_destroy(Shape* shape) {
    free(shape);
}

void shape_print(const Shape* shape) {
    shape->vtable->print(shape);
}
```

`Rectangle.h`

```
#ifndef RECTANGLE_H
#define RECTANGLE_H

#include "Shape.h"

typedefstruct Rectangle {
    Shape base;
    double width;
    double height;
} Rectangle;

Rectangle* rectangle_create(double width, double height);
void rectangle_destroy(Rectangle* rect);
void rectangle_print(const Rectangle* rect);

#endif // RECTANGLE_H
```

`Rectangle.c`

```
#include "Rectangle.h"
#include <stdio.h>
#include <stdlib.h>

static void rectangle_print(const Shape* shape) {
    const Rectangle* rect = (const Rectangle*)shape;
    printf("Rectangle: width = %.2f, height = %.2f\n", rect->width, rect->height);
}

// 虚函数表
staticconst ShapeVTable rectangle_vtable = {
    .print = rectangle_print
};

Rectangle* rectangle_create(double width, double height) {
    Rectangle* rect = (Rectangle*)malloc(sizeof(Rectangle));
    if (rect) {
        rect->base.vtable = &rectangle_vtable;
        rect->width = width;
        rect->height = height;
    }
    return rect;
}

void rectangle_destroy(Rectangle* rect) {
    free(rect);
}
```

`Circle.h`

```
#ifndef CIRCLE_H
#define CIRCLE_H

#include "Shape.h"

typedefstruct Circle {
    Shape base;
    double radius;
} Circle;

Circle* circle_create(double radius);
void circle_destroy(Circle* circle);
void circle_print(const Circle* circle);

#endif // CIRCLE_H
```

`Circle.c`

```
#include "Circle.h"
#include <stdio.h>
#include <stdlib.h>

static void circle_print(const Shape* shape) {
    const Circle* circle = (const Circle*)shape;
    printf("Circle: radius = %.2f\n", circle->radius);
}

// 虚函数表
staticconst ShapeVTable circle_vtable = {
    .print = circle_print
};

Circle* circle_create(double radius) {
    Circle* circle = (Circle*)malloc(sizeof(Circle));
    if (circle) {
        circle->base.vtable = &circle_vtable;
        circle->radius = radius;
    }
    return circle;
}

void circle_destroy(Circle* circle) {
    free(circle);
}
```

`main.c`

```
#include "Rectangle.h"
#include "Circle.h"

int main() {
    Shape* shapes[] = {
        (Shape*)rectangle_create(5.0, 3.0),
        (Shape*)circle_create(4.0)
    };

    for (int i = 0; i < 2; i++) {
        shape_print(shapes[i]);
        shape_destroy(shapes[i]);
    }

    return0;
}
```

>   **运行结果**：

```
Rectangle: width = 5.00, height = 3.00
Circle: radius = 4.00
```

>   **多态效果**：`shape_print()` 调用时会根据对象的实际类型（`Rectangle` 或 `Circle`）调用不同的 `print` 方法。

## 总结

C语言实现OOP的核心技术

| 特性 | 实现方式                       |
| :--- | :----------------------------- |
| 封装 | `struct` + 函数指针 + 静态函数 |
| 继承 | `struct` 嵌套                  |
| 多态 | 虚函数表（`vtable`）+ 函数指针 |

优点

-   **灵活性高**：可以在任何支持C的平台上使用。
-   **性能好**：没有运行时开销。
-   **兼容性强**：易于与C++、嵌入式系统等集成。

缺点

-   **手动管理复杂**：需要手动实现构造、析构、继承等。
-   **可读性差**：相比C++，代码更冗长、抽象层次低。
-   **缺乏编译器支持**：没有语法支持，容易出错。
