# <code>distances</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
import math

import numpy as np


def hellinger(p, q):
    """

    Parameters
    ----------
    p

    q


    Returns
    -------

    """
    return math.sqrt(np.sum((np.sqrt(p) - np.sqrt(q)) ** 2) / 2)


def psi(p, q):
    """

    Parameters
    ----------
    p

    q


    Returns
    -------

    """
    return np.sum((p - q) * np.log(p / q))


def kl_divergence(p, q):
    """

    Parameters
    ----------
    p

    q


    Returns
    -------

    """
    kl = np.sum(p * np.log(p / q))
    return kl


def js_divergence(p, q):
    """

    Parameters
    ----------
    p

    q


    Returns
    -------

    """
    m = (p + q) / 2
    pm = kl_divergence(p, m)
    qm = kl_divergence(q, m)
    jsd = (pm + qm) / 2
    return jsd


def ks(p, q):
    """

    Parameters
    ----------
    p

    q


    Returns
    -------

    """
    return np.max(np.abs(np.cumsum(p) - np.cumsum(q)))
```
</pre>
</details>
## Functions
<dl>
<dt id="anovos.drift.distances.hellinger"><code class="name flex">
<span>def <span class="ident">hellinger</span></span>(<span>p, q)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>p</code></strong></dt>
<dd>&nbsp;</dd>
<dt><strong><code>q</code></strong></dt>
<dd>&nbsp;</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def hellinger(p, q):
    """

    Parameters
    ----------
    p

    q


    Returns
    -------

    """
    return math.sqrt(np.sum((np.sqrt(p) - np.sqrt(q)) ** 2) / 2)
```
</pre>
</details>
</dd>
<dt id="anovos.drift.distances.js_divergence"><code class="name flex">
<span>def <span class="ident">js_divergence</span></span>(<span>p, q)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>p</code></strong></dt>
<dd>&nbsp;</dd>
<dt><strong><code>q</code></strong></dt>
<dd>&nbsp;</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def js_divergence(p, q):
    """

    Parameters
    ----------
    p

    q


    Returns
    -------

    """
    m = (p + q) / 2
    pm = kl_divergence(p, m)
    qm = kl_divergence(q, m)
    jsd = (pm + qm) / 2
    return jsd
```
</pre>
</details>
</dd>
<dt id="anovos.drift.distances.kl_divergence"><code class="name flex">
<span>def <span class="ident">kl_divergence</span></span>(<span>p, q)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>p</code></strong></dt>
<dd>&nbsp;</dd>
<dt><strong><code>q</code></strong></dt>
<dd>&nbsp;</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def kl_divergence(p, q):
    """

    Parameters
    ----------
    p

    q


    Returns
    -------

    """
    kl = np.sum(p * np.log(p / q))
    return kl
```
</pre>
</details>
</dd>
<dt id="anovos.drift.distances.ks"><code class="name flex">
<span>def <span class="ident">ks</span></span>(<span>p, q)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>p</code></strong></dt>
<dd>&nbsp;</dd>
<dt><strong><code>q</code></strong></dt>
<dd>&nbsp;</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def ks(p, q):
    """

    Parameters
    ----------
    p

    q


    Returns
    -------

    """
    return np.max(np.abs(np.cumsum(p) - np.cumsum(q)))
```
</pre>
</details>
</dd>
<dt id="anovos.drift.distances.psi"><code class="name flex">
<span>def <span class="ident">psi</span></span>(<span>p, q)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>p</code></strong></dt>
<dd>&nbsp;</dd>
<dt><strong><code>q</code></strong></dt>
<dd>&nbsp;</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def psi(p, q):
    """

    Parameters
    ----------
    p

    q


    Returns
    -------

    """
    return np.sum((p - q) * np.log(p / q))
```
</pre>
</details>
</dd>
</dl>