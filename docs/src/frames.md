# Draw Frames

## Geographic basemaps

Geographic basemaps may differ from regular plot axis in that some projections support a “fancy”
form of axis and is selected by the MAP_FRAME_TYPE setting. The annotations will be formatted according
to the FORMAT_GEO_MAP template and MAP_DEGREE_SYMBOL setting. A simple example of part of a basemap
is shown in Figure Geographic map border.

```julia
using GMT
basemap(R="-1/2/0/0.4", proj="M8", frame="a1f15mg5m S")
t = [-1.0 0 0 1.0
    0.25 0 0 0.25
    1.25 0 0 0.08333332];
GMT.xy!(t, symbol="v2p+b+e+a60", lw=0.5, fill="black", y_offset="-1.0", no_clip=true)
if (GMTver < 6)
    T = ["-0.5 0.05 annotation", "0.375 0.05 frame", "1.29166666 0.05 grid"];
else
    T = text_record([-0.5 0.05; 0.375 0.05; 1.29166666 0.05], ["annotation", "frame", "grid"]);
end
text!(T, text_attrib="+f9p+jCB", fmt="png", show=true)
```

!["B_geo_1"](figures/B_geo_1.png)

The machinery for primary and secondary annotations axes can be utilized for geographic basemaps. This may
be used to separate degree annotations from minutes- and seconds-annotations. For a more complicated basemap
example using several sets of intervals, including different intervals and pen attributes for grid lines and
grid crosses.

```julia
using GMT
basemap(region="-2/1/0/0.35", proj="M10", frame="pa15mf5mg5m wSe s1f30mg15m", MAP_FRAME_TYPE="fancy+",
	MAP_GRID_PEN_PRIMARY="thinnest,black,.", MAP_GRID_CROSS_SIZE_SECONDARY=0.25, MAP_FRAME_WIDTH=0.2,
	MAP_TICK_LENGTH_PRIMARY=0.25, FORMAT_GEO_MAP="ddd:mm:ssF", FONT_ANNOT_PRIMARY="+8", FONT_ANNOT_SECONDARY=12)
# Draw Arrows and text
t = [-1.875 0 0 0.33333
    -0.45833 0 0 0.11111
    0.541666 0 0 0.11111]
GMT.xy!(t, symbol="v0.08+b+e+jc", lw=0.5, fill="black", y_offset=-1, no_clip=true)
if (GMTver < 6)
    T = ["-2.1 0.025 10p RM P:", "-1.875 0.05 6p CB annotation",
         "-0.45833 0.05 6p CB frame", "0.541666 0.05 6p CB grid"]
else
    T = text_record([-2.1 0.025; -1.875 0.05; -0.45833 0.05; 0.541666 0.05], ["10p RM P:", "6p CB annotation", "6p CB frame", "6p CB grid"])
end
text!(T, text_attrib="+f+j", no_clip=true)
t = [-1.5 0 0 1.33333; -0.25 0 0 0.66666; 0.625 0 0 0.33333]
GMT.xy!(t, symbol="v0.08+b+e+jc", lw=0.5, fill="black", y_offset=-0.6, no_clip=true)

if (GMTver < 6)
    T = ["-2.1 0.025 10p RM S:", "-1.5  0.05 9p CB annotation",
         "-0.25 0.05 9p CB frame", "0.625 0.05 9p CB grid"]
else
    T = text_record([-2.1 0.025; -1.5  0.05; -0.25 0.05; 0.625 0.05], ["10p RM S:", "9p CB annotation", "9p CB frame", "9p CB grid"])
end
text!(T, text_attrib="+f+j", no_clip=true, fmt="png", show=1)
```

!["B_geo_2"](figures/B_geo_2.png)

## Cartesian linear axes

For non-geographic axes, the MAP_FRAME_TYPE setting is implicitly set to plain. Other than that,
cartesian linear axes are very similar to geographic axes. The annotation format may be controlled with
the FORMAT_FLOAT_OUT parameter. By default, it is set to “%g”, which is a C language format statement
for floating point numbers, and with this setting the various axis routines will automatically determine
how many decimal points should be used by inspecting the stride settings. If FORMAT_FLOAT_OUT is set to
another format it will be used directly (.e.g, “%.2f” for a fixed, two decimals format). Note that for
these axes you may use the unit setting to add a unit string to each annotation.

```julia
using GMT
basemap(region="0/12/0/1", proj="X12/1", frame="-Ba4f2g1+lFrequency+u\" \%\" S")
t = [0 0 0 1.57; 6.0 0 0 0.79; 9.0 0 0 0.39]
GMT.xy!(t, symbol="v2p+b+e+a60", lw=0.5, fill="black", y_offset=0.25, no_clip=true, Vd=1)
if (GMTver < 6)
    T = ["2 0.2 annotation"; "7 0.2 frame"; "9.5 0.2 grid"]
else
    T = text_record([2 0.2; 7 0.2; 9.5 0.2], ["annotation", "frame", "grid"])
end
text!(T, text_attrib="+f9p+jCB", clearance="0.025/0.025", fill="white", fmt="png", show=1)
```

