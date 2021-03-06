.. _plotting:

Plotting
========

Introduction
------------

Labeled data enables expressive computations. These same
labels can also be used to easily create informative plots.

Xray's plotting capabilities are centered around
:py:class:`xray.DataArray` objects.
To plot :py:class:`xray.Dataset` objects
simply access the relevant DataArrays, ie ``dset['var1']``.
Here we focus mostly on arrays 2d or larger. If your data fits
nicely into a pandas DataFrame then you're better off using one of the more
developed tools there.

Xray plotting functionality is a thin wrapper around the popular
`matplotlib <http://matplotlib.org/>`_ library.
Matplotlib syntax and function names were copied as much as possible, which
makes for an easy transition between the two.
Matplotlib must be installed before xray can plot.

For more extensive plotting applications consider the following projects:

- `Seaborn <http://stanford.edu/~mwaskom/software/seaborn/>`_: "provides
  a high-level interface for drawing attractive statistical graphics."
  Integrates well with pandas.

- `Holoviews <http://ioam.github.io/holoviews/>`_: "Composable, declarative
  data structures for building even complex visualizations easily." Works
  for 2d datasets.

- `Cartopy <http://scitools.org.uk/cartopy/>`_: Provides cartographic
  tools.

Imports
~~~~~~~

.. ipython:: python
    :suppress:

    # Use defaults so we don't get gridlines in generated docs
    import matplotlib as mpl
    mpl.rcdefaults()

The following imports are necessary for all of the examples.

.. ipython:: python

    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    import xray

For these examples we'll use the North American air temperature dataset.

.. ipython:: python

    airtemps = xray.tutorial.load_dataset('air_temperature')
    airtemps

    # Convert to celsius
    air = airtemps.air - 273.15


One Dimension
-------------

Simple Example
~~~~~~~~~~~~~~

Xray uses the coordinate name to label the x axis.

.. ipython:: python

    air1d = air.isel(lat=10, lon=10)

    @savefig plotting_1d_simple.png width=4in
    air1d.plot()

Additional Arguments
~~~~~~~~~~~~~~~~~~~~~

Additional arguments are passed directly to the matplotlib function which
does the work.
For example, :py:func:`xray.plot.line` calls
matplotlib.pyplot.plot_ passing in the index and the array values as x and y, respectively.
So to make a line plot with blue triangles a matplotlib format string
can be used:

.. _matplotlib.pyplot.plot: http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.plot

.. ipython:: python

    @savefig plotting_1d_additional_args.png width=4in
    air1d[:200].plot.line('b-^')

.. note::
    Not all xray plotting methods support passing positional arguments
    to the wrapped matplotlib functions, but they do all
    support keyword arguments.

Keyword arguments work the same way, and are more explicit.

.. ipython:: python

    @savefig plotting_example_sin3.png width=4in
    air1d[:200].plot.line(color='purple', marker='o')

Adding to Existing Axis
~~~~~~~~~~~~~~~~~~~~~~~

To add the plot to an existing axis pass in the axis as a keyword argument
``ax``. This works for all xray plotting methods.
In this example ``axes`` is an array consisting of the left and right
axes created by ``plt.subplots``.

.. ipython:: python

    fig, axes = plt.subplots(ncols=2)

    axes

    air1d.plot(ax=axes[0])
    air1d.plot.hist(ax=axes[1])

    plt.tight_layout()

    @savefig plotting_example_existing_axes.png width=6in
    plt.show()

On the right is a histogram created by :py:func:`xray.plot.hist`.

Two Dimensions
--------------

Simple Example
~~~~~~~~~~~~~~

The default method :py:meth:`xray.DataArray.plot` sees that the data is
2 dimensional and calls :py:func:`xray.plot.pcolormesh`.

.. ipython:: python

    air2d = air.isel(time=500)

    @savefig 2d_simple.png width=4in
    air2d.plot()

All 2d plots in xray allow the use of the keyword arguments ``yincrease``
and ``xincrease``.

.. ipython:: python

    @savefig 2d_simple_yincrease.png width=4in
    air2d.plot(yincrease=False)

.. note::

    We use :py:func:`xray.plot.pcolormesh` as the default two-dimensional plot
    method because it is more flexible than :py:func:`xray.plot.imshow`.
    However, for large arrays, ``imshow`` can be much faster than ``pcolormesh``.
    If speed is important to you and you are plotting a regular mesh, consider
    using ``imshow``.

Missing Values
~~~~~~~~~~~~~~

Xray plots data with :ref:`missing_values`.

.. ipython:: python

    bad_air2d = air2d.copy()

    bad_air2d[dict(lat=slice(0, 10), lon=slice(0, 25))] = np.nan

    @savefig plotting_missing_values.png width=4in
    bad_air2d.plot()

Nonuniform Coordinates
~~~~~~~~~~~~~~~~~~~~~~

It's not necessary for the coordinates to be evenly spaced. Both
:py:func:`xray.plot.pcolormesh` (default) and :py:func:`xray.plot.contourf` can
produce plots with nonuniform coordinates.

