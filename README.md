# Archives Assemble: Florida
Analyzing the shape of assembled archival collections about early Florida

## Goals
This project is an attempt to use quantitative research methods to explore and assess how archival materials were valued, assembled, and disseminated in the twentieth century, and how those practices influenced historical writing about the U. S. Gulf South. 

### Territorial Papers of the United States
*The Territorial Papers of the United States* is a 28-volume set of primary source documents published between 1934 and 1975. The project assembled historical documents from the United States Department of State and the National Archives dealing mostly with the administration of territories annexed by the United States that would later become states in the union. Historian [Clarence Edwin Carter](https://www.wikidata.org/wiki/Q26215248) compiled and edited the first 26 volumes until his death in 1961. (Aeschbacher 1962:404) The last two volumes were edited by John Porter Bloom. 

Print volumes of *The Territorial Papers of the United States* are commonly found in libraries throughout the United States. Digital scans are also available in the [HathiTrust](https://catalog.hathitrust.org/Record/000495370) repository. As U. S. government publications, these volumes are in the public domain. HathiTrust provides plain text files of each volume for download.

As a case study for the broader project goals, the current project will focus on volumes 22-26 in the series, the five volumes dedicated to the territorial period of Florida, 1821-1845.

## Progress
- Spring 2021: Cleaning and markup of plain text files for volumes 22-26 in progress. A detailed ``Readme.md`` file on this process is in the TPUS folder along with intermediary files created along the way.

## Data Dictionary
Document elements are enclosed in opening and closing tags. For example, `<place-time>`Tallahassee, Florida 24 February 1822`</place-time>`

### Document tags
- ``<document>``
- ``<dochead>`` Document Header
- ``<docbody>`` Document Body
- ``<place-time>`` Location and Date of Document
	- ``<location>`` Geographic origin of document
	- ``<year>`` Year document written
	- ``<month>`` Month document written
	- ``<date>`` Date document written. Where only a year, or year and month, given, marked as ``No date``
- ``<source>`` Archives Location of Original Document
- ``<enclosure>`` Denotes a document enclosed within another document, usually correspondence.
	- ``<enclnote>`` Identifies an Enclosure, often including a note on signatures 
	- ``<enclhead>`` Heading for the enclosed document
	- ``<encldate>`` Date of the enclosed document
	- ``<enclbody>`` Body of the enclosed document


===   
### Works Cited
Aeschbacher, William D. "Report of the Secretary-Treasurer for the Year 1961-1962." *The Mississippi Valley Historical Review* 49, no. 2 (September 1962): 404.