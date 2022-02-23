# <code>validations</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
from functools import wraps, partial

from loguru import logger

from anovos.shared.utils import attributeType_segregation


def check_list_of_columns(
    func=None,
    columns="list_of_cols",
    target_idx: int = 1,
    target: str = "idf_target",
    drop="drop_cols",
):
    """

    Parameters
    ----------
    func
        Default value = None)
    columns
        Default value = "list_of_cols")
    target_idx : int
        Default value = 1)
    target : str
        Default value = "idf_target")
    drop
        Default value = "drop_cols")
    target_idx : int
        Default value = 1)
    target : str
        Default value = "idf_target")
    target_idx
        int:  (Default value = 1)
    target
        str:  (Default value = "idf_target")
    target_idx: int
         (Default value = 1)
    target: str
         (Default value = "idf_target")

    Returns
    -------

    """
    if func is None:
        return partial(check_list_of_columns, columns=columns, target=target, drop=drop)

    @wraps(func)
    def validate(*args, **kwargs):
        """

        Parameters
        ----------
        *args

        **kwargs


        Returns
        -------

        """
        logger.debug("check the list of columns")

        idf_target = kwargs.get(target, "") or args[target_idx]

        if isinstance(kwargs[columns], str):
            if kwargs[columns] == "all":
                num_cols, cat_cols, other_cols = attributeType_segregation(idf_target)
                cols = num_cols + cat_cols
            else:
                cols = [x.strip() for x in kwargs[columns].split("|")]
        elif isinstance(kwargs[columns], list):
            cols = kwargs[columns]
        else:
            raise TypeError(
                f"'{columns}' must be either a string or a list of strings."
                f" Received {type(kwargs[columns])}."
            )

        if isinstance(kwargs[drop], str):
            drops = [x.strip() for x in kwargs[drop].split("|")]
        elif isinstance(kwargs[drop], list):
            drops = kwargs[drop]
        else:
            raise TypeError(
                f"'{drop}' must be either a string or a list of strings. "
                f"Received {type(kwargs[columns])}."
            )

        final_cols = list(set(e for e in cols if e not in drops))

        if not final_cols:
            raise ValueError(
                f"Empty set of columns is given. Columns to select: {cols}, columns to drop: {drops}."
            )

        if any(x not in idf_target.columns for x in final_cols):
            raise ValueError(
                f"Not all columns are in the input dataframe. "
                f"Missing columns: {set(final_cols) - set(idf_target.columns)}"
            )

        kwargs[columns] = final_cols
        kwargs[drop] = []

        return func(*args, **kwargs)

    return validate


def check_distance_method(func=None, param="method_type"):
    """

    Parameters
    ----------
    func
        Default value = None)
    param
        Default value = "method_type")

    Returns
    -------

    """
    if func is None:
        return partial(check_distance_method, param=param)

    @wraps(func)
    def validate(*args, **kwargs):
        """

        Parameters
        ----------
        *args

        **kwargs


        Returns
        -------

        """
        dist_distance_methods = kwargs[param]

        if isinstance(dist_distance_methods, str):
            if dist_distance_methods == "all":
                dist_distance_methods = ["PSI", "JSD", "HD", "KS"]
            else:
                dist_distance_methods = [
                    x.strip() for x in dist_distance_methods.split("|")
                ]

        if any(x not in ("PSI", "JSD", "HD", "KS") for x in dist_distance_methods):
            raise TypeError(f"Invalid input for {param}")

        kwargs[param] = dist_distance_methods

        return func(*args, **kwargs)

    return validate