!["B_linear"](figures/B_linear.png)

## Cartesian log10 axes

Due to the logarithmic nature of annotation spacings, the stride parameter takes on specific meanings.
The following concerns are specific to log axes (see Figure Logarithmic projection axis):

   * stride must be 1, 2, 3, or a negative integer -n. Annotations/ticks will then occur at 1, 1-2-5,
     or 1,2,3,4,...,9, respectively, for each magnitude range. For -n the annotations will take place
     every n‘th magnitude.
   * Append l to stride. Then, log10 of the annotation is plotted at every integer log10 value
     (e.g., x = 100 will be annotated as “2”) [Default annotates x as is].
   * Append p to stride. Then, annotations appear as 10 raised to log10 of the value (e.g., 10-5).


```julia
using GMT
gmt("set MAP_GRID_PEN_PRIMARY thinnest,.")
basemap(region="1/1000/0/1", proj="X8l/0.7", frame="1f2g3p+l\"Axis Label\" S")
basemap!(frame="1f2g3l+l\"Axis Label\" S", y_offset=2.2)
basemap!(frame="1f2g3+l\"Axis Label\" S", y_offset=2.2, fmt="png", show=true)
```

!["B_log"](figures/B_log.png)

## Cartesian exponential axes

Normally, stride will be used to create equidistant (in the user’s unit) annotations or ticks,
but because of the exponential nature of the axis, such annotations may converge on each other
at one end of the axis. To avoid this problem, you can append p to stride, and the annotation
interval is expected to be in transformed units, yet the annotation itself will be plotted as
un-transformed units. E.g., if stride = 1 and power = 0.5 (i.e., sqrt), then equidistant
annotations labeled 1, 4, 9, ... will appear.

```julia
using GMT
gmt("set MAP_GRID_PEN_PRIMARY thinnest,.")
basemap(region="0/100/0/0.9", proj="X3ip0.5/0.25i", frame="a3f2g1p+l\"Axis Label\" S")
basemap!(frame="20f10g5+l\"Axis Label\" S",  y_offset=2.2, fmt="png", show=true)
```

!["B_pow"](figures/B_pow.png)

## Cartesian time axes

What sets time axis apart from the other kinds of plot axes is the numerous ways in which we may want
to tick and annotate the axis. Not only do we have both primary and secondary annotation items but we
also have interval annotations versus tick-mark annotations, numerous time units, and several ways in
which to modify the plot. We will demonstrate this flexibility with a series of examples. While all
our examples will only show a single x-axis (south, selected via -BS), time-axis annotations are
supported for all axes.

Our first example shows a time period of almost two months in Spring 2000. We want to annotate the month
intervals as well as the date at the start of each week. Note the leading hyphen in the FORMAT_DATE_MAP
removes leading zeros from calendar items (e.g., 03 becomes 3).

```julia
using GMT
basemap(region="2000-4-1T/2000-5-25T/0/1", proj="X12/0.5", frame="pa7Rf1d sa1O S",
        FORMAT_DATE_MAP="-o", FONT_ANNOT_PRIMARY="+9p", fmt="png", show=true)
```

!["B_time1"](figures/B_time1.png)

The next example shows two different ways to annotate an axis portraying 2 days in July 1969:

```julia
using GMT
gmt("set FORMAT_DATE_MAP \"o dd\" FORMAT_CLOCK_MAP hh:mm FONT_ANNOT_PRIMARY +9p")
basemap(region="1969-7-21T/1969-7-23T/0/1", proj="X12/0.5", frame="pa6Hf1h sa1K S")
basemap!(frame="pa6Hf1h sa1D S", y_offset=1.7, fmt="png", show=true)
```

The lower example chooses to annotate the weekdays (by specifying a1K) while the upper example
choses dates (by specifying a1D). Note how the clock format only selects hours and minutes
(no seconds) and the date format selects a month name, followed by one space and a two-digit
day-of-month number.

!["B_time2"](figures/B_time2.png)

The lower example chooses to annotate the weekdays (by specifying a1K) while the upper example
choses dates (by specifying a1D). Note how the clock format only selects hours and minutes
(no seconds) and the date format selects a month name, followed by one space and a two-digit
day-of-month number.

The third example presents two years, annotating both the years and every 3rd month.

