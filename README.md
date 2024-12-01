# markdown-to-typst

easily convert markdown generated by LLM to typst! There might be some corner cases, tell me if you find them.

```rust
#import "@preview/mitex:0.2.4": *

// #set text(font: ("libertinus serif", "Noto Serif CJK SC"))

#let replace_title(cont) = {
  "=" * (cont.end - cont.start - 1)
  " "
}

#let replace_strong(cont) = {
  cont.text.slice(1, -1)
}

/*
#let replace_bracket(cont) = {
  cont.text.slice(0, 1) + "(" + cont.text.slice(2, -1) + ")"
}
*/

#let convert_math(cont) = {
  // deal with escape symbols, they won't be added \ in front automatically
  let temp = cont.text.replace("\t", "\\t") // for \text, 
  temp = temp.replace("\r", "\\r") // for \right, 
  // caution! typst has problem dealing with string, thats why we cannot mannually replace \n, it may appears inside math equation. Also makes regex more complicated https://github.com/typst/typst/issues/5460. Let's hope there is no \r or \t in math
  // temp = temp.replace("\n", "\\n")
  mitex-convert(mode: str, temp)
}

#let convert(cont) = {
  let temp
  temp = cont.replace(regex("(?m)^#{1,6}\s"), replace_title)
  temp = temp.replace(regex("\*\*.*?\*\*"), replace_strong)
  // temp = temp.replace("\( ", "$")
  // temp = temp.replace(" \)", "$")
  // temp = temp.replace("\[", "$ ")
  // temp = temp.replace("\]", " $")
  // temp = temp.replace(regex("\{.*?\}"), replace_bracket)
  // no need to change math block manually, we have mitex!
  temp = temp.replace(regex("\\\\\((.|\n)*?\\\\\)|\\\\\[(.|\n)*?\\\\\]"), convert_math) // .*? means minimum match, so that we can avoid adjacent \( ... \) 
  eval(temp, mode: "markup", scope: mitex-scope) // render string as content
}

#convert("以下是一些复杂的数学公式，涵盖了不同的数学领域，包括微积分、线性代数、复分析、概率论等。

### 1. 高斯积分
高斯积分是概率论和统计学中的一个重要积分，表示为：
\[
\int_{-\infty}^{\infty} e^{-x^2} \, dx = \sqrt{\pi}
\]

### 2. 欧拉公式
欧拉公式是复分析中的一个重要公式，表示为：
\[
e^{i\pi} + 1 = 0
\]

### 3. 傅里叶变换
傅里叶变换是信号处理和分析中的一个重要工具，表示为：
\[
\mathcal{F}\{f(t)\} = \hat{f}(\omega) = \int_{-\infty}^{\infty} f(t) e^{-i\omega t} \, dt
\]

### 4. 拉普拉斯变换
拉普拉斯变换是微分方程求解中的一个重要工具，表示为：
\[
\mathcal{L}\{f(t)\} = F(s) = \int_{0}^{\infty} f(t) e^{-st} \, dt
\]

### 5. 矩阵的行列式
行列式是线性代数中的一个重要概念，表示为：
\[
\det(A) = \sum_{\sigma \in S_n} \text{sgn}(\sigma) \prod_{i=1}^n a_{i,\sigma(i)}
\]
其中 \(A\) 是一个 \(n \times n\) 的矩阵，\(\sigma\) 是 \(n\) 个元素的排列，\(\text{sgn}(\sigma)\) 是排列 \(\sigma\) 的符号。

### 6. 泰勒级数
泰勒级数是函数在某点附近的无穷级数展开，表示为：
\[
f(x) = \sum_{n=0}^{\infty} \frac{f^{(n)}(a)}{n!} (x-a)^n
\]
其中 \(f^{(n)}(a)\) 是函数 \(f\) 在点 \(a\) 处的第 \(n\) 阶导数。

### 7. 格林公式
格林公式是多元微积分中的一个重要公式，表示为：
\[
\oint_{\partial D} (P \, dx + Q \, dy) = \iint_D \left( \frac{\partial Q}{\partial x} - \frac{\partial P}{\partial y} \right) \, dA
\]
其中 \(D\) 是一个平面区域，\(\partial D\) 是 \(D\) 的边界。

### 8. 高斯-博内定理
高斯-博内定理是微分几何中的一个重要定理，表示为：
\[
\int_M K \, dA + \int_{\partial M} k_g \, ds = 2\pi \chi(M)
\]
其中 \(M\) 是一个二维紧致定向流形，\(K\) 是高斯曲率，\(k_g\) 是测地曲率，\(\chi(M)\) 是欧拉示性数。

### 9. 热方程
热方程是偏微分方程中的一个重要方程，表示为：
\[
\frac{\partial u}{\partial t} = \alpha \nabla^2 u
\]
其中 \(u(x, t)\) 是温度分布，\(\alpha\) 是热扩散系数。

### 10. 薛定谔方程
薛定谔方程是量子力学中的一个基本方程，表示为：
\[
i\hbar \frac{\partial \psi}{\partial t} = \hat{H} \psi
\]
其中 \(\psi\) 是波函数，\(\hat{H}\) 是哈密顿算符，\(\hbar\) 是约化普朗克常数。

这些公式涵盖了数学的多个领域，展示了数学的深度和广度。")

#convert("题目：计算函数 \( f(t) = e^{-|t|} \) 的傅里叶变换。

解答：

### 1. 定义傅里叶变换
傅里叶变换的定义为：
\[
\mathcal{F}\{f(t)\} = \hat{f}(\omega) = \int_{-\infty}^{\infty} f(t) e^{-i\omega t} \, dt
\]

### 2. 分析函数 \( f(t) \)
函数 \( f(t) = e^{-|t|} \) 是一个偶函数，因为 \( f(t) = f(-t) \)。我们可以将其分为两部分来处理：
\[
f(t) = \begin{cases} 
e^{-t} & \text{if } t \geq 0 \\
e^{t} & \text{if } t < 0 
\end{cases}
\]

### 3. 计算傅里叶变换
我们将积分分成两部分来计算：
\[
\hat{f}(\omega) = \int_{-\infty}^{\infty} e^{-|t|} e^{-i\omega t} \, dt = \int_{-\infty}^{0} e^{t} e^{-i\omega t} \, dt + \int_{0}^{\infty} e^{-t} e^{-i\omega t} \, dt
\]

### 4. 计算第一部分积分
对于 \( t < 0 \) 的部分：
\[
\int_{-\infty}^{0} e^{t} e^{-i\omega t} \, dt = \int_{-\infty}^{0} e^{t(1 - i\omega)} \, dt
\]

令 \( u = 1 - i\omega \)，则 \( du = dt \)，积分变为：
\[
\int_{-\infty}^{0} e^{tu} \, dt = \left[ \frac{e^{tu}}{u} \right]_{-\infty}^{0} = \frac{1}{1 - i\omega}
\]

### 5. 计算第二部分积分
对于 \( t \geq 0 \) 的部分：
\[
\int_{0}^{\infty} e^{-t} e^{-i\omega t} \, dt = \int_{0}^{\infty} e^{-t(1 + i\omega)} \, dt
\]

令 \( v = 1 + i\omega \)，则 \( dv = dt \)，积分变为：
\[
\int_{0}^{\infty} e^{-tv} \, dt = \left[ \frac{e^{-tv}}{-v} \right]_{0}^{\infty} = \frac{1}{1 + i\omega}
\]

### 6. 合并结果
将两部分积分结果合并：
\[
\hat{f}(\omega) = \frac{1}{1 - i\omega} + \frac{1}{1 + i\omega}
\]

### 7. 化简结果
将两个分数合并：
\[
\hat{f}(\omega) = \frac{1}{1 - i\omega} + \frac{1}{1 + i\omega} = \frac{(1 + i\omega) + (1 - i\omega)}{(1 - i\omega)(1 + i\omega)} = \frac{2}{1 + \omega^2}
\]

### 最终答案
\[
\boxed{\frac{2}{1 + \omega^2}}
\]")

#convert("题目：计算三维积分 \(\iiint_V (x^2 + y^2 + z^2) \, dV\)，其中 \(V\) 是由球面 \(x^2 + y^2 + z^2 = a^2\) 所围成的区域。

解答：

### 1. 确定积分区域
积分区域 \(V\) 是一个球体，其边界是球面 \(x^2 + y^2 + z^2 = a^2\)。我们可以使用球坐标系来简化积分。

### 2. 转换为球坐标系
在球坐标系中，三维空间中的点 \((x, y, z)\) 可以表示为 \((r, \theta, \phi)\)，其中：
- \(r\) 是点到原点的距离，范围是 \(0 \leq r \leq a\)。
- \(\theta\) 是与 \(z\) 轴的夹角，范围是 \(0 \leq \theta \leq \pi\)。
- \(\phi\) 是与 \(x\) 轴的夹角，范围是 \(0 \leq \phi \leq 2\pi\)。

在球坐标系中，体积元素 \(dV\) 可以表示为：
\[
dV = r^2 \sin \theta \, dr \, d\theta \, d\phi
\]

### 3. 转换被积函数
被积函数 \(x^2 + y^2 + z^2\) 在球坐标系中可以表示为：
\[
x^2 + y^2 + z^2 = r^2
\]

因此，被积函数变为 \(r^2\)。

### 4. 写出三重积分
将所有部分代入，我们得到三重积分：
\[
\iiint_V (x^2 + y^2 + z^2) \, dV = \int_0^{2\pi} \int_0^\pi \int_0^a r^2 \cdot r^2 \sin \theta \, dr \, d\theta \, d\phi
\]

### 5. 计算积分
首先计算 \(r\) 的积分：
\[
\int_0^a r^4 \, dr = \left[ \frac{r^5}{5} \right]_0^a = \frac{a^5}{5}
\]

然后计算 \(\theta\) 的积分：
\[
\int_0^\pi \sin \theta \, d\theta = \left[ -\cos \theta \right]_0^\pi = -\cos \pi - (-\cos 0) = 1 + 1 = 2
\]

最后计算 \(\phi\) 的积分：
\[
\int_0^{2\pi} d\phi = \left[ \phi \right]_0^{2\pi} = 2\pi
\]

### 6. 合并结果
将所有部分合并，我们得到：
\[
\iiint_V (x^2 + y^2 + z^2) \, dV = \left( \frac{a^5}{5} \right) \cdot 2 \cdot 2\pi = \frac{4\pi a^5}{5}
\]

### 最终答案
\[
\boxed{\frac{4\pi a^5}{5}}
\]")

#convert("题目：计算定积分 \(\int_0^1 x^2 e^x \, dx\)。

解答：

我们可以使用分部积分法来解决这个定积分问题。分部积分法的公式是：
\[
\int u \, dv = uv - \int v \, du
\]

首先，我们选择 \(u\) 和 \(dv\)：
\[
u = x^2 \quad \text{和} \quad dv = e^x \, dx
\]

接下来，我们计算 \(du\) 和 \(v\)：
\[
du = 2x \, dx \quad \text{和} \quad v = \int e^x \, dx = e^x
\]

将这些代入分部积分公式：
\[
\int_0^1 x^2 e^x \, dx = \left. x^2 e^x \right|_0^1 - \int_0^1 2x e^x \, dx
\]

我们先计算边界值：
\[
\left. x^2 e^x \right|_0^1 = 1^2 e^1 - 0^2 e^0 = e - 0 = e
\]

现在我们需要计算第二个积分 \(\int_0^1 2x e^x \, dx\)。我们再次使用分部积分法：
\[
u = 2x \quad \text{和} \quad dv = e^x \, dx
\]
\[
du = 2 \, dx \quad \text{和} \quad v = e^x
\]

将这些代入分部积分公式：
\[
\int_0^1 2x e^x \, dx = \left. 2x e^x \right|_0^1 - \int_0^1 2 e^x \, dx
\]

计算边界值：
\[
\left. 2x e^x \right|_0^1 = 2 \cdot 1 \cdot e^1 - 2 \cdot 0 \cdot e^0 = 2e - 0 = 2e
\]

接下来计算第二个积分：
\[
\int_0^1 2 e^x \, dx = 2 \int_0^1 e^x \, dx = 2 \left. e^x \right|_0^1 = 2 (e - 1)
\]

将所有部分合并：
\[
\int_0^1 x^2 e^x \, dx = e - (2e - 2(e - 1)) = e - 2e + 2(e - 1) = e - 2e + 2e - 2 = e - 2
\]

因此，定积分的值是：
\[
\int_0^1 x^2 e^x \, dx = e - 2
\]

最终答案是：
\[
\boxed{e - 2}
\]")
```
