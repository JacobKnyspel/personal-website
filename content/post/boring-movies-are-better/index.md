---
title: Proving that Boring Movies are Better with Weighted Least Squares (WLS) Regression
date: 2022-11-01
math: true
image:
  placement: 1
  caption: "unsplash.com"
---

But first, what would our regression look like if we stick to using OLS estimation? Remember that in simple linear regression, we are trying to estimate the linear relationship between a predictor variable {{< math >}}$x${{< /math >}} and an outcome variable {{< math >}}$y${{< /math >}} using the following general model...

{{< math >}}
$$y=\alpha+\beta x$$
{{< /math >}}

...where {{< math >}}$\alpha${{< /math >}} is the intercept and {{< math >}}$\beta${{< /math >}} is the coefficient for the predictor variable {{< math >}}$x${{< /math >}}. This model is exactly the same for OLS and WLS regression — the only difference between the two is the way in which the intercept {{< math >}}$\alpha${{< /math >}} and the coefficient {{< math >}}$\beta${{< /math >}} are estimated. In OLS, they are estimated as follows...

{{< math >}}
$$\beta = \frac{\sum{\left(x_i-{\bar{x}}\right)\left(y_i-{\bar{y}}\right)}}{\sum{\left(x_i-{\bar{x}}\right)^2}}$$
$$\alpha={\bar{y}}-\beta{\bar{x}}$$
{{< /math >}}

This can be done very straightforwardly, since all we need is the individual values within each variable and the mean values of each variable, {{< math >}}$\bar{x}${{< /math >}} and {{< math >}}$\bar{y}${{< /math >}}. Let's enter the values from our IMDb dataset and see if we can estimate the relationship between movie year and movie rating...

{{< math >}}
$$\beta = \frac{-24542.44}{1449355} = -0.017$$
$$\alpha = {6.76}+0.017\times{2002.79} = 40.670$$
$$y = 40.670 - 0.017 x$$
{{< /math >}}

Looks like we can! Using OLS estimation, we calculated a coefficient value of {{< math >}}$\beta = -0.017${{< /math >}}, which means that for every 1 year that goes by, movie ratings are estimated to decrease by 0.017. We also produced an intercept value of {{< math >}}$\alpha = 40.670${{< /math >}}, which means that in the year 0, the estimated movie rating would have been 40.670 out of 10 (impressive!). 

Now let's see if we can get the same results in *R*. We use the `lm()` function to create a linear regression model with `Rating` as our outcome variable, `Year` as our predictor variable, and `imdb_data` as our dataset. By default this function will use OLS estimation. We can then use the `summary()` function to output a full description of our model.

```R
lm.1.1 <- lm(Rating ~ Year, data = imdb.data)
summary(lm.1.1)
```
{{< spoiler text="Click to view output" >}}
```
Call:
lm(formula = Rating ~ Year, data = imdb.data)

Residuals:
    Min      1Q  Median      3Q     Max 
-5.5153 -0.4983  0.0630  0.5985  2.8694 

Coefficients:
              Estimate Std. Error t value Pr(>|t|)    
(Intercept) 40.6698590  1.4913895   27.27   <2e-16 ***
Year        -0.0169334  0.0007446  -22.74   <2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 0.8965 on 5560 degrees of freedom
Multiple R-squared:  0.0851,	Adjusted R-squared:  0.08493 
F-statistic: 517.1 on 1 and 5560 DF,  p-value: < 2.2e-16
```
{{< /spoiler >}}

Sure enough, the intercept and coefficient values that the `lm()` function outputs are exactly the same as our own. We can also see that the relationship between movie year and movie rating is statistically significant, as shown by the calculated {{< math >}}$p${{< /math >}} value of 2.2e-16, which is the smallest possible value that *R* will display by default.

If we want to calculate the {{< math >}}$\eta^2${{< /math >}} effect size of this relationship along with confidence intervals, we can use the `eta_squared()` function from the `effectsize` package we imported earlier, specifying that our alternative hypothesis is *two-sided*. We do not need to differentiate between {{< math >}}$\eta^2${{< /math >}} and its standardised counterpart {{< math >}}$\eta_p^2${{< /math >}} for this because in simple linear regression the two are equivalent. We can also return the `eta_squared()` output as a matrix using the `as.matrix()` function if we want to view more than two decimals.

