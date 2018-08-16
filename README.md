# Flux Calculator

calculates flux in Jy for akari dataset.

## Getting Started

place the calculation.py into your working directory to be able to use the methods

### Prerequisites

Python 3
```
pip install numpy
```

```
pip install astropy
```

```
import RegionGrow
```






### Methods



```
calculation.getF(img.fits,error.fits)
```

Parameters: 

* **img.fits** - (string) fits file containg the object to preform the calculation
* **error.py** - (string) fits file containg the error values for each pixel of the img.fits file

Returns: 

* **values** - (tuple of floats) Jy value for flux and the associated error value in Jy. 




## Example Tutorial

Example: printing flux and error of a star in Jy from a fits file and its error file

-----------

```python
import RegionGrow # region grow repository is in same thread as this
import numpy
import astropy
from astropy.io import fits
import calculation

values=calculation.getF(img.fits,error.fits)

flux=values[0]
flux_error=calues[1]
```
-----------



*editors notes on Example:
 The Region Grow method is used within this calculation and the threshold is set to median+2*sigma. where medain is the median pixel value of the entire image and sigma is the standard deviation of every pixel.  

## Step by step of the calculation

**1.**

image data is extracted from both the img file and the error file

**2.**

image and error file go through a power law conversion the parameters for this conversion differ between bands so tis must be accounted for as shown below

```python
 ########################
    #   Surface Brightness correction for extended sources using the inverse power law
    #
    #           S'= (S/c)^(1/n)
    #           Where S' = corected value & S = origional data (Ueta et al. 2016)
    #
    #   Calibration is done for each pixel over entire map
    #   before any calculation takes place
    #  
    #           Values   
    #           n             c
    #     N60   .91+-.01      1.08+-.01
    #     Wides .90+-.01      1.28+-.01
    #     widel .92+-.01      1.39+-.02
    #     N160  .96+-.02      .52+-.01
    ########################
    N60n,Widesn,Wideln,N160n=.897,.875,.949,.977
    N60c,Widesc,Widelc,N160c=1.23,1.30,1.16,.462
    
    N60nerr,Widesnerr,Widelnerr,N160nerr=.009,.009,.009,.010
    N60cerr,Widescerr,Widelcerr,N160cerr=.03,.02,.02,.012
    
    if "lw_n" in str(inputfile):
        n=N160n
        c=N160c
        nerr=N160nerr
        cerr=N160cerr
    if "lw_w" in str(inputfile):
        n=Wideln
        c=Widelc
        nerr=Widelnerr
        cerr=Widelcerr
    if "sw_n" in str(inputfile):
        n=N60n
        c=N60c
        nerr=N60nerr
        cerr=N60cerr
    if "sw_w" in str(inputfile):
        c=Widesc
        n=Widesn
        nerr=Widescerr
        cerr=Widesnerr
    
    
    #######################
    #   surface brightness correction for error map
    #
    #error_map=(error_map/Widesc)**(1/Widesn) error propagation
    #      
    
    
    def ds(s):
        return (1/(s*n))*((s/c)**(1/n))
    
    
    def dn(s):
        s=abs(s)
        if(s==0):
            return 0.0
        else:
            return (-1.0)*(numpy.log(s/c)/(n*n))*((s/c)**(1.0/n))
    
    
    def dc(s):
        return(-1.0/(c*n))*((s/c)**(1.0/n))
        
    
    
    
    def converterror(imgpixel,errorpixel):
        test=math.sqrt((ds(imgpixel)*ds(imgpixel))*((errorpixel*errorpixel))+(dn(imgpixel)*dn(imgpixel))*(nerr*nerr)+
                             (dc(imgpixel)*dc(imgpixel))*(cerr*cerr))
        return test
    
    
    
    
    
    
    
    
    for y in range(numpy.shape(image_data)[0]):
            for x in range(numpy.shape(image_data)[1]):
                if(image_data[y][x]>0):
                    error_map[y][x]=converterror(image_data[y][x],error_map[y][x])
                    
                if(image_data[y][x]<0):
                    image_data[y][x]=image_data[y][x]*(-1) #change to negative then back to pos
                    error_map[y][x]=converterror(image_data[y][x],error_map[y][x])
                    image_data[y][x]=image_data[y][x]*(-1) 
                    
        
    
    
    
    #######################
    ##
    # some of the points are negative 
    # we must treat them as positive for calc
    #
    #######################
    
    
    for point in numpy.nditer(image_data,op_flags=['readwrite']):
        if point>0:
            point[...]=(point/c)**(1/n)
        if point<0:
            point[...]=point*(-1)
            point[...]=(point/c)**(1/n)
            point[...]=point*(-1)
```



## Built With

* [Beautifulsoup](https://www.crummy.com/software/BeautifulSoup/) 


## Authors

* **Andrew Torres** - *Initial work* 


## Acknowledgments

* Hat tip to anyone whose code was used
* Thanks Mom
* etc
