# markdown-to-typst

easily convert markdown generated by LLM to typst! There might be some corner cases, tell me if you find them.

```rust
#import "@preview/mitex:0.2.5": *

#set text(font: ("libertinus serif", "Noto Serif CJK SC"))

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

// let's conclude that, two convert function I defined here are basically solving weird \ problem in str of typst

// this func manipulate on each equation captured by regex in func convert
#let convert_math(cont) = {
  // Why replace \t, \r by \\t? Cuz typst has problem dealing with string(I issued https://github.com/typst/typst/issues/5460. But it says it is intentional). In string format, \t doesn't become \\t in var, it remains \t, but \ becomes \\ in var. But \quad becomes \\quad! So math string like \text{} and \rightarrow cannot be converted by mitex-convert(mode: str) normally.
  let temp = cont.text.replace("\t", "\\t") // for \text, 
  temp = temp.replace("\r", "\\r") // for \right, 
  // Why not replace \n by \\n? Cuz \n appears at high frequency in a str, but \t, \r rarely appear(almost not?). So we must manually replace specific math symbol
  temp = temp.replace("\neq", "\\neq")

  // \\ in input string won't become \\\\ in var, instead in remains \\. This will cause wrong matrix display. So we need to replace it manually
  temp = temp.replace(regex("\\\\ |\\\\\n"), "\\\\ ") // replace \\ for \\\\ (fix matrix), or replace \\\n for \\\\ (fix cases)

  mitex-convert(mode: str, temp)
}

#let convert(cont) = {
  let temp
  temp = cont.replace(regex("(?m)^#{1,6}\s"), replace_title)
  temp = temp.replace(regex("\*\*.*?\*\*"), replace_strong)

  // no need to change math block manually, we have mitex! So commet 5 lines below
  // temp = temp.replace("\( ", "$")
  // temp = temp.replace(" \)", "$")
  // temp = temp.replace("\[", "$ ")
  // temp = temp.replace("\]", " $")
  // temp = temp.replace(regex("\{.*?\}"), replace_bracket)

  temp = temp.replace(regex("\$\$(.|\n)*?\$\$|\$(.|\n)*?\$|\\\\\((.|\n)*?\\\\\)|\\\\\[(.|\n)*?\\\\\]"), convert_math) // need to merge all convert_math to avoid order problem of temp
  // i gotta say it's really weird that, \ in input string will become \\ in var(hold the mouse on the var then you can see it becomes \\), and if you want to capture it in regex, you have to use \\\\. But when showed on pdf, \\ in var turns back to \
  // temp = temp.replace(regex("\\\\\((.|\n)*?\\\\\)"), convert_math) // .*? means minimum match, so that we can avoid adjacent \( ... \)
  // temp = temp.replace(regex("\\\\\[(.|\n)*?\\\\\]"), convert_math)
  
  eval(temp, mode: "markup", scope: mitex-scope) // render string as content
}

#align(center, text(17pt)[
  *HW1*
])

#grid(
  columns: (1fr),
  align(center)[
    djhbh
  ]
)

* #text([(1)], size: 1.6em) * 

#convert("#### **1. 矩阵定义与特征值**
- **Pauli矩阵**：  
  - \( \sigma_x = \begin{pmatrix} 0 & 1 \\ 1 & 0 \end{pmatrix} \)，\( \sigma_y = \begin{pmatrix} 0 & -i \\ i & 0 \end{pmatrix} \)，\( \sigma_z = \begin{pmatrix} 1 & 0 \\ 0 & -1 \end{pmatrix} \)。  
  - 三者均为厄米矩阵，特征值为 **\( \lambda = \pm 1 \)**。  
  - 以 \( \sigma_x \) 为例，解特征方程 \( \det(\sigma_x - \lambda I) = \lambda^2 - 1 = 0 \)，得 \( \lambda = 1 \) 和 \( \lambda = -1 \)；类似方法适用于 \( \sigma_y \) 和 \( \sigma_z \)。  

- **单位矩阵 \( I \)**：  
  - \( I = \begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix} \)。  
  - 特征方程为 \( \det(I - \lambda I) = (1 - \lambda)^2 = 0 \)，特征值 **\( \lambda = 1 \)**。

#### **2. 特征向量**
- **\(\sigma_x\)**：  
  - \( \lambda = 1 \) 时，解 \( (\sigma_x - I)\mathbf{v} = 0 \)，得 \( v_1 = v_2 \)，归一化为 \( \frac{1}{\sqrt{2}} \begin{pmatrix} 1 \\ 1 \end{pmatrix} \)。  
  - \( \lambda = -1 \) 时，解 \( (\sigma_x + I)\mathbf{v} = 0 \)，得 \( v_1 = -v_2 \)，归一化为 \( \frac{1}{\sqrt{2}} \begin{pmatrix} 1 \\ -1 \end{pmatrix} \)。  

- **\(\sigma_y\)**：  
  - \( \lambda = 1 \) 时，解 \( (\sigma_y - I)\mathbf{v} = 0 \)，得 \( v_1 = iv_2 \)，归一化为 \( \frac{1}{\sqrt{2}} \begin{pmatrix} 1 \\ i \end{pmatrix} \)。  
  - \( \lambda = -1 \) 时，解 \( (\sigma_y + I)\mathbf{v} = 0 \)，得 \( v_1 = -iv_2 \)，归一化为 \( \frac{1}{\sqrt{2}} \begin{pmatrix} 1 \\ -i \end{pmatrix} \)。  

- **\(\sigma_z\)**：  
  - 因已对角化，直接读出特征向量：\( \lambda = 1 \) 对应 \( \begin{pmatrix} 1 \\ 0 \end{pmatrix} \)，\( \lambda = -1 \) 对应 \( \begin{pmatrix} 0 \\ 1 \end{pmatrix} \)。  

- **\(I\)**：  
  - 所有非零向量均为特征向量，对应 \( \lambda = 1 \)。标准正交基为 \( \begin{pmatrix} 1 \\ 0 \end{pmatrix} \) 和 \( \begin{pmatrix} 0 \\ 1 \end{pmatrix} \)。

---

#### **3. 对角化表示**
- **\(\sigma_x\)**：  
  通过幺正矩阵 \( P = \frac{1}{\sqrt{2}} \begin{pmatrix} 1 & 1 \\ 1 & -1 \end{pmatrix} \) 对角化为 \( D = \begin{pmatrix} 1 & 0 \\ 0 & -1 \end{pmatrix} \)，即 \( \sigma_x = P D P^\dagger \)。  

- **\(\sigma_y\)**：  
  通过幺正矩阵 \( P = \frac{1}{\sqrt{2}} \begin{pmatrix} 1 & 1 \\ i & -i \end{pmatrix} \) 对角化为相同 \( D \)，即 \( \sigma_y = P D P^\dagger \)。  

- **\(\sigma_z\)**：  
  已为对角矩阵，\( D = \sigma_z \)，对应 \( P = I \)。  

- **\(I\)**：  
  天然对角形式，对任意基保持 \( D = I \)。")

* #text([(2)], size: 1.6em) * 

#convert("
#### **1. Pauli矩阵 \(\sigma_x\) 的特征向量**
- **\(\lambda = 1\) 的特征向量**：  
  \[
  |\psi\rangle = \frac{1}{\sqrt{2}} \begin{pmatrix} 1 \\ 1 \end{pmatrix} \quad \Rightarrow \quad \alpha = \frac{1}{\sqrt{2}},\beta = \frac{1}{\sqrt{2}}
  \]
  计算坐标：
  \[
  r_x = 2 \cdot \text{Re}\left(\frac{1}{\sqrt{2}} \cdot \frac{1}{\sqrt{2}}\right) = 1, \quad r_y = 0, \quad r_z = 0
  \]
  **Bloch球上的点**：\((1, 0, 0)\)（x轴正方向）。

- **\(\lambda = -1\) 的特征向量**：  
  \[
  |\psi\rangle = \frac{1}{\sqrt{2}} \begin{pmatrix} 1 \\ -1 \end{pmatrix} \quad \Rightarrow \quad \alpha = \frac{1}{\sqrt{2}},\beta = -\frac{1}{\sqrt{2}}
  \]
  计算坐标：
  \[
  r_x = -1, \quad r_y = 0, \quad r_z = 0
  \]
  **Bloch球上的点**：\((-1, 0, 0)\)（x轴负方向）。

#### **2. Pauli矩阵 \(\sigma_y\) 的特征向量**
- **\(\lambda = 1\) 的特征向量**：  
  \[
  |\psi\rangle = \frac{1}{\sqrt{2}} \begin{pmatrix} 1 \\ i \end{pmatrix} \quad \Rightarrow \quad \alpha = \frac{1}{\sqrt{2}},\beta = \frac{i}{\sqrt{2}}
  \]
  计算坐标：
  \[
  r_x = 2 \cdot \text{Re}\left(\frac{1}{\sqrt{2}} \cdot \frac{i}{\sqrt{2}}\right) = 0, \quad r_y = 2 \cdot \text{Im}\left(\frac{i}{2}\right) = 1, \quad r_z = 0
  \]
  **Bloch球上的点**：\((0, 1, 0)\)（y轴正方向）。

- **\(\lambda = -1\) 的特征向量**：  
  \[
  |\psi\rangle = \frac{1}{\sqrt{2}} \begin{pmatrix} 1 \\ -i \end{pmatrix} \quad \Rightarrow \quad \alpha = \frac{1}{\sqrt{2}}, \beta = -\frac{i}{\sqrt{2}}
  \]
  计算坐标：
  \[
  r_x = 0, \quad r_y = -1, \quad r_z = 0
  \]
  **Bloch球上的点**：\((0, -1, 0)\)（y轴负方向）。

#### **3. Pauli矩阵 \(\sigma_z\) 的特征向量**
- **\(\lambda = 1\) 的特征向量**：  
  \[
  |\psi\rangle = \begin{pmatrix} 1 \\ 0 \end{pmatrix} \quad \Rightarrow \quad \alpha = 1,\beta = 0
  \]
  计算坐标：
  \[
  r_x = 0, \quad r_y = 0, \quad r_z = 1
  \]
  **Bloch球上的点**：\((0, 0, 1)\)（z轴正方向，北极）。

- **\(\lambda = -1\) 的特征向量**：  
  \[
  |\psi\rangle = \begin{pmatrix} 0 \\ 1 \end{pmatrix} \quad \Rightarrow \quad \alpha = 0,\beta = 1
  \]
  计算坐标：
  \[
  r_x = 0, \quad r_y = 0, \quad r_z = -1
  \]
  **Bloch球上的点**：\((0, 0, -1)\)（z轴负方向，南极）。

#### **4. 单位矩阵 \(I\) 的特征向量**
- **任何非零向量都是 \(I\) 的特征向量**，对应特征值 \(\lambda = 1\)。  
- **标准基向量**：  
  - \(|0\rangle = \begin{pmatrix} 1 \\ 0 \end{pmatrix}\) 对应 **北极** \((0, 0, 1)\)。  
  - \(|1\rangle = \begin{pmatrix} 0 \\ 1 \end{pmatrix}\) 对应 **南极** \((0, 0, -1)\)。  
")
```
