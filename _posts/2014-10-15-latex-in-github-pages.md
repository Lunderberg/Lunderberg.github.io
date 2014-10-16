---
layout: post
title: LaTeX in Github Pages
tags: [site-info]
---

$\LaTeX$ is a useful tool for rendering documents,
especially documents with many equations.
While the document aspect doesn't translate very well to the web,
the capability to use equations is magnificent.
Simple equations are displayed better as $\frac{1}{x}$ than as 1/x,
and I would be hard-pressed to find a nice way to write
$\frac{\lambda^k e^{-\lambda}}{k!}$ without typesetting.

[MathJax](http://www.mathjax.org) is a javascript engine that can render Latex formulas.
By including it on each page, any formulas are appropriately rendered.
This can be enabled by including it in the header

```html
<script type="text/javascript"
	  src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```