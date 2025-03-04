# R 可视化 - 基本流程
Orion GUO

- [<span class="toc-section-number">1</span> R 可视化 -
  基本流程](#r-可视化---基本流程)
- [<span class="toc-section-number">2</span> 总结](#总结)

# R 可视化 - 基本流程

本文旨在总结近期学习内容，并保持持续输出。

核心聚焦于 R 语言可视化工具 `ggplot2`
的使用方法与绘图流程，结合散点图实例进行演示。

## **ggplot2 工作原理**

**数据可视化**通过线条、形状、颜色等视觉元素**映射（mapping）**数据变量，揭示其内在结构关系
。然而，并非所有映射均适用于所有变量类型，且某些表示形式更难解读。`ggplot2`
提供了一套系统化工具，用于将数据映射至图形的视觉元素，定义图形类型（如散点图、条形图等），并精细化控制其展示方式。

掌握 `ggplot2` 的关键在于理解其逻辑结构：代码通过***美学映射*（aesthetic
mapping）**连接数据变量与视觉元素（如颜色、点、形状）。在 `ggplot()`
函数中，首先指定数据及其变量如何映射至图形美学属性；随后，选择具体的图形类型（称为
*geom*），例如 `geom_point()` 绘制散点图，`geom_bar()`
绘制条形图，`geom_boxplot()` 绘制箱线图等。通过 “`+`” 符号，可将不同
geom 层叠加，构建复杂图形。

``` r
# 载入数据
library(pacman)
p_load(gapminder, bruceR)
data <- gapminder
data <- as_tibble(data)
names(data)
```

    [1] "country"   "continent" "year"      "lifeExp"   "pop"       "gdpPercap"

## **ggplot2 工作流程**

### **Step 1. 数据整理**

`ggplot2` 的核心要求是数据以**Tidy Data**
的形式组织，即长格式表结构。每一列代表一个变量，每一行代表一个观测值
\[\[5\]\]。在 Tidy Data 中，变量如 `year` 可称为*key*，而对应的值（如
`lifeExp`）则为*value*。

``` r
head(data)
```

    # A tibble: 6 × 6
      country     continent  year lifeExp      pop gdpPercap
      <fct>       <fct>     <int>   <dbl>    <int>     <dbl>
    1 Afghanistan Asia       1952    28.8  8425333      779.
    2 Afghanistan Asia       1957    30.3  9240934      821.
    3 Afghanistan Asia       1962    32.0 10267083      853.
    4 Afghanistan Asia       1967    34.0 11537966      836.
    5 Afghanistan Asia       1972    36.1 13079460      740.
    6 Afghanistan Asia       1977    38.4 14880372      786.

### **Step 2. 数据映射**

数据可视化的核心在于将数据变量映射到图形的视觉元素上。首先，定义绘图对象，通常为数据框或其增强形式（如
*tibble*）。通过 `ggplot()` 函数指定数据源，并使用 `aes()`
函数定义变量与视觉元素之间的映射关系。

例如，若要绘制人均
GDP（`gdpPercap`）与预期寿命（`lifeExp`）的关系图，需明确以下两点： 1.
**数据来源**：通过 `data` 参数指定。 2. **映射规则**：通过
`mapping = aes()` 定义，其中 `x` 和 `y` 分别对应横纵轴变量。

`aes()`
函数的作用是将数据中的变量链接到图形的视觉属性（如位置、颜色、形状等），但并不直接决定具体的视觉样式。这些映射仅说明数据中的哪些变量将由哪些视觉元素表示。

``` r
# 数据映射
p <- data %>% ggplot(mapping = aes(x = gdpPercap, 
                                   y = lifeExp))
```

### Step 3. 图形绘制

`p` 对象由 `ggplot()` 函数创建，其中已包含映射信息及默认设置（可通过
`str(p)`
查看详细结构）。然而，绘图尚未指定具体图形类型。为此，需添加一个*几何对象层*（geom），例如使用
`geom_point()` 绘制散点图。该函数根据 `x` 和 `y` 的映射值生成点状图形。

``` r
# 图形绘制
p + geom_point()
```

![](绘图流程_files/figure-commonmark/unnamed-chunk-4-1.png)

### Step 4. 图形调整

绘图默认采用笛卡尔坐标系，由 x 轴和 y
轴定义平面空间。若需进一步调整，可通过叠加不同 `geom_`
函数实现。此过程具有累加性：每一层均在前一层基础上构建，便于逐步完善图形。

例如，使用 `geom_smooth()`
可为数据拟合平滑曲线，并显示标准误差的置信带：

``` r
# 使用不同的geom_函数
p + geom_smooth()
```

    `geom_smooth()` using method = 'gam' and formula = 'y ~ s(x, bs = "cs")'

![](绘图流程_files/figure-commonmark/unnamed-chunk-5-1.png)

若需同时展示数据点与平滑曲线，可叠加 `geom_point()` 和 `geom_smooth()`：

``` r
p + geom_point() + geom_smooth()
```

    `geom_smooth()` using method = 'gam' and formula = 'y ~ s(x, bs = "cs")'

![](绘图流程_files/figure-commonmark/unnamed-chunk-6-1.png)

`geom_smooth()` 默认使用广义加法模型（GAM）进行拟合。通过调整其参数（如
`method = "lm"`），可切换为线性模型（LM）：

``` r
p + geom_point() + geom_smooth(method = "lm")
```

    `geom_smooth()` using formula = 'y ~ x'

![](绘图流程_files/figure-commonmark/unnamed-chunk-7-1.png)

当数据分布不均匀时（如人均 GDP 呈偏态分布），可通过对数变换优化 x
轴刻度。使用 `scale_x_log10()` 将 x 轴转换为以 10 为底的对数刻度：

``` r
p + geom_point() + geom_smooth(method = "lm") + 
  scale_x_log10()
```

    `geom_smooth()` using formula = 'y ~ x'

![](绘图流程_files/figure-commonmark/unnamed-chunk-8-1.png)

为进一步提升图形可读性，可调整轴标签格式。例如，将科学记数法替换为美元符号表示，可通过
`scales::dollar` 实现。此外，`scale_`
系列函数支持自定义轴刻度标签。例如，`labels = scales::comma`
可为数值添加千位分隔符，从而增强数据表达的直观性：

``` r
p + geom_point() + geom_smooth(method = "lm") + 
  scale_x_log10(labels = scales::dollar)
```

    `geom_smooth()` using formula = 'y ~ x'

![](绘图流程_files/figure-commonmark/unnamed-chunk-9-1.png)

### 拓展：美学映射 *Aesthetic Mapping*

美学映射（aesthetic
mapping）将数据变量与视觉元素（如颜色、大小、形状等）关联起来。例如，以下代码将
**`continent`** 变量映射到点的颜色属性

``` r
# 数据映射
p <- data %>% ggplot(mapping = aes(x = gdpPercap, 
                                   y = lifeExp,
                                   color = continent))
p + geom_point() +
    geom_smooth(method = "lm") +
    scale_x_log10()
```

    `geom_smooth()` using formula = 'y ~ x'

![](绘图流程_files/figure-commonmark/unnamed-chunk-10-1.png)

需要注意的是，**`aes()`**
仅用于定义变量与视觉元素的映射关系，而非直接指定固定值（如“紫色”）。若需设置固定属性（如点的颜色），应在
**`geom_`** 函数中直接指定，而非通过 **`aes()`**
实现。以下代码展示了如何设置固定颜色为紫色，并调整平滑线的样式：

``` r
p <- ggplot(data = gapminder,
            mapping = aes(x = gdpPercap,
                          y = lifeExp))
p + geom_point(color = "purple") +
    geom_smooth(method = "lm") +
    scale_x_log10()
```

    `geom_smooth()` using formula = 'y ~ x'

![](绘图流程_files/figure-commonmark/unnamed-chunk-11-1.png)

在此例中，**`color = "purple"`** 是直接应用于 **`geom_point()`**
的固定属性，而非通过 **`aes()`** 映射。此外，**`alpha`**
参数可用于控制透明度（范围为 0 到 1），适用于处理重叠数据点的情况：

``` r
p + geom_point(alpha = 0.3) +
    geom_smooth(color = "darkred", se = FALSE, size = 3, method = "lm") +
    scale_x_log10()
```

    Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
    ℹ Please use `linewidth` instead.

    `geom_smooth()` using formula = 'y ~ x'

![](绘图流程_files/figure-commonmark/unnamed-chunk-12-1.png)

### Step 5. 标签和标题

通过 **`labs()`** 函数可为图形添加标题、副标题、轴标签及注释。结合
**`scale_x_log10()`** 和 **`scales::dollar`**，可优化 x
轴刻度的显示格式。

``` r
p_out <- ggplot(data = gapminder, 
            mapping = aes(x = gdpPercap, 
                          y=lifeExp))

p_out <- p_out + geom_point(alpha = 0.3) +
         geom_smooth(method = "lm", color = "darkred", fill = "darkred") +
         scale_x_log10(labels = scales::dollar) +
         labs(x = "GDP Per Capita", y = "Life Expectancy in Years",
              title = "Economic Growth and Life Expectancy",
              subtitle = "Data points are country-years",
              caption = "Source: Gapminder.") +
         theme_bruce()
p_out
```

    `geom_smooth()` using formula = 'y ~ x'

![](绘图流程_files/figure-commonmark/unnamed-chunk-13-1.png)

### 拓展：**美学映射的继承与分离**

默认情况下，所有几何对象（**`geom_`**）会继承 **`ggplot()`**
中定义的美学映射。例如，以下代码按 **`continent`**
对点和平滑线进行分组着色：

``` r
p <- ggplot(data = gapminder,
            mapping = aes(x = gdpPercap,
                          y = lifeExp,
                          color = continent,
                          fill = continent))
p + geom_point() +
    geom_smooth(method = "loess") +
    scale_x_log10() +
    theme_bruce()
```

    `geom_smooth()` using formula = 'y ~ x'

![](绘图流程_files/figure-commonmark/unnamed-chunk-14-1.png)

若需分离美学映射，可在特定 **`geom_`** 函数中重新定义
**`aes()`**。例如，仅对点按 **`continent`** 着色，而平滑线保持统一颜色：

``` r
p <- ggplot(data = gapminder, mapping = aes(x = gdpPercap, y = lifeExp))
p + geom_point(mapping = aes(color = continent)) +
    geom_smooth(method = "loess", color = "darkred", fill = "darkred") +
    scale_x_log10() + 
    theme_bruce()
```

    `geom_smooth()` using formula = 'y ~ x'

![](绘图流程_files/figure-commonmark/unnamed-chunk-15-1.png)

除了分类变量，连续变量也可映射至美学属性。例如，将人口（**`pop`**）的对数值映射至颜色，生成渐变色图例。此方法适用于探索**连续变量**的分布模式，但需注意避免**过度分箱或离散化**，以保留数据的连续性特征：

``` r
p <- ggplot(data = gapminder,
            mapping = aes(x = gdpPercap,
                          y = lifeExp))
p + geom_point(mapping = aes(color = log(pop)),
               alpha = 0.3) +
    scale_x_log10() +
    theme_bruce()
```

![](绘图流程_files/figure-commonmark/unnamed-chunk-16-1.png)

### Step 6. 保存图形

使用 **`ggsave()`** 可将图形导出为多种格式（如 PDF、PNG
等）。以下代码将图形保存为 PDF 文件，并指定尺寸与分辨率。

``` r
## 保存图形
ggsave(here::here("figures", "图1.pdf"),
       plot = p_out, height = 8, width = 10, units = "in")
```

    `geom_smooth()` using formula = 'y ~ x'

# 总结

在探索数据可视化的过程中，应当牢记两个核心原则：**实践驱动学习和图层思维。**

- **实践驱动学习。**代码驱动的图形制作为我们提供了一个低风险、高回报的实验平台。每次尝试都是一次学习机会，有助于我们深入理解工具的本质，并可能揭示数据中隐藏的规律。

- **图层思维。**ggplot2
  的核心在于其独特的图层构建逻辑。从数据到可视化，遵循着一个清晰的流程：数据→美学映射→几何对象。这种模块化的方法使得复杂图表的创建变得系统化和可管理。