.. ipython:: python

    b = air2d.copy()
    # Apply a nonlinear transformation to one of the coords
    b.coords['lat'] = np.log(b.coords['lat'])

    @savefig plotting_nonuniform_coords.png width=4in
    b.plot()

Calling Matplotlib
~~~~~~~~~~~~~~~~~~

Since this is a thin wrapper around matplotlib, all the functionality of
matplotlib is available.

.. ipython:: python

    air2d.plot(cmap=plt.cm.Blues)
    plt.title('These colors prove North America\nhas fallen in the ocean')
    plt.ylabel('latitude')
    plt.xlabel('longitude')
    plt.tight_layout()

    @savefig plotting_2d_call_matplotlib.png width=4in
    plt.show()

.. note::

    Xray methods update label information and generally play around with the
    axes. So any kind of updates to the plot
    should be done *after* the call to the xray's plot.
    In the example below, ``plt.xlabel`` effectively does nothing, since
    ``d_ylog.plot()`` updates the xlabel.

    .. ipython:: python

        plt.xlabel('Never gonna see this.')
        air2d.plot()

        @savefig plotting_2d_call_matplotlib2.png width=4in
        plt.show()

Colormaps
~~~~~~~~~

Xray borrows logic from Seaborn to infer what kind of color map to use. For
example, consider the original data in Kelvins rather than Celsius:

.. ipython:: python

    @savefig plotting_kelvin.png width=4in
    airtemps.air.isel(time=0).plot()

The Celsius data contain 0, so a diverging color map was used. The
Kelvins do not have 0, so the default color map was used.

Robust
~~~~~~

Outliers often have an extreme effect on the output of the plot.
Here we add two bad data points. This affects the color scale,
washing out the plot.

.. ipython:: python

    air_outliers = airtemps.air.isel(time=0).copy()
    air_outliers[0, 0] = 100
    air_outliers[-1, -1] = 400

    @savefig plotting_robust1.png width=4in
    air_outliers.plot()

This plot shows that we have outliers. The easy way to visualize
the data without the outliers is to pass the parameter
``robust=True``.
This will use the 2nd and 98th
percentiles of the data to compute the color limits.

.. ipython:: python

    @savefig plotting_robust2.png width=4in
    air_outliers.plot(robust=True)

Observe that the ranges of the color bar have changed. The arrows on the
color bar indicate
that the colors include data points outside the bounds.

Discrete Colormaps
~~~~~~~~~~~~~~~~~~

It is often useful, when visualizing 2d data, to use a discrete colormap,
rather than the default continuous colormaps that matplotlib uses. The
``levels`` keyword argument can be used to generate plots with discrete
colormaps. For example, to make a plot with 8 discrete color intervals:

.. ipython:: python

    @savefig plotting_discrete_levels.png width=4in
    air2d.plot(levels=8)

It is also possible to use a list of levels to specify the boundaries of the
discrete colormap:

.. ipython:: python

    @savefig plotting_listed_levels.png width=4in
    air2d.plot(levels=[0, 12, 18, 30])

You can also specify a list of discrete colors through the ``colors`` argument:

.. ipython:: python

    flatui = ["#9b59b6", "#3498db", "#95a5a6", "#e74c3c", "#34495e", "#2ecc71"]
    @savefig plotting_custom_colors_levels.png width=4in
    air2d.plot(levels=[0, 12, 18, 30], colors=flatui)

Finally, if you have `Seaborn <http://stanford.edu/~mwaskom/software/seaborn/>`_
installed, you can also specify a `seaborn` color palette to the ``cmap``
argument. Note that ``levels`` *must* be specified with seaborn color palettes
if using ``imshow`` or ``pcolormesh`` (but not with ``contour`` or ``contourf``,
since levels are chosen automatically).

.. ipython:: python

    @savefig plotting_seaborn_palette.png width=4in
    air2d.plot(levels=10, cmap='husl')

.. _plotting.faceting:

Faceting
--------

Faceting here refers to splitting an array along one or two dimensions and
plotting each group.
Xray's basic plotting is useful for plotting two dimensional arrays. What
about three or four dimensional arrays? That's where facets become helpful.

Consider the temperature data set. There are 4 observations per day for two
years which makes for 2920 values along the time dimension.
One way to visualize this data is to make a
seperate plot for each time period.

The faceted dimension should not have too many values;
faceting on the time dimension will produce 2920 plots. That's
too much to be helpful. To handle this situation try performing
an operation that reduces the size of the data in some way. For example, we
could compute the average air temperature for each month and reduce the
size of this dimension from 2920 -> 12. A simpler way is
to just take a slice on that dimension.
So let's use a slice to pick 6 times throughout the first year.

.. ipython:: python

    t = air.isel(time=slice(0, 365 * 4, 250))
    t.coords

Simple Example
~~~~~~~~~~~~~~

The easiest way to create faceted plots is to pass in ``row`` or ``col``
arguments to the xray plotting methods/functions. This returns a
:py:class:`xray.plot.FacetGrid` object.

