The ascp file, for example ascp2000.405, consists of blocks of 341 lines each. 
The block starts with a line like:

"1   1018" 

Where the first number is the block number, the second number is the number of coefficients in the block (unchanged). Each block contains a number of coefficients for 
approximating the equatorial rectangular coordinates (XYZ) of the planets of the Solar System within a 32-day interval. 
The block consists of 340 lines of 3 coefficients per line, a total of 1020 magic numbers, but the first two are not used.
The ascp2000.405 file does not start from January 1, 2000, as one might think, but from 12/24/1999 0.0 UT.

Header file.405 contains such an important group of numbers:

GROUP   1050
     3   171   231   309   342   366   387   405   423   441   753   819   899
    14    10    13    11     8     7     6     6     6    13    11    10    10
     4     2     2     1     1     1     1     1     1     8     2     4     4

The first vertical triple of numbers (3, 14, 4) is intended for Mercury. The coefficients for Mercury start from the 3rd coefficient in the block (and up to 170,
as you might guess), there are 14 coefficients for each coordinate, and the block interval of 32 days is divided into 4 subintervals, that is, for each 8-day
subinterval there is a set of coefficients.
Accordingly, the coefficient numbers for Venus in the block are from 171 to 230, 10 coefficients per coordinate, 2 subintervals (each subinterval = 16 days).

The coefficients for the bodies are arranged in the following order:
Mercury,  Venus,  Earth-Moon barycenter,  Mars ,  Jupiter ,  Saturn,  Uranus,  Neptune,  Pluto,  Moon (geocentric),  Sun.

Obviously, the first step is to calculate the number of the desired block:

MD0 = MJD(1999,12,24) – the initial Julian date of the ascp2000.405
MD - is the desired date for which ephemerides need to be calculated.
Interval = 32 (the length of the interval in days during which a separate block of coefficients is valid), constant
Block number:
NumerBlock= Int((MD - md0) / Interval) + 1
The start date of the desired block:
MD1 = MD0 + (NumerBlock - 1) * Interval

Then there is a need to calculate the length and number of the subinterval:
subInterval = Interval / Set_Body  - is the length of the subinterval in days
where Nset_Body is the number of subintervals for a specific body (the third line of the 1050 group in the header.405)
The number of full subintervals in the block before the desired date:
podint = Int((MD - MD1) / sub Interval) 
The initial date of the desired subinterval:
Mdat = MD1 + podint * subInterval
Shift of the beginning of reading coefficients taking into account the subinterval:
IndBegin = 3 * Nkoef_Body * podint
Where Nkoef_Body is the number of coefficients for this body (the second line of the 1050 group in header.405).

For example, let's take the Earth and want to calculate its barycentric coordinates on 03.01.2016 (day, month, year) 5:30 UT.
MD0 =  51536
MD = 57390.2299558333 (this is the specified date + 32.184 sec + TAI_UTC )
NumerBlock = 183
MD1 = 57360 (the 183rd block starts from this date)
subinterval = 16 (for Earth Nset_Body = 2)
podint = 1 (so many subintervals to skip)
Mdat = 57376 (the coefficients of the 2nd subinterval, which starts from this date, are needed)
From the 183rd block, we read the coefficients from 231 to 308 (the position of the coefficients for the Earth, the first line of the group 1050 in header.405), there are 78 of them in total, 
they belong to two subintervals of 16 days each. We need the coefficients of the second subinterval.
IndBegin  =    39
Skip the first 39 coefficients and read the remaining ones: kX(from 1 to 13) for X, kY(from 1 to 13) for Y, kZ(from 1 to 13) for Z.

The argument in Chebyshev polynomials is the normalized time (from -1 to 1) inside the subinterval:
     tau = 2 * (MD - Mdat) / subInterval - 1
For the example in question, tau= 0.778744479166562

There are 13 coefficients (for Earth), 13 Chebyshev polynomials are needed.
      p(1) = 1
      p(2) = tau
      p(i) = 2 * tau * p(i - 1) - p(i - 2)
And at the same time 13 derivatives of Chebyshev polynomials:
      pt(1) = 0
      pt(2) = 1
      pt(i) = 2 * (p(i - 1) + tau * pt(i - 1)) - pt(i - 2)

Calculate coordinates and velocities:
     X = kX(1)*p(1) + ... + kX(13)*p(13)
     X’ = kX(1)*pt(1) +... + kX(13)*pt(13)
     Y = kY(1)*p(1) + ... + kY(13)*p(13)

As a result, we have X,Y,Z barycentric in kilometers. To switch to the heliocentric coordinate system, you need to similarly calculate the barycentric vector of the Sun and take the difference of the vectors. To switch to the ecliptic coordinate system, the vector must be expanded accordingly. For the moon , the vector is calculated geocentric.

The calculated velocity vector must be divided by the coefficient koefVelocity = subInterval / 2 * 86400 (the number of seconds in half of the subinterval) to obtain
the dimension of km/sec.

For the example in question:
Barycentric position vector (-30121319.6, 132198969.6, 57283461.0)
Barycentric velocity vector ( -29.617, -5.792, -2.511)
Heliocentric position vector (-30680782.2, 131994773.2, 57221236.6)
Heliocentric velocity vector (-29.619, -5.803, -2.516)