```
</pre>
</details>
## Functions
<dl>
<dt id="anovos.drift.validations.check_distance_method"><code class="name flex">
<span>def <span class="ident">check_distance_method</span></span>(<span>func=None, param='method_type')</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>func</code></strong></dt>
<dd>Default value = None)</dd>
<dt><strong><code>param</code></strong></dt>
<dd>Default value = "method_type")</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def check_distance_method(func=None, param="method_type"):
    """

    Parameters
    ----------
    func
        Default value = None)
    param
        Default value = "method_type")

    Returns
    -------

    """
    if func is None:
        return partial(check_distance_method, param=param)

    @wraps(func)
    def validate(*args, **kwargs):
        """

        Parameters
        ----------
        *args

        **kwargs


        Returns
        -------

        """
        dist_distance_methods = kwargs[param]

        if isinstance(dist_distance_methods, str):
            if dist_distance_methods == "all":
                dist_distance_methods = ["PSI", "JSD", "HD", "KS"]
            else:
                dist_distance_methods = [
                    x.strip() for x in dist_distance_methods.split("|")
                ]

        if any(x not in ("PSI", "JSD", "HD", "KS") for x in dist_distance_methods):
            raise TypeError(f"Invalid input for {param}")

        kwargs[param] = dist_distance_methods

        return func(*args, **kwargs)

    return validate
```
</pre>
</details>
</dd>
<dt id="anovos.drift.validations.check_list_of_columns"><code class="name flex">
<span>def <span class="ident">check_list_of_columns</span></span>(<span>func=None, columns='list_of_cols', target_idx: int = 1, target: str = 'idf_target', drop='drop_cols')</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>func</code></strong></dt>
<dd>Default value = None)</dd>
<dt><strong><code>columns</code></strong></dt>
<dd>Default value = "list_of_cols")</dd>
<dt><strong><code>target_idx</code></strong> :&ensp;<code>int</code></dt>
<dd>Default value = 1)</dd>
<dt><strong><code>target</code></strong> :&ensp;<code>str</code></dt>
<dd>Default value = "idf_target")</dd>
<dt><strong><code>drop</code></strong></dt>
<dd>Default value = "drop_cols")</dd>
<dt><strong><code>target_idx</code></strong> :&ensp;<code>int</code></dt>
<dd>Default value = 1)</dd>
<dt><strong><code>target</code></strong> :&ensp;<code>str</code></dt>
<dd>Default value = "idf_target")</dd>
<dt><strong><code>target_idx</code></strong></dt>
<dd>int:
(Default value = 1)</dd>
<dt><strong><code>target</code></strong></dt>
<dd>str:
(Default value = "idf_target")</dd>
<dt><strong><code>target_idx</code></strong> :&ensp;<code>int</code></dt>
<dd>(Default value = 1)</dd>
<dt><strong><code>target</code></strong> :&ensp;<code>str</code></dt>
<dd>(Default value = "idf_target")</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def check_list_of_columns(
    func=None,
    columns="list_of_cols",
    target_idx: int = 1,
    target: str = "idf_target",
    drop="drop_cols",
):
    """

    Parameters
    ----------
    func
        Default value = None)
    columns
        Default value = "list_of_cols")
    target_idx : int
        Default value = 1)
    target : str
        Default value = "idf_target")
    drop
        Default value = "drop_cols")
    target_idx : int
        Default value = 1)
    target : str
        Default value = "idf_target")
    target_idx
        int:  (Default value = 1)
    target
        str:  (Default value = "idf_target")
    target_idx: int
         (Default value = 1)
    target: str
         (Default value = "idf_target")

    Returns
    -------

    """
    if func is None:
        return partial(check_list_of_columns, columns=columns, target=target, drop=drop)

    @wraps(func)
    def validate(*args, **kwargs):
        """

        Parameters
        ----------
        *args

        **kwargs


        Returns
        -------

        """
        logger.debug("check the list of columns")

        idf_target = kwargs.get(target, "") or args[target_idx]

        if isinstance(kwargs[columns], str):
            if kwargs[columns] == "all":
                num_cols, cat_cols, other_cols = attributeType_segregation(idf_target)
                cols = num_cols + cat_cols
            else:
                cols = [x.strip() for x in kwargs[columns].split("|")]
        elif isinstance(kwargs[columns], list):
            cols = kwargs[columns]
        else:
            raise TypeError(
                f"'{columns}' must be either a string or a list of strings."
                f" Received {type(kwargs[columns])}."
            )

        if isinstance(kwargs[drop], str):
            drops = [x.strip() for x in kwargs[drop].split("|")]
        elif isinstance(kwargs[drop], list):
            drops = kwargs[drop]
        else:
            raise TypeError(
                f"'{drop}' must be either a string or a list of strings. "
                f"Received {type(kwargs[columns])}."
            )

        final_cols = list(set(e for e in cols if e not in drops))

        if not final_cols:
            raise ValueError(
                f"Empty set of columns is given. Columns to select: {cols}, columns to drop: {drops}."
            )

        if any(x not in idf_target.columns for x in final_cols):
            raise ValueError(
                f"Not all columns are in the input dataframe. "
                f"Missing columns: {set(final_cols) - set(idf_target.columns)}"
            )

        kwargs[columns] = final_cols
        kwargs[drop] = []

        return func(*args, **kwargs)

    return validate
```
</pre>
</details>
</dd>
</dl>