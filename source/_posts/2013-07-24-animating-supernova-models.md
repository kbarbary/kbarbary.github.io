---
layout: post
title: "Animating Supernova Models"
date: 2013-07-24 18:26
comments: true
categories: 
---

I recently added the [Nearby Supernova Factory data](http://snfactory.lbl.gov/snf/data) for SN 2011fe to the built-in supernova
templates in the [sncosmo](http://sncosmo.readthedocs.org) python package.
I started wondering how easy it would be to visualize this and other models
using the animation package in matplotlib 1.1+, following the very nice
tutorial
[here](http://jakevdp.github.io/blog/2012/08/18/matplotlib-animation-tutorial/).
It turns out to be a lot easier than I thought. With a model in
SNCosmo, getting the spectrum of the object at any given time is just
a matter of:

```python
model.flux(time)
```

where `time` is the time in days and the return value is a numpy array
of the flux. Internally, a spline is used to interpolate between
phases having model data. Using this, I wrote a short function that
does the animation. Calling

```python
animate_model('2011fe', time_range=(None, 30.))
```

will pop up a window with an animation that looks something like this:

{% video /downloads/videos/2011fe.mp4 /downloads/videos/2011fe.webm 800 600 /downloads/videos/2011fe.png %}

(except better because it won't be compressed using H.264 or VP8). Above,
the `time_range` is used to restrict the animation to the model phases
before the rather large gap (between 23.7 and 74.1 days) where the
interpolation becomes rather poor. The `model.flux()` call only takes
about 200 microseconds:

```python
In [1]: import sncosmo
In [2]: model = sncosmo.get_model('2011fe')
In [3]: timeit model.flux(0.)
10000 loops, best of 3: 182 us per loop
```

So it is plenty fast to animate the model live at 30 frames per
second.  Saving the animation to a file takes longer due to the need
to write each frame to a file and pass them all to ffmpeg.

Comparing models
----------------

That's great, but what would be *really* useful is the ability to
compare two models side by side. Here's the call to do that:

```python
salt2model = sncosmo.get_model('salt2')
salt2model.set(x1=-0.206, c=-0.066)
animate_model([salt2model, '2011fe'],  time_range=(None, 30.))
```

We could have simply done `animate_model(['salt2', '2011fe'], ...` but
this would have given us the default SALT2 parameters, x1=0. and
c=0. Here, we have set x1 and c to the best fit parameters reported by
[Pereira et al. (2013)](http://adsabs.harvard.edu/abs/2013A%26A...554A..27P).

{% video /downloads/videos/salt2_vs_2011fe.mp4 /downloads/videos/salt2_vs_2011fe.webm 800 600 /downloads/videos/salt2_vs_2011fe.png %}

The SALT2 template actually matches 2011fe surprisingly well after
maximum light! However, 2011fe has a faster rise that the x1 parameter
isn't able to capture, resulting in a poor match at early times. One
additional caveat here is that we're using an overly simple way to
match the phases and overall flux scale of the models: For each model,
the date of B-band maximum is determined. The models are offset in
phase so that this date matches. The fluxes are scaled so that the
B-band flux at this date matches. This is very easy to do with sncosmo
models, as the phase of B-band maximum (by default) is determined upon
loading the model, and can be retrieved by

```python
model.refband   # band used (default is B-band)
model.refphase  # phase of maximum flux in refband 
model.bandflux(model.refband, model.refphase)  # maximum B-band flux
```

A slightly better visual fit impression might be achieved by minimizing
the differences over the entire light curve rather than just this single point.

Two more interesting notes:

* The 2011fe template seems to bounce on the way up, most likely a result
  of the spline interpolation.
* The oscillation of the SALT2 model is evident, particularly at early times
  and at wavelengths above about 8700 angstroms, where the model seems
  completely unconstrained.


SALT2 versus Hsiao
------------------

As a final exercise, I compared the SALT2 and Hsiao templates:

```python
animate_model(['salt2', 'hsiao'],  time_range=(None, 30.), disp_range=(2000., 9200.))
```

where the `disp_range` keyword is used to restrict the plot to the wavelengths where both templates are defined.

{% video /downloads/videos/salt2_vs_hsiao.mp4 /downloads/videos/salt2_vs_hsiao.webm 800 600 /downloads/videos/salt2_vs_hsiao.png %}

The `animate_model` function is not (yet) part of sncosmo, but is available as a GitHub gist [here](https://gist.github.com/kbarbary/6075466).