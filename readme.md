DHS Survey download tool (FME)
------------------------------

The DHS website is several links “deep” in terms of downloading survey files; i.e. there is not a single page linking to all survey ZIP files. However the URLs of the various pages follow some common structures so I’ve built an FME workbench to download all the surveys. 

This uses regular expressions and basic text searches to match certain lines that are common on the pages in order to derive the URLs required. Of course it could break whenever DHS change their website!

It is driven by DHS numeric survey IDs. These are used to build the URL for the corresponding survey info page on the DHS website. These have the format:
https://dhsprogram.com/what-we-do/survey/survey-display-{0}.cfm
where {0} = the numeric survey ID as given in the CSV table.

This page is downloaded and the source is parsed to see if it is a DHS or MIS type survey by looking for a line like 
```
<td colspan="2"><div id="title_container"> <span class="h2alt1">Angola: MIS, 2011</span> </div></td>
```
or 
```
<td colspan="2"><div id="title_container"> <span class="h2alt1">Angola: Standard DHS, 2015</span> </div></td>
```
If this is matched, the page is also parsed to extract the page linked to for downloading that survey’s data (the “Data Available” link), by looking for a line like
```
<a href="/data/dataset/Angola_MIS_2011.cfm?flag=1"><strong>Data Available</strong></a>
```

The data page is then loaded and parsed in turn to find the name of the individual recode hierarchical data file – this is a file with a name that matches the regular expression 
```
..IR..\.ZIP
```

We also look for a shapefile data link which matches 
```
..GE….\.ZIP
```

This would give the download URL for the data file but it didn’t work directly at first attempt. I did not investigate but I believe that this link needs the ID of the session cookie appending, and then it redirects to the actual download URL for the zip file. In FME this didn’t seem to work, although it might well be possible. 
Instead of messing about with this, I build the direct download URL directly by observing the requests made by a browser when loading one of these URLs. These are much simpler! In either case it’s a simple structure that only needs the filename substituting in.
The zip files are then downloaded into a subdirectory with the name of the survey ID, such as 379.

A simple python script is provided in an ipython notebook to unzip all of these zip files and rename the files within to prepend the survey id number.

