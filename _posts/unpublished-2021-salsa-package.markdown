---
layout: single
title:  "SALSA Python Package"
author_profile: true
---

Over the Summer while finishing up my research at MSU I made the SALSA python package as a sort of culmination of all/some of my research work. I hope it'll end up being useful to others but it at least let me discover and learn a lot of the different things that go into making a python package. This includes documentation, unit tests, how to design the software in a user friendly way (or at least try to).  In the end a small [JOSS paper][salsa-paper] was written and it is currently being used in on going research done by the [FOGGIE Collaboration](https://foggie.science). 

More recently I finally got around to adding it to PyPI and it is now super easy to install with pip (though dependencies might be a pain). To celebrate that here is post giving some explanation and quick walk through the package and why it's useful. This will be a far less formal iteration than what you might find in the paper or the documentatio for SALSA which you can read [here][salsa-rtd].

## What is SALSA?
SALSA stands for Synthetic Absorption Line Surveyor Application. It's purpose is to generate a catalog of synthetic observations from simulation data. A synthetic observation tries to mimic the way we observe real physical galaxies. Analyzing the simulation data in this way is useful in a few different ways. 

* First, it makes it really easy to compare simulations to real observations. 
* Second, it ensures that the results of your simulation analysis is physically observable. 
	Maybe you found something really interesting about how galaxies form in your simulation. But if say it is only prelavent in the very dilute gas that can't be probed by current observations. Then your results are far less useful since they can't be confirmed or denied by real observations.
 


[salsa-paper]: https://joss.theoj.org/papers/10.21105/joss.02581
[salsa-rtd]: https://salsa.readthedocs.io
