## The DeDuplicator (Heritrix add-on module)

### Release information

Current stable release is [3.0.0](./release3.html#3.0.0).

All releases, including interim (potentially unstable) releases can be found here: [Release History of DeDuplicator for Heritrix 1](./release.html) and here: [Release History of DeDuplicator for Heritrix 3](./release3.html)
 
### News

#### Heritrix changes and so too must the DeDuplicator 21/07/2014

Today a big change was merged into Heritrix master that substantially alters how 3rd party tools (like this one) interact with Heritrix and its notion of duplicates/revisits.

This change is entirely beneficial and will allow the next version of the DeDuplicator to properly implement duplicate detection between different URLs.

Consequently, we are now releasing 3.0.0 of the DeDuplicator. This works with Heritrix 3.0.0-3.2.0. Future versions will be built using the new revisit API in Heritrix and will not work with Heritrix 3.2.0 or older.
 
For those using SNAPSHOT builds of Heritrix, the actual API change was introduced on 21.07.2014 so you'll need the DeDuplicator 3.0.0 for builds made before then. Builds made after 21.07.2014 will require the upcoming 3.1.0 version. 


#### DeDuplicator for Heritrix 3 source now on GitHub - 29/05/2012

The code for the Heritrix 3 Deduplicator can now be found on [GitHub](https://github.com/Landsbokasafn/deduplicator).
 
No further development will be made on the version for Heritrix 1, but the source code for it and an older revision of the one for H3 can still be found in the CVS on SourceForge. 
 
Version 3.0.0-SNAPSHOT builds are now available [here](./release3.html). Updated for Heritrix 3.1. 


#### DeDuplicator for Heritrix 3 - 23/07/2010

Version 3.0.0-SNAPSHOT-20100727 is now available [here](./release3.html). 

This version is compiled against Heritrix 3.0.0.

It also updates to use Lucene 3.0.2 (from 2.0.0). Please note that changes in the Lucene library mean that memory usage will be approximately 40% greater than before. Memory usage appears to be approximately 5 bytes per URL in index, as compared to 3.6 bytes per URL previously. Query times have however improved significantly and are now fixed time without regard for the index size. For large indexes this can mean as much as 10-30 times shorter query times. Building indexes is also much faster (approximately 3-4 times as fast).

Currently the DeDupFetchHTTP processor has not been converted. 

This release heralds the end of the existing DeDuplicator, built against Heritrix 1.14. One final release (1.0.0) will be released soon with some accumulated bugfixes. A release candidate is available [here](release.html).


#### Version 0.4.0 released / Future plans - 15/07/2008

Version 0.4.0 includes numerous tweaks and patches introduced since 0.2.0.
 
Notable changes:
 
 * Support for changed crawl.log format that Heritrix introduced in 1.12.0.
 * Improved memory usage for large indexes.
 * Can now exclude duplicate URIs from new index.
 * Various bug fixes.  

This will be the last version of the DeDuplicator that is built against Heritrix 1.10.0. Building against that version of Heritrix has made the DeDuplicator compatible with almost all 1.x versions of Heritrix. Note though that 0.4.0 is built with Java 1.5, unlike 0.2.0 which was built with Java 1.4.2.
 
In version 1.12.0 Heritrix added some useful features that the DeDuplicator should make use of, most notably marking content as 'not novel'  (i.e. duplicate). Also in 1.14.0 there is rudimentary WARC support and the aim is to have the DeDuplicator support writing to WARC files. Therefor, any future versions will be built against Heritrix 1.14.0.  
 
Support for Heritrix 2.0 is planned but there is no set timeframe for it. This requires considerable changes to the DeDuplicator and will likely not be implemented until Heritrix 2.x is sufficiently mature that it is used routinely instead of 1.x for large scale production crawls.
 

#### Support for Heritrix 1.12.0 - 1/06/2007

A new interim release has been uploaded to deal with the changed crawl.log format in Heritrix 1.12.0. 0.4.0 will be the final release for Heritrix up to version 1.12.0 and should be released soon. 
 
Heritrix version 2.0.0, currently in development, will greatly change Heritrix's API and so will require significant changes to the DeDuplicator. Look for the first interim release built against the new Heritrix API as soon as the changes are moved into the trunk of the Heritrix project. Probably sometime this month.


#### Moved to Sourceforge.net - 7/11/2006

The project has now been moved to Sourceforge.net. The code has been moved to SF's CVS and anonymous access is now possible. Initial commit was of version 0.2.0. Change history prior to 0.2.0 will be discarded except that we will keep the packaged PreRelease versions that were made.
 
Along with the public CVS, SourceForge also provides [bug and RFE trackers](http://sourceforge.net/tracker/?group_id=181565). The project website has also been moved to http://deduplicator.sourceforge.net/.
 
Stable releases will now be distributed via SourceForge while iterim builds will continue to be made available on the [Release History](release.html) page (until continous integration is set up).

#### Managing duplicates across sequential crawls - 31/10/2006
 
On September 21, Kristinn Sigurðsson presented a paper on the DeDuplicator titled 'Managing duplicates across sequential crawls' at the [6th International Web Archiving Workshop](http://iwaw.net/06) held in conjunction with the [10th ECDL](http://ecdl2006.org) in Alicante, Spain.
 
The paper is available in the [Workshop Procedings](http://www.iwaw.net/06/PDF/iwaw06-proceedings.pdf) and can also be downloaded by itself directly from here: [Managing duplicates across sequential crawls](http://hdl.handle.net/1946/6074).