# SLiPy

#### A Spectroscopy and astrophysical Library for Python 3

This Python package is an expanding code base for doing computational
astronomy, particularly spectroscopy. It contains both a *Spectrum* class
for handling spectra as objects (with +, -, \*, /, etc... operations defined)
and a growing suite of analysis tools.

**Dependencies:** Python 3.x, astropy, matplotlib, numpy, scipy

[![astropy](http://img.shields.io/badge/powered%20by-AstroPy-orange.svg?style=flat)](http://www.astropy.org/)

Quick note: the subpackage **astrolibpy** was not developed
by me. It was coded by Sergey Koposov (@segasai) at Cambridge (then at least).
I found it useful for performing velocity corrections on my spectroscopic
data. I've modified several modules such that it can be imported and used in
Python 3.x. See his README file.

## Modules

The following modules are elevated to the package level and are available
to import:

* [**Fits**](#Fits) [Find, RFind, GetData, Header, Search, PositionSort, ]

* [**DataType**](#DataType) [WaveVector, Spectrum, ]

* [**Simbad**](#Simbad) [Query, Position, Distance, Sptype, IDList, ]

* [**Correlate**](#Correlate) [XCorr, ]

* [**Telluric**](#Telluric) [Correct, ]

* [**Velocity**](#Velocity) [HelioCorrect, ]

* [**Observatory**](#Observatory) [OHP, ]

* [**Plot**](#Plot) [SPlot, Iterate, ]

* [**Elodie**](#Elodie) [Archive, Download, ]

* [**Montage**](#Montage) [Mosaic, SubField, Field, SolveGrid, ]

* [**Display**](#Display) [Monitor, ]

SLiPy is split into several components. The principle component is the
subpackage **SLiPy** itself, which contains all the relevant
functionality. Further, **Data** is a package I'm working on that will provide
an API for searching astronomical data archives in a simple way. The other two
subpackages **Framework** and **astrolibpy** are of utility to the project but
not necessarily intended for export. As stated previously, astrolibpy was not
developed by me, only modified. I'm not going to document it's usage here. Its
name is unfortunate for me as it is a bit over done with the convention I was
already using, but for consistency I will keep it as it was from the author.

#<a name=Fits></a>Fits

Manipulate FITS files. Import data into *Spectrum* objects. Filter results
by right ascension and declination. Grab header elements. Search for attributes
of the data such as distance, spectral type, etc.

- **Find** (*toplevel* = './', *pattern* = '\*.fits'):

    Search for file paths below *toplevel* fitting *pattern*. Returns a list
    of string values.

- **RFind** (*toplevel* = './', *pattern* = '\*.fits'):

    Recursively search for file paths below *toplevel* fitting *pattern*.
    Returns a list of string values.

- **GetData** ( \**files*, *\*\*kwargs*):
```Python
def GetData( *files, **kwargs ):
	"""
	Import data from FITS `files`. Returns a list of Spectrum objects.

	kwargs = {
			verbose   : True    , # display messages, progress
			toplevel  : ''      , # request import from directory `toplevel`
			pattern   : '*.fits', # pattern matching with `toplevel`
			recursive : False   , # search recursively below `toplevel`
			wavecal   : True    , # fit wavelength vector to data
			crpix1    : 'crpix1', # reference pixel header keyword
			crval1    : 'crval1', # value at reference pixel
			cdelt1    : 'cdelt1', # resolution (delta lambda)
		}
	"""
```
```Python
def Header( filename, keyword, **kwargs ):
	"""
	Retrieve `keyword` from FITS header in `filename`. Return type depends on
    the value.
	"""
```
```Python
def Search( *files, **kwargs ):
	"""
	Extract object names from Fits `files` and use Simbad module
	to resolve the `attribute` (a required keyword argument)
	from the SIMBAD astronomical database. Currently available attributes
    are 'Position', 'Distance', 'Sptype', and 'IDList'. Returns a list of
    results (type depends on the values).

	kwargs = {
			verbose   : True    , # display messages, progress
			toplevel  : ''      , # search under `toplevel` directory
			pattern   : '*.fits', # for files under `toplevel`
			recursive : False   , # search recusively under `toplevel`
			attribute : ''      , # attribute to search for (no default)
		}
	"""
```
```Python
def PositionSort( center, radius, *files, **kwargs ):
    """
    Return a list of files from `files` that lie in a `radius` (in degrees)
    from `center`, based on the `ra` (right ascension) and `dec` (declination).

    kwargs = {
            'ra'       : 'pos1'  , # header element for right ascension
            'dec'      : 'pos2'  , # header element for declination
            'obj'      : 'object', # header element for object id
            'raconvert': True    , # convert decimal hours to decimal degrees
            'verbose'  : True    , # display messages, progress
            'toplevel' : ''      , # `toplevel` directory to look for files in
            'recursive': False   , # search `recursive`ly below `toplevel`
            'pattern'  : '*.fits', # glob `pattern` for file search
            'useSimbad': False     # use Simbad instead of header elements
        }
    """

```
#<a name=DataType></a>DataType

Objects for representing astronomical data.

```Python
def WaveVector( rpix, rval, delt, npix ):
	"""
	Construct numpy array of wavelength values based on:

		`rpix` : reference pixel index
		`rval` : wavelength at reference pixel
		`delt` : resolutions (delta lambda)
		`npix` : length of desired array
	"""
```
```Python
class Spectrum:
	"""
	Spectrum objects consist of a `data` vector, and optionally a  
	`wavelength` vector (accessed with .data and .wave respectively).
    (+, -, *, /, +=, -=, *=, /=) are overloaded. The LHS spectrum is the
    reference and the RHS spectrum is resampled onto the wavelength space
    of the LHS spectrum before applying operations pixel-wise. Scalar
    operations applied to all pixels.
	"""
	def __init__(self, argument, **kwargs ):
		"""
		Hold spectrum `data` from file name `argument`. Alternatively,
		construct spectra with a one-dimensional numpy.ndarray as `argument`.
		`wavecal` is assumed to be true for file input.

		kwargs = {
    		'wavecal': True     , # fit wavelength vector to data
    		'crpix1' : 'crpix1' , # reference pixel header keyword
    		'crval1' : 'crval1' , # value at reference pixel
    		'cdelt1' : 'cdelt1' , # resolution (delta lambda)
		}
    """
```

#<a name=Simbad></a>Simbad

This module allows the user to query the SIMBAD astronomical database from
inside Python or shell commands/scripts.

As a shell script:

```
usage: Simbad.py @Attribute <identifier> [**kwargs]

The 'Attribute' points to a function within this module and indicates
what is to be run. Execute 'Simbad.py @Attribute help' for usage details of
a specific function. Currently available attributes are: `Position`,
`Distance`, `Sptype` and `IDList`.

The identifier names can be anything recognized by SIMBAD (e.g., Regulus,
"alpha leo", "HD 121475", "del cyg", etc ...) if the name is two parts make
sure to use quotation to enclose it.

The **kwargs is the conventional reference to Python keyword arguments.
These should be specific to the 'Attribute' being pointed to.
```

The following objects/functions are available:

```Python
class Query:
	"""
	Query( identifier, criteria, **kwargs ):

	Class for querying the SIMBAD astronomical database for 'citeria'
	of 'identifier'. This object is not intended to be used directly; it
    is an abstraction and is used by the other functions which should be
    called by the user.

	kwargs = {
		'parse' : True,  # extract relevant data from SIMBAD return file
		'dtype' : float, # output datatype
	}
	"""
```
```Python
def Position( identifier, **kwargs ):
	"""
	Position( identifier, **kwargs ):

	Handle to the Query class with criteria='%C00(d;C)'. Return right
    ascension and declination in decimal degrees of `identifier`.

    Example:

    ra, dec = Position('Sirius')
	"""
```
```Python
def Distance( identifier, **kwargs ):
	"""
	Distance( identifier, **kwargs ):

	Handle to the Query class with criteria='%PLX' Return the distance
    in parsecs to `identifier`.

    Example:

    d = Distance('rigel kent')
	"""
```
```Python
def Sptype(identifier, **kwargs):
	"""
	Sptype( identifier, **kwargs ):

	Handle to the Query class with criteria='%SP'. Return the
    spectral type as resolved by SIMBAD.

    Example:

    sptype = Sptype('HD 87901') # returns 'B8IVn' (HD 87901 is Regulus)
	"""
```
```Python
def IDList(identifier, **kwargs):
	"""
	IDList(identifier, **kwargs):

	Handle to the Query class with criteria='%IDLIST'.
	With `parse` = True, return a list of alternate IDs for
	the `identifier` provided.

    Example:

    other_names = IDList('proxima centauri')
    """
```

#<a name=Correlate></a>Correlate

Module of correlation functions for astronomical data.

```Python
def Xcorr( spectrumA, spectrumB, **kwargs ):
	"""
	Cross correlate two spectra of equal pixel length. The function returns
	an integer value representing the best shift within a `lag` based on
	the computed RMS of each configuration.
	"""
```

#<a name=Telluric></a>Telluric

Removal of atmospheric adsorption lines in spectra.

```Python
def Correct(spectrum, *calibration, **kwargs):
	"""
	Perform a telluric correction on `spectrum` with one or more
	`calibration` spectra. If more than one calibration spectrum is
	provided, the one with the best fit after performing both a
	horizontal cross correlation and a vertical amplitude fit is used.
	The spectrum and all the calibration spectra must have the same
	number of pixels (elements). If a horizontal shift in the calibration
	spectra is appropriate, only the corresponding range of the spectrum
	is divided out!

	kwargs = {
			lag  : 25            , # pixel shift limit for XCorr()
			range:(0.5, 2.0, 151), # numpy.linspace for amplitude trials
		}
	"""
```

![example](Figures/HD192640.png "Results of Telluric.Correct()")

**Figure 1:** The above figure is an example of applying the
**Telluric.Correct()** algorithm to a spectrum. In this case, six spectra of
*Regulus* from the Elodie archive were used as calibration spectra.

#<a name=Velocity></a>Velocity

Radial velocity corrections for 1D spectra.

```Python
def HelioCorrect( obs, *spectra, **kwargs ):
	"""
	Perform heliocentric velocity corrects on `spectra` based on
	`obs`ervatory information (longitude, latitude, altitude) and the
	member attributes, ra (right ascension), dec (declination), and jd
	(julian date) from the `spectra`.

    The `ra` and `dec` must be attached to each spectrum. `obs` should
    be of type Observatory.
	"""
```
```Python
def IrafInput( *files, **kwargs):
	"""
	Build an input file for IRAF's rvcorrect task.

	`files` should be a list of FITS file names to build the output table for.
	The user can optionally specify a `toplevel` directory to search for files
    under. If `outfile` is given, write results to the file.

	kwargs = {
		'toplevel' : ''      , # search `toplevel` directory for files
		'pattern'  : '*.fits', # files under `toplevel` fitting `pattern`
		'recursive': False   , # search recusively under `toplevel`
		'outfile'  : ''        # write lines to file named `outfile`
	}
	"""
```
```Python
def HeaderInfo( fpath ):
	"""
	Helper function of IrafInput().

	Return a formatted string containing the year, month, and day of
	observation from the FITS file with name `fpath`, as well as the
	universal time of observation and the right ascension and declination
	of the target.
	"""
```
#<a name=Observatory></a>Observatory

Define observatory parameter similar to the IRAF task. All observatories
should follow the following pattern. The user can add as many as they like
to this module. I welcome suggestions.

```Python
class OHP(Observatory):
	"""
	The Observatoire de Haute-Provence, France.
	"""
	def __init__(self):
		self.name      = 'Observatoire de Haute-Provence'
		self.latitude  = 43.9308334 # degrees N
		self.longitude = 356.28667  # degrees W
		self.altitude  = 650        # meters
```

#<a name=Plot></a>Plot

Convenience tools for plotting spectra. **SPlot** takes a Spectrum object
and remains like a handle to the plot for that object. All of the typical
member commands to matplotlib.pyplot exist, but once called are *remembered*.
Additionally, spectra can be `overlay`ed. **Iterate** is a function that takes
any number of SPlot figures and iterates through them interactively and lets
the user mark which ones to `keep`. The return is a list of either the names
of the figures or the actually objects themselves.

```Python
class SPlot:
	"""
	Spectrum Plot - Plot the data in `spectrum`.
	"""
	def __init__(self, spectrum, **kwargs):
		"""
		Assign `options` in `kwargs` and initialize the plot.

        kwargs = {
			'marker': 'b-'          , # marker for plot
			'label' : 'unspecified' , # label for data
			'usetex': False           # pdflatex setting
		}
        """
```
```Python
def Iterate( *plots, **kwargs ):
	"""
	Iterate thru `plots` to inspect data, the user marks `plots` of
	interest. The function returns a list of `names` marked.
	"""
```

#<a name=Elodie></a>Elodie

Methods for data retrieval from the Elodie Archive.

```Python
class Archive:
    """
    Import and parse ascii catalog of Elodie archive files. The complete
    archive is stored in the member `data`. It's organized in a dictionary
    by unique target names. The reduced archive of contains identified `HD`,
    `BD`, `GC`, and `GJ` objects, choosing the file pertaining to the spectra
    with the highest signal-to-noise ratio available.
    """
```
```Python
def Script(filename, pipeline=''):
	"""
	Construct url script for Elodie archive given `filename` and optionally
	`pipeline` instructions (e.g., `&z=wrs|fca[1,nor]`).
	"""
```
```Python
def Download( *files, **kwargs ):
    """
    Download `files` from Elodie archive via url scripts. The spectra can be
    further reduced via Elodie`s pipeline with the following options.

    kwargs = {
            'verbose'  : True           , # display messages, progress
            'resample' : (min, max, res), # resample spectra (no default)
            'normalize': True           , # continuum normalization
            'outpath'  : './'           , # directory for downloaded files
            'names'    : []               # alternative output names for `files`
        }
    """
```

#<a name=Montage></a>Montage

Wrapper to the *montage* mosaic software. See
[online](http://montage.ipac.caltech.edu/). The user should have Montage`s
executables available on their path. This module automates the process of
constructing large mosaics. It is largely modeled after the examples given on
the website. More documentation will be provided in the future ...

```Python
def Mosaic(resolution, *folders, **kwargs):
	"""
	Mosaic(resolution, *folders, **kwargs):

	Conduct standard build procedures for all `folders`. `resolution` is the
	number of pixels per degree for the output image. Note: `folders` should
	be absolute paths. See the M101 example online. All `folder`s should
    have subfolders "projected", "corrected", "final", and "differences"
    available as well as a "raw" directory containing the images to be
    mosaic-ed.

	kwargs = {
			verbose : True, # display messages, progress
			bkmodel : True  # model and correct for background effects
		}
	"""
```
```Python
def SolveGrid( sides, grid ):
	"""
	SolveGrid( sides, grid ):

	Helper function for the Field and SubField classes. Both `sides` and `grid`
	need to be array-like and of length two. `sides` is the side length of the
	field in decimal degrees in right ascension and declination respectively.
	`grid` specifies the subdivision along these axis (e.g., (2,2) says 2x2).

	The user should mindful of their choices. If the side lengths cannot be
	subdivided into well-behaved (rational) segments, higher decimal places
	will be lossed in the SubField.ArchiveList() task resulting in small
	gaps in the mosaic.
	"""
```

#<a name=Display></a>Display


```Python
class Monitor:
	"""
	Class for displaying a progress bar during iterative tasks.
	"""
	def __init__(self, **kwargs ):
        """
        kwargs = {
			'width'    : 45    , # number of characters wide
			'numbers'  : True  , # display numberical percent
			'template' : '[=>]', # template for progress bars
			'freq'     : 0.25  , # refresh rate
			'ETC'      : False , # display estimated time of completion
			'inline'   : True    # vanish after completion
		}
        """
```