```R
eta_squared(lm.1.1, alternative = "two.sided")
as.matrix(eta_squared(lm.1.1, alternative = "two.sided"))
```
{{< spoiler text="Click to view output" >}}
```
Parameter | Eta2 |       95% CI
-------------------------------
Year      | 0.09 | [0.07, 0.10]

Parameter Eta2         CI     CI_low       CI_high     
"Year"    "0.08509522" "0.95" "0.07177453" "0.09920089"
```
{{< /spoiler >}}

Based on these outputs, we can say that a small yet significant negative relationship is observed between movie year and movie rating when OLS estimation is used, {{< math >}}$\beta${{< /math >}} = -0.017, {{< math >}}$t${{< /math >}}(5560) = -22.74, {{< math >}}$p${{< /math >}} < .001,  {{< math >}}$\eta^2${{< /math >}} = .085, 95\% CI = [.072, .099]. 

This all looks fine, but remember that we previously said that OLS estimation would be inappropriate for this regression, given the heteroskedasdicity we observed in our scatterplot. Now that we have built a model using OLS estimation, we can also generate a fitted vs residual plot using `ggplot` which should show us the same problem...

```R
ggplot(data = lm.1.1, aes(x = fitted(lm.1.1), y = resid(lm.1.1))) +
  geom_point(alpha = 0.2) +
  geom_hline(yintercept = 0) +
  labs(x = "Fitted", y = "Residuals", title = "Fitted vs Residuals Plot")
```
{{< spoiler text="Click to view output" >}}
{{< /spoiler >}}

Like our original scatterplot, this fitted vs residual plot is clearly fan-shaped, which is indicative of heteroskedasdicity and suggests that we are not fulfilling the statistical assumptions of OLS estimation. But if this is the case, we might wonder how OLS estimation still managed to give us results, and statistically significant results at that? 

Well, this speaks to an unfortunate truth about much of statistics — just because we are able to produce a statistically significant result using a particular method does not mean that that method is the best method to use, or even a correct method to use. In other words, *R* doesn't know whether we are fulfilling the assumptions of the methods we use. That is *our* responsibility to ensure.

{{< math >}}
$$\beta = \frac{\sum_{i=1}^{n}{w_i\left(x_i-{\bar{x}}_w\right)\left(y_i-{\bar{y}}_w\right)}}{\sum_{i=1}^{n}{w_i\left(x_i-{\bar{x}}_w\right)^2}}$$
$$\alpha={\bar{y}}_w-\beta{\bar{x}}_w$$
$${\bar{x}}_w=\frac{\sum_{i=1}^{n}{w_ix_i}}{\sum_{i=1}^{n}w_i}$$
$${\bar{y}}_w=\frac{\sum_{i=1}^{n}{w_iy_i}}{\sum_{i=1}^{n}w_i}$$
{{< /math >}}


Wowchemy is designed to give technical content creators a seamless experience. You can focus on the content and Wowchemy handles the rest.

**Highlight your code snippets, take notes on math classes, and draw diagrams from textual representation.**

On this page, you'll find some examples of the types of technical content that can be rendered with Wowchemy.

## Examples

### Code

Wowchemy supports a Markdown extension for highlighting code syntax. You can customize the styles under the `syntax_highlighter` option in your `config/_default/params.yaml` file.

    ```python
    import pandas as pd
    data = pd.read_csv("data.csv")
    data.head()
    ```

renders as

```python
import pandas as pd
data = pd.read_csv("data.csv")
data.head()
```

### Mindmaps

Wowchemy supports a Markdown extension for mindmaps.

Simply insert a Markdown `markmap` code block and optionally set the height of the mindmap as shown in the example below.

A simple mindmap defined as a Markdown list:

<div class="highlight">
<pre class="chroma">
<code>
```markmap {height="200px"}
- Hugo Modules
  - wowchemy
  - wowchemy-plugins-netlify
  - wowchemy-plugins-netlify-cms
  - wowchemy-plugins-reveal
```
</code>
</pre>
</div>

renders as

```markmap {height="200px"}
- Hugo Modules
  - wowchemy
  - wowchemy-plugins-netlify
  - wowchemy-plugins-netlify-cms
  - wowchemy-plugins-reveal
```

A more advanced mindmap with formatting, code blocks, and math:

<div class="highlight">
<pre class="chroma">
<code>
```markmap
- Mindmaps
  - Links
    - [Wowchemy Docs](https://wowchemy.com/docs/)
    - [Discord Community](https://discord.gg/z8wNYzb)
    - [GitHub](https://github.com/wowchemy/wowchemy-hugo-themes)
  - Features
    - Markdown formatting
    - **inline** ~~text~~ *styles*
    - multiline
      text
    - `inline code`
    -
      ```js
      console.log('hello');
      console.log('code block');
      ```
    - Math: $x = {-b \pm \sqrt{b^2-4ac} \over 2a}$
```
</code>
</pre>
</div>

