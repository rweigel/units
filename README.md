# 2024-10-08

* We agreed that we should develop a translation of unit strings found Master CDFs ([Units.json](https://github.com/rweigel/cdawmeta-spase/blob/main/Units.json)).
* Need to agree on what standard to use [`VOUnits`](https://www.ivoa.net/documents/VOUnits/20231215/REC-VOUnits-1.1.html) and [`NIST SP 811`](https://www.nist.gov/pml/special-publication-811/) were both discussed. It seems that `NIST SP 811` does not provide a standard for how mathematical operations are represented, e.g., `cm^2` or `cm**2` whereas `VOUnits` does. It was agreed that the [`VOUnits` symbols for multiplication and exponentiation](https://www.ivoa.net/documents/VOUnits/20231215/REC-VOUnits-1.1.html#tth_sEc2.9) were disliked. However, if we agreed that it would be better to use an existing standard than to invent our own.
* Bob Weigel will discuss these notes with Baptiste at next week's DASH/IHDEA meeting.
* We agreed to meet again the week of October 21st.
* We decided the question of how to handle the [~10% (~10,000) of CDAWeb variables](http://mag.gmu.edu/git-data/cdawmeta/data/master_resolved/master_resolved.errors.ISTP.UNITS.log) needs futher investigation.
* Lee has [a sed script](https://github.com/spase-group/adapt/blob/2923ea562f297a206c848f345c01df1347f83d91/CDAWEB/variable_attribute_units.sed) that modified CDF `UNIT` strings.
* There was discussion of adding to [Units.json](https://github.com/rweigel/cdawmeta-spase/blob/main/Units.json) a field with the conversion factor to SI base units. There was some agreement that this was redundant because software exists to do this. However, it may be convenient.
* For Master CDF `UNIT` strings that are not scientific units (e.g., `0=Good`), we noted that this information needs to be preserved. For HAPI, if we use a `VOUnit` translation of the CDF `UNIT`, we add `x_UNIT_Original` to address this.
* We discussed that there are two types of numbers that do not have dimensions. One is a dimensionless number, such as Reynold's number. Another is a pure number, and some of them have names in VOUnits such as "pixel," "count," or "percent." The only `VOUnits` representation for something like `0=Good` seems to be "No unit, dimensionless." However, it seems that "No unit" should be distinguished from "dimensionless" - a Reynolds number is a different thing than a flag number such as `0=Good`, and ideally, they would be distinguished. I'll discuss this with Baptiste.
* Jeremy mentioned that time values in CDF files are relative to an epoch have units, but there is no attribute in HAPI or SPASE that allows one to specify the time the values are relative to. If one reads an epoch variable in CDF, one has to use the variable's data type and the CDF manual to determine the zero time. (Typically, a user does not need to do this because libraries translate epoch variable values to UTC, but I think Jeremy's point is that for completeness, the zero time should be in the metadata. NetCDF handles this by allowing strings like "seconds relative to 1970-01-01T00:00:00Z.)

**Action items**
* Bob - Meet with Baptiste
* All - study [`VOUnits`](https://www.ivoa.net/documents/VOUnits/20231215/REC-VOUnits-1.1.html) and [`NIST SP 811`](https://www.nist.gov/pml/special-publication-811/)
* Someone - Schedule a meeting for the week of October 21st

## Email from Bob before meeting:

All,

[Units.json](https://github.com/rweigel/cdawmeta-spase/blob/main/Units.json) contains a list of all `UNIT` strings found in CDF Masters (including those found in `UNITS_PTR`), excluding cases where the unit string is an empty string or all whitespace. The process we planned to follow was to replace nulls with the [`VOUnits`](https://www.ivoa.net/documents/VOUnits/20231215/REC-VOUnits-1.1.html) equivalent.

Although the [ISTP conventions](https://spdf.gsfc.nasa.gov/istp_guide/vattributes.html#UNITS) require units for variables with VAR_TYPE = data or support_data, [~10% (~10,000) of CDAWeb variables](http://mag.gmu.edu/git-data/cdawmeta/data/master_resolved/master_resolved.errors.ISTP.UNITS.log) have "missing" `UNITS`, meaning they do not have a `UNITS` or `UNIT_PTR` attribute or they have a unit value that is an empty string or all whitespace.

[units-CDFUNITS_to_SPASEUnit-map.json](http://mag.gmu.edu/git-data/cdawmeta/data/reports/units-CDFUNITS_to_SPASEUnit-map.json) contains keys of unique nonempty and not-all-whitespace unit strings found in Master CDF files. For each unit string encountered for a variable in a Master CDF, we looked up the unit string for that same variable in the SPASE record referenced by the spase_DatasetResourceID attribute in the Master CDF and counted the occurrences. So, for example,

```
[fraction]: {
    Ratio: 88,
   (cm^2 s sr MeV)^-1): 60
}
```

means that a CDF variable with units of `fraction` was found to be associated with a SPASE parameter with Units of `Ratio` 88 times and `(cm^2 s sr MeV)^-1)` 60 times (an incorrect translation). The 240 CDF variables where [fraction] occurred can be viewed in [a table](https://hapi-server.org/meta/cdaweb/variable/#UNITS='[fraction]').

Bob

# From Jeremy after meeting

> Here is the document on Steven's Levels of Measurement, which I've found very helpful in representing units:
https://en.wikipedia.org/wiki/Level_of_measurement
>
> Autoplot (and really Das2 which is under the hood) uses a Units class which is inspired by this, so I have units like CDF_TT2000 which is an interval scale, meaning that differences are meaningful (resulting in nanoseconds), but ratios are meaningless (2001-01-01/2001-01-01 makes no sense).  Units like "nT" are "Ratio Scale", meaning that ratios are meaningful as well as sums.  Note when I parse strings like "seconds since 2000-01-01T00:00Z" the "since" keyword is used to form an interval scale unit.
>
>I don't mean to say this is the right way to do things in this case, but you might find this interesting and helpful when thinking about the problem.
