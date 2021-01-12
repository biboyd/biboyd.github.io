---
layout: single
title:  "SALSA Python Package"
author_profile: true
---

Over the Summer, while finishing up my research at MSU, I made the
[SALSA][salsa-git] python package as a sort of culmination of all/some of my
research work. I hope it'll end up being useful to others but it at least let
me discover and learn a lot of the different things that go into making a python
 package. This includes documentation, unit tests, how to design the software in
 a user friendly way (or at least try to).  In the end a small
[JOSS paper][salsa-paper] was written and it is currently being used in on
going research done by the [FOGGIE Collaboration](https://foggie.science).

More recently I finally got around to adding it to PyPI and it is now super easy
 to install with pip (though dependencies still might be a pain). To celebrate
that here is a post giving some explanation and quick walk through the package
and why it's useful. This will be a far less formal iteration than what you
might find in the paper or the documentation for SALSA which you can read
[here][salsa-rtd].

## What is SALSA?
SALSA stands for Synthetic Absorption Line Surveyor Application. It's purpose is to generate a catalog of synthetic observations from simulation data. A synthetic observation tries to mimic the way we observe real physical galaxies. Analyzing the simulation data in this way is useful in a few different ways.

* First, it makes it really easy to compare simulations to real observations.
* Second, it ensures that the results of your simulation analysis is physically observable.
	Maybe you found something really interesting about how galaxies form in your simulation. But if for whatever reason your results can't be probed by current observations. Then your results are far less useful since they can't be confirmed or denied by real observations. (unfortunately for theorists, observations are the truth)

So the observation that we are trying to mimic, through the use of [Trident][trident-git], are quasar absorption spectra. These give us informations by showing us what light is absorbed from the quasar and since different atoms absorb light at very specific wavelengths, this information tells us what type of stuff is there (hydrogen gas, helium gas, metals[^1] etc.). Additionally there is some other information that can be extracted.

## So, what is a Quasar??
Well that doesn't really matter for us. All we really care about is that it is a very bright and distant light source that shines it's light through the outer reaches of a galaxy. And the gas in the outer reaches of the galaxy, which we call the CGM, absorb some of the quasar's light.

So this is the main way to observe the CGM because it doesn't emit enough of it's own light for us to detect. And so in our simulations we can mimic quasars by creating a `LightRay` that travels through our simulated galaxy and then generate a spectra by using all the information we have available like gas density, temperature, content etc. We call this a synthetic observation because synthetic is a cool word.

## So What Does SALSA do Again??
SALSA doesn't directly make these `LightRay`'s or generate the spectra. That's hard and complicated and luckily some very nice people made software to do that called [Trident][trident-git], as mentioned before. So what SALSA adds is a streamlined way to both create a bunch of `LightRay`'s using Trident, and then process those `LightRay`'s to extract the data that your actually want.

Doing this allows us to generate a catalog of observations. This is what observers do when they study real quasars. A lot of that work is done by hand with manually fitting spectra. So my goal was to automate this process for our synthetic observations because otherwise I'd have to also do it by hand, which is painful once you start generating tens of thousands of synthetic observations.

### How Does it Extract the Useful Info? (SPICE)
So, a quick recap of what is happening that should help set the stage of what the SPICE method does which is the main method of extracting data that SALSA can use. So,

* Generate lightray which is just straight line going through a part of the galaxy.

* Look at all the gas that the lightray passes through

* does some calculations and generates a spectra based on that data

So where SPICE comes in is it tries to isolate which parts of the lightray are contributing to the spectra. We call these absorbers, and they are a contiquous group of cells with high enough gas density to show up in the spectra. Isolating these regions is super useful because the simulation holds a whole bunch of data in these cells like the temperature, density, metallicity, velocity and whatever else the modeler decided to keep track of in their simulation. What's even better is you never have to actually generate a spectra because the SPICE method works on the cell level data of the lightray.

### Is there a way to extract data from the spectra though?
Funny you should ask, yes there actually is. SALSA utilizes a package called [Spectacle][spectacle-git] which fits Voigt profiles to the synthetic spectra generated by Trident. A Voigt profile is a mix between a Gaussian and Lorentzian profile and is just what absorption spectra look like because of physics.

So having this method which looks at the spectra gives an entirely independent method of extracting information. The downside being it doesn't give you quite as much as the SPICE method does though it is closer to how observers get their information.

## Can we just Start using it Already?
Yes, yes, enough background information, let's write some code. 


[^1]: To Astronomers, metals are all the elements heavier than helium (so carbon, oxygen, basically everything). Yes, chemists tend to take offense to this.

[salsa-git]: https://github.com/biboyd/SALSA
[salsa-paper]: https://joss.theoj.org/papers/10.21105/joss.02581
[salsa-rtd]: https://salsa.readthedocs.io
[trident-git]: https://github.com/trident_project/trident