.. ipython:: python

    @savefig plot_facet_dataarray.png height=12in
    g_simple = t.plot(x='lon', y='lat', col='time', col_wrap=3)

4 dimensional
~~~~~~~~~~~~~~

For 4 dimensional arrays we can use the rows and columns of the grids.
Here we create a 4 dimensional array by taking the original data and adding
a fixed amount. Now we can see how the temperature maps would compare if
one were much hotter.

.. ipython:: python

    t2 = t.isel(time=slice(0, 2))
    t4d = xray.concat([t2, t2 + 40], pd.Index(['normal', 'hot'], name='fourth_dim'))
    # This is a 4d array
    t4d.coords

    @savefig plot_facet_4d.png height=12in
    t4d.plot(x='lon', y='lat', col='time', row='fourth_dim')

Other features
~~~~~~~~~~~~~~

Faceted plotting supports other arguments common to xray 2d plots.

.. ipython:: python

    hasoutliers = t.isel(time=slice(0, 5)).copy()
    hasoutliers[0, 0, 0] = -100
    hasoutliers[-1, -1, -1] = 400

    @savefig plot_facet_robust.png height=12in
    g = hasoutliers.plot.pcolormesh('lon', 'lat', col='time', col_wrap=3,
                                    robust=True, cmap='viridis')

FacetGrid Objects
~~~~~~~~~~~~~~~~~

:py:class:`xray.plot.FacetGrid` is used to control the behavior of the
multiple plots.
It borrows an API and code from `Seaborn
<http://stanford.edu/~mwaskom/software/seaborn/tutorial/axis_grids.html>`_.
The structure is contained within the ``axes`` and ``name_dicts``
attributes, both 2d Numpy object arrays.

.. ipython:: python

    g.axes

    g.name_dicts

It's possible to select the :py:class:`xray.DataArray` or
:py:class:`xray.Dataset` corresponding to the FacetGrid through the
``name_dicts``.

.. ipython:: python

   g.data.loc[g.name_dicts[0, 0]]

Here is an example of using the lower level API and then modifying the axes after
they have been plotted.

.. ipython:: python

    g = t.plot.imshow('lon', 'lat', col='time', col_wrap=3, robust=True)

    for i, ax in enumerate(g.axes.flat):
        ax.set_title('Air Temperature %d' % i)

    bottomright = g.axes[-1, -1]
    bottomright.annotate('bottom right', (240, 40))

    @savefig plot_facet_iterator.png height=12in
    plt.show()

TODO: add an example of using the ``map`` method to plot dataset variables
(e.g., with ``plt.quiver``).

Maps
----

To follow this section you'll need to have Cartopy installed and working.

This script will plot the air temperature on a map.

.. literalinclude:: examples/cartopy_example.py

Here is the resulting image:

.. image:: examples/cartopy_example.png

Details
-------

Ways to Use
~~~~~~~~~~~

There are three ways to use the xray plotting functionality:

1. Use ``plot`` as a convenience method for a DataArray.

2. Access a specific plotting method from the ``plot`` attribute of a
   DataArray.

3. Directly from the xray plot submodule.

These are provided for user convenience; they all call the same code.

.. ipython:: python

    import xray.plot as xplt
    da = xray.DataArray(range(5))
    fig, axes = plt.subplots(ncols=2, nrows=2)
    da.plot(ax=axes[0, 0])
    da.plot.line(ax=axes[0, 1])
    xplt.plot(da, ax=axes[1, 0])
    xplt.line(da, ax=axes[1, 1])
    plt.tight_layout()
    @savefig plotting_ways_to_use.png width=6in
    plt.show()

Here the output is the same. Since the data is 1 dimensional the line plot
was used.

The convenience method :py:meth:`xray.DataArray.plot` dispatches to an appropriate
plotting function based on the dimensions of the ``DataArray`` and whether
the coordinates are sorted and uniformly spaced. This table
describes what gets plotted:

=============== ===========================
Dimensions      Plotting function
--------------- ---------------------------
1               :py:func:`xray.plot.line`
2               :py:func:`xray.plot.pcolormesh`
Anything else   :py:func:`xray.plot.hist`
=============== ===========================

Coordinates
~~~~~~~~~~~

If you'd like to find out what's really going on in the coordinate system,
read on.

.. ipython:: python

    a0 = xray.DataArray(np.zeros((4, 3, 2)), dims=('y', 'x', 'z'),
            name='temperature')
    a0[0, 0, 0] = 1
    a = a0.isel(z=0)
    a

The plot will produce an image corresponding to the values of the array.
Hence the top left pixel will be a different color than the others.
Before reading on, you may want to look at the coordinates and
think carefully about what the limits, labels, and orientation for
each of the axes should be.

.. ipython:: python

    @savefig plotting_example_2d_simple.png width=4in
    a.plot()

It may seem strange that
the values on the y axis are decreasing with -0.5 on the top. This is because
the pixels are centered over their coordinates, and the
axis labels and ranges correspond to the values of the
coordinates.
