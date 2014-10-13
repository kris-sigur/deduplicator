## DeDuplicator Manual

The following applies to the (as yet unreleased) DeDuplicator version 3.1.0 for Heritrix 3.3.0 (also not yet 
released as stable). For older versions, see [here](started-old.html).

### Requirements

 * **Operating system:** Linux (using bash command shell).  
   It is possible to run under other configurations, but those are not directly supported and will require
   advanced configuration.
 * **Heritrix:** Version 3.3.0 (still unreleased, use a build made after July 2014).
 * **Java:** Java 6 (or better). This matches Heritrix. 

### Install the DeDuplicator

A pre-built download will be made available soonish. For now, you need to download the source from Github 
(https://github.com/Landsbokasafn/deduplicator) either via Git clone, or by downloading it as a 
[ZIP archive](https://github.com/Landsbokasafn/deduplicator.git).

Once downloaded, use Maven 3 to build it using the `package` goal. This will cause the distributables to be 
created under `deduplicator-dist/target/`.

Explode the file named `deduplicator-<version>-SNAPSHOT-<buildnumber>-dist.tar` into a suitable 
directory. This creates the `deduplicator` directory. We'll refer to that location as the installation directory.

Under the installation directory you'll find the `bin` directory that contains the shell script to run the indexing
part of the DeDuplicator.

You'll also find a `heritrix` folder that we'll get to later.

### Understanding the index

The DeDuplicator relies on a pre-built Lucene index. This is consulted during crawl time to determine if a resource
can be filtered out as a duplicate/revisit.

At minimum the index must contain:

 * The content digest - This must always be indexed (i.e. searchable)
 * The original URL
 * The original time of capture
 
The URL and date in the index are used to set the `WARC-Refers-To-Target-URI` and `WARC-Refers-To-Date`
WARC headers, respectively, as specified in the IIPC's adopted [Proposal for Standardizing the Recording of Arbitrary Duplicates in WARC Files](https://iipc.github.io/warc-specifications/specifications/warc-deduplication/recording-arbitrary-duplicates.html).

This is the most fundamental deduplication, that pays no heed to the relation between the current URL and the 
URL of the original capture. As long as the digests are the same, that is enough.

It is also possible to have the URL field indexed. If this is done, it is possible to either require that the URL 
match, or simply prefer URL matches when possible. This will be discussed more later in search strategies.

### Run the indexer

Under the installation directory, you'll find `conf/deduplicator.properties`. This file contains default
configuration options. Most can be amended via command line parameters. The default properties file contains the
most up-to-date explanation of the options and those comments should be consulted when making changes.

To run the indexing process, first (from the installation directory) run:

```
$ ./bin/index --help
``` 
 
This will print the available options.

```
Usage: DigestIndexer [options] source target
Options:
 -a,--add                   Add source data to existing index.
 -e,--etag                  Include etags in the index (if available in the
                            source).
 -h,--help                  Prints this message and exits.
 -i,--iterator <classname>  An iterator suitable for the source data (default
                            iterator works WARC files).
 -m,--mime <reg.expr.>      A filter on what mime types are added into the index
                            (blacklist). Default: ^text/.*
 -s,--no-canonicalized      Do not add a canonicalized version of the URL to the
                            index.
 -u,--no-url-index          Do not index the URLs. Index will only be searchable
                            by digest.
 -v,--verbose               Make the program print progress info to standard
                            out.
 -w,--whitelist             Make the --mime filter a whitelist instead of
                            blacklist.
Arguments:
 source                     Data to iterate over (typically a directory containing
                            WARC files). If using a non-standard iterator, consult
                            relevant documentation
 target                     Target directory for index output. Directory need not
                            exist, but unless --add should be empty.
```
 
#### Filter input

It is possible to filter what URLs are added to the index based on their mimetype (content type). This is done
via the `--mime` option. This is a blacklist by default, but can be treated as a whitelist by using the `--whitelist` option.

The default is to exclude `^text/.*`. I.e. anything whose mimetype begins with `text`. The rationale being that
those resources are very likely to be dynamically created, be small and compress well. Thus being of low value
for deduplication. 

Indexing all URLs is simply a matter of setting `--mime .*` and `--whitelist`.
 
Currently, only those URLs that are successfully crawled (i.e. have an HTTP status code of 200) will be included, regardless of mimetype.

#### CrawlDataIterator

Data is fed to the indexing process by a `CrawlDataIterator`. This is configured via the `--iterator` (fully qualified classname) or, more easily via the `deduplicator.properties` file.

The DeDuplicator ships with two such iterators.

 1. **WarcIterator**  
 Iterates over all response and revisit records in all WARC files within the directory specified by `source` (the argument given to the index script), including sub-directories.  
 Note that revisit records can only be indexed if they contain the `WARC-Refers-To-Target-URI` and `WARC-Refers-To-Date` headers. Otherwise, they'll be counted as 'unresolved' and omitted from the index.  
 This is the default option.
 2. **CrawlLogIterator**  
 Iterates over all the URLs in a Heritrix `crawl.log`. The `source` argument, here, refers to the crawl.log file.  
 As with the WarcIterator, the original URL and time of capture must be in the crawl.log for duplicates to be indexed.  
 If the crawl.log is the output of a crawl using the DeDuplicator's default profile (see further on), it will contain
 suitable annotations and JSON 'extra information'.  
 Otherwise, it may be necessary to adjust the `deduplicator.crawllogiterator.revisit-annotation-regex` property in the properties to ensure that duplicate records do not get indexed as original captures.

#### Building the index

The only thing left, now, is to decide on the structure of the index.

The digest must be indexed and the original capture time will always be included, but not indexed.

The URL must be included and is indexed by default, but you can opt to not index it via `--no-url-index`. This limits your search strategies to `DIGEST_ANY`.

If the URL is indexed, a canoncalized form of the URL can also be included (uses AggressiveUrlCanonicalizer from OpenWayback). This enables certain search strategies. This can be suppressed via `--no-canonicalized`.

Lastly, you can use the `--add` option if you wish to add to an already existing index. Care should be taken not to mix indexes with different options regarding `--no-url-index` and `--no-canonicalized`.

Assuming that URLs are indexed, any URL+Digest match will be replaced in the index if it occurs again. If only digest is
indexed, then a new occurrence of the digest will replace previous ones in the index.

## Heritrix module

TODO: Installing in Heritrix

### Configuring Heritrix

You'll find that a new profile job is available in your Heritrix install. Called `profile-deduplicator`.

If you base your job on it, you only need to edit one thing (in addition to the usual things you'd change) in the properties override near the top.

```
deduplicatorIndex.indexLocation=ENTER_THE_LOCATION_OF_YOUR_INDEX_HERE
```

This is simply a fully qualified path to where you created the index.

You can now run the crawl (assuming you've set the seeds and changed the `operatorContactUrl` and made any other general changes to the configuration that your crawl may require).

This profile is based on the one that ships with Heritrix 3.3.0. To see exactly what has been modified, you could diff the two. Most of the changes are, however, a bit down the file where it says "DeDuplicator module defined". There are also a few tweaks here and there, such as enabling extra info in crawl logs (see further) and surfacing the DeDuplicator report.

#### Search Strategies

You may have noticed another setting, right after the index location.

```
deduplicatorIndex.searchStrategy=DIGEST_ANY
```

There are three available search strategies.

 1. **DIGEST_ANY**  
 Any record with an identical content digest to the current URL results in the current URL being deemed a duplicate.  
 If (and only if) the URL field is indexed, searches are performed in such a manner that exact URL matches will should be found when they are available in the index (will also use canonical form if available). This is not necessary, from a technical standpoint, but may make things a little clearer, especially as you transition to digest based deduplication.  
 If this behavior is not desired, do not index the URL field.
 2. **EXACT_URL**  
 The URL of the record in the index must match the current URL exactly (and the digest must, of course, also match) for there to be a duplicate/revisit verdict. This is similar to how things have been done in the past.  
 URL field must have been indexed.
 3. **CANONICAL_URL**  
 Similar to EXACT_URL, except on the canonical forms of the URL.   
  URL must be indexed and canonical URL must have been included in the index.

As you can see, DIGEST_ANY is, by itself, sufficient. The other are provided for backwards compatibilty. For example, in cases where the presentation layer does not (yet, hopefully) support URL-agnostic deduplication. 

#### Crawl.log Extra Info


#### The DeDuplicator report 

  