renders as

```markmap
- Mindmaps
  - Links
    - [Wowchemy Docs](https://wowchemy.com/docs/)
    - [Discord Community](https://discord.gg/z8wNYzb)
    - [GitHub](https://github.com/wowchemy/wowchemy-hugo-themes)
  - Features
    - Markdown formatting
    - **inline** ~~text~~ *styles*
    - multiline
      text
    - `inline code`
    -
      ```js
      console.log('hello');
      console.log('code block');
      ```
    - Math: $x = {-b \pm \sqrt{b^2-4ac} \over 2a}$
```

### Charts

Wowchemy supports the popular [Plotly](https://plot.ly/) format for interactive charts.

Save your Plotly JSON in your page folder, for example `line-chart.json`, and then add the `{{</* chart data="line-chart" */>}}` shortcode where you would like the chart to appear.

Demo:

{{< chart data="line-chart" >}}

You might also find the [Plotly JSON Editor](http://plotly-json-editor.getforge.io/) useful.

### Math

Wowchemy supports a Markdown extension for $\LaTeX$ math. You can enable this feature by toggling the `math` option in your `config/_default/params.yaml` file.

To render _inline_ or _block_ math, wrap your LaTeX math with `{{</* math */>}}$...${{</* /math */>}}` or `{{</* math */>}}$$...$${{</* /math */>}}`, respectively. (We wrap the LaTeX math in the Wowchemy _math_ shortcode to prevent Hugo rendering our math as Markdown. The _math_ shortcode is new in v5.5-dev.)

Example **math block**:

```latex
{{</* math */>}}
$$
\gamma_{n} = \frac{ \left | \left (\mathbf x_{n} - \mathbf x_{n-1} \right )^T \left [\nabla F (\mathbf x_{n}) - \nabla F (\mathbf x_{n-1}) \right ] \right |}{\left \|\nabla F(\mathbf{x}_{n}) - \nabla F(\mathbf{x}_{n-1}) \right \|^2}
$$
{{</* /math */>}}
```

renders as

{{< math >}}
$$\gamma_{n} = \frac{ \left | \left (\mathbf x_{n} - \mathbf x_{n-1} \right )^T \left [\nabla F (\mathbf x_{n}) - \nabla F (\mathbf x_{n-1}) \right ] \right |}{\left \|\nabla F(\mathbf{x}_{n}) - \nabla F(\mathbf{x}_{n-1}) \right \|^2}$$
{{< /math >}}

Example **inline math** `{{</* math */>}}$\nabla F(\mathbf{x}_{n})${{</* /math */>}}` renders as {{< math >}}$\nabla F(\mathbf{x}_{n})${{< /math >}}.

Example **multi-line math** using the math linebreak (`\\`):

```latex
{{</* math */>}}
$$f(k;p_{0}^{*}) = \begin{cases}p_{0}^{*} & \text{if }k=1, \\
1-p_{0}^{*} & \text{if }k=0.\end{cases}$$
{{</* /math */>}}
```

renders as

{{< math >}}

$$
f(k;p_{0}^{*}) = \begin{cases}p_{0}^{*} & \text{if }k=1, \\
1-p_{0}^{*} & \text{if }k=0.\end{cases}
$$

{{< /math >}}

### Diagrams

Wowchemy supports a Markdown extension for diagrams. You can enable this feature by toggling the `diagram` option in your `config/_default/params.toml` file or by adding `diagram: true` to your page front matter.

An example **flowchart**:

    ```mermaid
    graph TD
    A[Hard] -->|Text| B(Round)
    B --> C{Decision}
    C -->|One| D[Result 1]
    C -->|Two| E[Result 2]
    ```

renders as

```mermaid
graph TD
A[Hard] -->|Text| B(Round)
B --> C{Decision}
C -->|One| D[Result 1]
C -->|Two| E[Result 2]
```

An example **sequence diagram**:

    ```mermaid
    sequenceDiagram
    Alice->>John: Hello John, how are you?
    loop Healthcheck
        John->>John: Fight against hypochondria
    end
    Note right of John: Rational thoughts!
    John-->>Alice: Great!
    John->>Bob: How about you?
    Bob-->>John: Jolly good!
    ```

renders as

```mermaid
sequenceDiagram
Alice->>John: Hello John, how are you?
loop Healthcheck
    John->>John: Fight against hypochondria
end
Note right of John: Rational thoughts!
John-->>Alice: Great!
John->>Bob: How about you?
Bob-->>John: Jolly good!
```

An example **Gantt diagram**:

    ```mermaid
    gantt
    section Section
    Completed :done,    des1, 2014-01-06,2014-01-08
    Active        :active,  des2, 2014-01-07, 3d
    Parallel 1   :         des3, after des1, 1d
    Parallel 2   :         des4, after des1, 1d
    Parallel 3   :         des5, after des3, 1d
    Parallel 4   :         des6, after des4, 1d
    ```

renders as

```mermaid
gantt
section Section
Completed :done,    des1, 2014-01-06,2014-01-08
Active        :active,  des2, 2014-01-07, 3d
Parallel 1   :         des3, after des1, 1d
Parallel 2   :         des4, after des1, 1d
Parallel 3   :         des5, after des3, 1d
Parallel 4   :         des6, after des4, 1d
```

An example **class diagram**:

    ```mermaid
    classDiagram
    Class01 <|-- AveryLongClass : Cool
    Class03 *-- Class04
    Class05 o-- Class06
    Class07 .. Class08
    Class09 --> C2 : Where am i?
    Class09 --* C3
    Class09 --|> Class07
    Class07 : equals()
    Class07 : Object[] elementData
    Class01 : size()
    Class01 : int chimp
    Class01 : int gorilla
    Class08 <--> C2: Cool label
    ```

renders as

```mermaid
classDiagram
Class01 <|-- AveryLongClass : Cool
Class03 *-- Class04
Class05 o-- Class06
Class07 .. Class08
Class09 --> C2 : Where am i?
Class09 --* C3
Class09 --|> Class07
Class07 : equals()
Class07 : Object[] elementData
Class01 : size()
Class01 : int chimp
Class01 : int gorilla
Class08 <--> C2: Cool label
```

An example **state diagram**:

    ```mermaid
    stateDiagram
    [*] --> Still
    Still --> [*]
    Still --> Moving
    Moving --> Still
    Moving --> Crash
    Crash --> [*]
    ```

renders as

```mermaid
stateDiagram
[*] --> Still
Still --> [*]
Still --> Moving
Moving --> Still
Moving --> Crash
Crash --> [*]
```

### Todo lists

You can even write your todo lists in Markdown too:

```markdown
- [x] Write math example
  - [x] Write diagram example
- [ ] Do something else
```

renders as

- [x] Write math example
  - [x] Write diagram example
- [ ] Do something else

### Tables

Save your spreadsheet as a CSV file in your page's folder and then render it by adding the _Table_ shortcode to your page:

```go
{{</* table path="results.csv" header="true" caption="Table 1: My results" */>}}
```

renders as

{{< table path="results.csv" header="true" caption="Table 1: My results" >}}

### Callouts

Academic supports a [shortcode for callouts](https://wowchemy.com/docs/content/writing-markdown-latex/#callouts), also referred to as _asides_, _hints_, or _alerts_. By wrapping a paragraph in `{{%/* callout note */%}} ... {{%/* /callout */%}}`, it will render as an aside.

```markdown
{{%/* callout note */%}}
A Markdown aside is useful for displaying notices, hints, or definitions to your readers.
{{%/* /callout */%}}
```

renders as

{{% callout note %}}
A Markdown aside is useful for displaying notices, hints, or definitions to your readers.
{{% /callout %}}

### Spoilers

Add a spoiler to a page to reveal text, such as an answer to a question, after a button is clicked.

```markdown
{{</* spoiler text="Click to view the spoiler" */>}}
You found me!
{{</* /spoiler */>}}
```

renders as

{{< spoiler text="Click to view the spoiler" >}} You found me! {{< /spoiler >}}

### Icons

Academic enables you to use a wide range of [icons from _Font Awesome_ and _Academicons_](https://wowchemy.com/docs/getting-started/page-builder/#icons) in addition to [emojis](https://wowchemy.com/docs/content/writing-markdown-latex/#emojis).

Here are some examples using the `icon` shortcode to render icons:

```markdown
{{</* icon name="terminal" pack="fas" */>}} Terminal  
{{</* icon name="python" pack="fab" */>}} Python  
{{</* icon name="r-project" pack="fab" */>}} R
```

renders as

{{< icon name="terminal" pack="fas" >}} Terminal  
{{< icon name="python" pack="fab" >}} Python  
{{< icon name="r-project" pack="fab" >}} R

### Did you find this page helpful? Consider sharing it 🙌