```julia
using GMT
basemap(region="1997T/1999T/0/1", proj="X12/0.25", frame="pa3Of1o sa1Y S", FORMAT_DATE_MAP="o",
    FORMAT_TIME_PRIMARY_MAP="Character", FONT_ANNOT_PRIMARY="+9p", fmt="png", show=true)
```

Note that while the year annotation is centered on the 1-year interval, the month annotations must
be centered on the corresponding month and not the 3-month interval. The FORMAT_DATE_MAP selects
month name only and FORMAT_TIME_PRIMARY_MAP selects the 1-character, upper case abbreviation of
month names using the current language (selected by GMT_LANGUAGE).

!["B_time3"](figures/B_time3.png)

The fourth example only shows a few hours of a day, using relative time by specifying *t* in the
*region* option while the TIME_UNIT is *d* (for days). We select both primary and secondary annotations,
ask for a 12-hour clock, and let time go from right to left:

```julia
using GMT
gmt("set FORMAT_CLOCK_MAP=-hham FONT_ANNOT_PRIMARY +9p TIME_UNIT d")
basemap(region="0.2t/0.35t/0/1", proj="X-12/0.25", frame="pa15mf5m sa1H S",
    FORMAT_CLOCK_MAP="-hham", FONT_ANNOT_PRIMARY="+9p", TIME_UNIT="d", fmt="png", show=true)
```

!["B_time4"](figures/B_time4.png)

The fifth example shows a few weeks of time (Figure Cartesian time axis, example 5). The lower
axis shows ISO weeks with week numbers and abbreviated names of the weekdays. The upper uses
Gregorian weeks (which start at the day chosen by TIME_WEEK_START); they do not have numbers.

```julia
using GMT
gmt("set FORMAT_DATE_MAP u FORMAT_TIME_PRIMARY_MAP Character FORMAT_TIME_SECONDARY_MAP full
     FONT_ANNOT_PRIMARY +9p")
basemap(region="1969-7-21T/1969-8-9T/0/1", proj="X12/0.25", frame="pa1K sa1U S")
gmt("set FORMAT_DATE_MAP o TIME_WEEK_START Sunday FORMAT_TIME_SECONDARY_MAP Chararacter")
basemap!(frame="pa3Kf1k sa1r S", y_offset=1.7, fmt="png", show=true)
```

!["B_time5"](figures/B_time5.png)

Our sixth example shows the first five months of 1996, and we have annotated each month with an
abbreviated, upper case name and 2-digit year. Only the primary axes information is specified.

```julia
using GMT
basemap(region="1996T/1996-6T/0/1", proj="X12/0.25", frame="a1Of1d S",
    FORMAT_DATE_MAP="\"o yy\"", FORMAT_TIME_PRIMARY_MAP="Abbreviated", fmt="png", show=true)
```

!["B_time6"](figures/B_time6.png)

Our seventh and final example illustrates annotation of year-days. Unless we specify the
formatting with a leading hyphen in FORMAT_DATE_MAP we get 3-digit integer days. Note that in
order to have the two years annotated we need to allow for the annotation of small fractional
intervals; normally such truncated interval must be at least half of a full interval.

```julia
using GMT
gmt("set FORMAT_DATE_MAP jjj TIME_INTERVAL_FRACTION 0.05 FONT_ANNOT_PRIMARY +9p")
basemap(region="2000-12-15T/2001-1-15T/0/1", proj="X12/0.25", frame="pa5Df1d sa1Y S",
    FORMAT_DATE_MAP="jjj", TIME_INTERVAL_FRACTION=0.05, FONT_ANNOT_PRIMARY="+9p", fmt="png", show=true)
```

!["B_time7"](figures/B_time7.png)

## Custom axes

if (GMTver < 6)
    T1 = ["416.0 ig Devonian"; "443.7 ig Silurian"; "488.3 ig Ordovician"; "542 ig Cambrian"];
    T2 = ["0 a"; "1 a"; "2 f"; "2.71828 ag e"; "3 f"; "3.1415926 ag @~p@~"; "4 f"; "5 f"; "6 f";
          "6.2831852 ag 2@~p@~"];
else
    T1 = text_record([416.0 443.7 488.3 542], ["ig Devonian", "ig Silurian", "ig Ordovician", "ig Cambrian"]);
    T2 = text_record([0 1 2 2.71828 3 3.1415926 4 5 6 6.2831852],
                     ["a", "a", "f", "ag e", "f", "ag @~p@~", "f", "f", "f", "ag 2@~p@~"]);
end
basemap(T2,  region="416/542/0/6.2831852", proj="X-5i/2.5i", frame="WS+glightblue px25f5g25+u\" Ma\" pyc")
basemap!(T1, frame="WS sxc", MAP_ANNOT_OFFSET_SECONDARY="10p", MAP_GRID_PEN_SECONDARY="2p", show=1, Vd=1)
