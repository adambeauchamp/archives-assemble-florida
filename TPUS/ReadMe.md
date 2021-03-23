# Text Cleaning

Started with fresh copy of Territorial-Papers_v22....txt file. 

*Note: these instructions also followed to clean other volumes in TPUS. Where changes were made to accommodate unique features in another volume, they are noted after the initial instructions.*

**TPUSv23**: Discovered numerous poorly scanned pages from the University of Minnesota scan of this volume, resulting in corrupted plain text files. Replaced with University of California scan, but this resulted in several instances where established regular expressions did not capture the OCR idiosyncracies of this volume. Alternative steps are noted where appropriate.

## Steps
1. Identify all PDF HathiTrust page number lines in format: \## p. (#1) ##################################################

	FIND ``^.*\((#\d+)\).*$``
	REPLACE ``\<pb\1/>``
	RESULT ``<pb#1/>`` for page 1, etc.

1. Manually copied and pasted frontmatter (pp. 1-22) into a separate .txt file (e.g. ``TPUSv22_frontmatter.txt``).

1. Find and delete internal page headings (separate from PDF HathiTrust page numbers) with headers like: 8 T E R R IT OR I. A L PA P E R S and F L OR I D A T E R R IT OR Y 7

	FIND ``^.*T E R.*$``
	REPLACE null

1. Find and delete all line breaks that do not separate textual elements (i.e. mid-paragraph line breaks). Retain line breaks after document titles, repository citation, date, author.

	FIND ``([a-z]) ?\n(?=[a-z])``
	REPLACE ``\1·``   (note the space, here shown as “·” after the \1 to create a space between words where the line break used to be)

	FIND ``([a-z]) ?\n(?=[A-Z])``
	REPLACE ``\1·``

	And for hyphenations across line breaks:
	FIND ``([a-z])- ?\n(?=[a-z])``
	REPLACE ``\1``  (note there is no space after the \1 since we are rejoining whole words)

	And for line breaks after commas:
	FIND ``([a-z],) ?\n(?=.)``
	REPLACE ``\1·``

	More line breaks after commas:
	FIND ``(.,)\n``
	REPLACE ``\1·``

1. Delete internal chapter titles (Part one, Part two, etc.)
	FIND ``^PART.*$``
	REPLACE null

1. Manually copied and pasted index (backmatter) into a separate .txt file (e.g. ``TPUSv22_index.txt``).

1. Make all footnotes single lines of text.
	FIND ``^((\*|^"|^\d{1,3}\s).*) ?\n(?!(\*|"|\d{1,3}\s))``
	REPLACE ``\1·``  (note space after \1)
	
	*Translation: any line that begins with either * or “ or a 1-3 digit number followed by a space, capture all text until the line break, if that line break is not followed by a * or “ or a 1-3 digit number followed by a space. Replace with captured text + space.*

1. Delete empy lines
	FIND ``^\W$``
	REPLACE null

1. Delete hexadecmial character <0x0c>
	FIND ``\x0c``
	REPLACE null

1. Save to a new file: (e.g. ``TPUSv22_documents.txt``).
	
*We will now delete the footnotes from the documents file, and the documents from the footnotes file.*


### On file ``TPUSv22_documents.txt`` (and ``_documents.txt`` for other volumes)
#### Identify, copy, and remove all footnotes
Still finding footnotes split over more than one line.
1. Get rid of trailing spaces after full stops.
	FIND ``((^\*|^").*\.) $``
	REPLACE ``\1``

1. Find any line that begins with either ``*`` or ``"``, which is how OCR catches footnote suprascript, and the line break IS NOT followed by a ``*``, ``<``, or ``\n``. This would exclude subsequent footnotes, page breaks, or blank lines, respectively. In other words, the footnote continues as either a new sentence, paragraph, or continues a sentence where an abbreviation is mistaken for a full stop.
	FIND ``(^\*|^")(.*)?\n(?!(\*|<|\n))``
	REPLACE ``\1\2·``
	REPEAT this find replace pattern until it no longer matches anything, thereby deleting any line breaks that remain within a footnote.

	*Incidentally, this revealed three instances whre document text was preceded by an asterisk. I corrected these by removing the erroneous asterisk manually.*
1. Delete all blank lines.
	FIND ``^(?:[\t ]*(?:\r?\n|\r))+``
	REPLACE null
	*Found this regex at <https://gist.github.com/fomightez/706c1934a07c08b6c441>.*

1. Find all footnotes. 
	FIND ``^((\d{1,3}\s|^\*|^").*\n)+(<pb#\d{1,3}/>)``

	*This locates all lines that begin with either ``*`` or ``"`` or 1-3 digit number followed by a space, and end with a full stop and line break, repeating until finding the page break notation since footnotes can start anywhere on the page but always end at the bottom of a page.*
	
	COPY found text and PASTE into new file named (e.g. ``TPUSv22_footnotes.txt``).

	*This will retain the page breaks with the copied footnotes.*
	REPLACE ``\3``

	*This deletes the footnotes while retaining the page breaks.*

	**TPUSv23** has better OCR on the footnotes, resulting in numbered footnotes rather than misidentified ``*`` or ``"``.
	FIND ``^(\d{1,3}\s.*\.\n)+(<pb#\d{1,3}>)``
	REPLACE ``\2``
	*This deleted the 140 footnotes. They can be retrieved from original ``.txt`` file if needed for later use.*


#### Tag all elements in each document
1. Identify the archival source, pattern [ABBREV. ….].
	FIND ``^(\[(NA|LC).*\])``
	REPLACE ``<source>\1</source>``

	FIND ``(\[|\(|\|)(NA|LC)(.*)(\]|\)|\|)``
	REPLACE ``<source>[\2\3]</source>``
	*Instead of replace all, replaced one at a time since there were not many instance of these mis-OCR’ed brackets, but there are instances where “(NA” means something else.*
	**TPUSv23** had numerous instances of open squarebrack but closed parenthesis around the source data (e.g. ``[NA:GLO, Lets from SG, Fla. :ALS)``. Used FIND ``^(\[(NA|LC).*\])`` and REPLACE ``<source>\1</source>``. Not a significant problem for v24, v25, or v26. Also discovered several instances of footnotes not captured by previous work, but discovered here through their references to NA or LC repositories. Corrected by hand.

1. Identify place/date of document, which follows the source notation.
	FIND ``(</source>\n)(.+\d{4}.*)$``
	REPLACE ``\1<place-time>\2</place-time>``

1. Identify document headers, which are in all CAPS and precede the source note.
	FIND ``^([A-Z])(.*)(\n)(?=<source>)``
	REPLACE ``<dochead>\1\2</dochead>\n``

1. Identity the two-line document headers.
	FIND ``^(([A-Z]){2,999}\s)(.*)([A-Z])(\n)(?=<dochead>)``
	REPLACE ``<dochead>\1\3\4</dochead>\n``
	*The {2,999} avoids capturing sentences that begin with the word “I”, as in the signature: “I am JOHN QUICY ADAMS.”*

1. Merge multi-line docheads.
	FIND ``(</dochead>)\n(<dochead>)``
	REPLACE *space*

1. Remove digits from docheads, which seem to always represent a footnote.
	FIND ``(<dochead>)([^\d]*)\d+([^\d].*)(</dochead>)``
	REPLACE ``\1\2\3\4``

	FIND ``(<dochead>)([^\d]*)\s\d+(</dochead>)``
	REPLACE ``\1\2\3``
	*This catches instances where the footnote is at the end of the dochead.*

1. Locate any texted marked as ``<dochead>`` in error. Docheads should be in all CAPS.
	FIND ``(<dochead>)(.+?[a-z]+.*?)(([A-Z]+\s)+)([A-Z]+)(</dochead>)``
	REPLACE ``\2\n\1\3\5\6``

1. Locate any stray punctuation (OCR footnotes) in dochead.
	FIND ``<dochead>.*[[:punct:]]</dochead>``
	Manually remove erroneous punctuation. In process discovered a few mis-tagged dochead. Corrected manually.

1. Tag all non-metadata aspects of each document.
	FIND ``(<dochead>.*</dochead>\n<source>.*</source>\n<place-time>.*</place-time>\n)``
	REPLACE ``</docbody>\n\1<docbody>``

*Extra cleanup needs discovered during preceeding operations*
1. Remove left over page headings in format: “[page number] [\n or \s] Territorial Papers” in varying patterns of capital letters, periods, and spaces.
	FIND ``^\d+(\s|\n)T(\.|\s|\.\s)E(\.|\s|\.\s)R(\.|\s|\.\s)R.*$``
	REPLACE *null*
	FIND ``^.*?T(\.|\s|\.\s)E(\.|\s|\.\s)R(\.|\s|\.\s)R.*$``
	REPLACE *null*

#### Prepare to create a table in OpenRefine; Create a well-formed XML file.
1. Create new file from TPUSv22_documents.txt, named TPUSv22_documents_nopagebreaks.txt.
1. Remove all page break notations.
	FIND ``<pb#\d+/>``
	REPLACE *null*
1. Remove all line breaks within tagged elements, so that each element becomes a single line.
	FIND ``(?<!>)\n``
	REPLACE *space*
1. Find any line that does not begin with a tag. Manually insert appropriate tags and line breaks at each error.
	FIND ``^[^<]``
	
	*Alternatively, for **v23** though doesn't capture all instances:*
	FIND ``([^a-z]+)(<source>.+?</source>\n)(<place-time>.+?</place-time>\n)([^<])``
	REPLACE ``</docbody>\n<dochead>\1</dochead>\n\2\3<docbody>\4``
	*Repeat with the following to discover similar pattern but with the ``<place-time>`` tags also missing:*
	FIND ``([^a-z]+)(<source>.+?</source>\n)([^<])``
	REPLACE ``</docbody>\n<dochead>\1</dochead>\n\2<place-time>\3``
	*Use ``<place-time>.+?</docbody>`` to locate these substitutions and insert ``</place-time>`` and ``<docbody>`` manually.
	*Then use ``.+<dochead>`` to locate any misaligned headers and correct manually, usually requiring addition of ``</docbody>`` and sometimes ``<place-time>`` tags.
1. Find any duplicate tags and correct.
	FIND ``<place-time><place-time>``
	REPLACE ``<place-time>``
1. Check to make sure there are equal numbers of each tag, opening and closing. Simple FIND of each tag to see total count. Make manual corrections in few instances where a tag is missing or misspelled.
	FIND ``^.+(<dochead>)``
	*Locates any mid-line ``<dochead>`` tags which can then be manually corrected.
1. Add XML statement to beginning of file: ``<?xml version="1.0" encoding="UTF-8"?>``
1. Create root tag for entire file. Add ``<text>`` to line 2, following XML statement, and add ``</text>`` to last line.
1. Enclose each document with ``<document>`` and ``</document>`` tags (each containing a set of four tagged elements).
	FIND ``(<dochead>)``
	REPLACE ``<document>\n\1``
	FIND ``(</docbody>)``
	REPLACE ``\1\n</document>``
1. The ampersand is an escape character in XML. 
	FIND ``&``
	REPLACE ``&amp;``
1. Save file as TPUS22_documents_nopagebreaks.xml.
	1. Opened file with Firefox to test for well-formed-ness. Found one error; a map had been transformed by OCR into a line of random punctuation, including a ``<``, which caused the XML parsing error.
	1. Deleted the offending characters.

### Tag date, month, year, and location within the ``<place-time>`` tags for more effetive manipulation and visualization.
1. Remove brackets within place-time and source tags
	FIND ``(<place-time>)\[``
	REPLACE ``\1``
	FIND ``\](</place-time>)``
	REPLACE ``\1``

	FIND ``(<source>)\[``
	REPLACE ``\1``
	FIND ``\](</source>)``
	REPLACE ``\1``

1. Separate places and dates within ``<place-time>`` tags.
	1. Tag years.
	FIND ``(18\d\d).*(</place-time>)``
	REPLACE ``<year>\1</year>\2``
	FIND ``[[:punct:]](\d\d).*(</place-time>)``
	REPLACE ``<year>18\1</year>\2``
1. Tag dates.
	FIND ``(\d{1,2})\s(?=<year>)``
	REPLACE ``<date>\1</date>``
	FIND ``(\d{1,2})[[:punct:]](?=<year>)``
	REPLACE ``<date>\1</date>``
	FIND ``(\d{1,2})[[:punct:]]\s(?=<year>)``
	REPLACE ``<date>\1</date>``
1. Tag months.
	FIND ``([JFMASOND][a-z]+)([[:punct:]]|\s)(?=<date>)``
	REPLACE ``<month>\1</month>``
	FIND ``([JFMASOND][a-z]+)([[:punct:]]|\s)(?=<year>)``
	REPLACE ``<month>\1</month>``
1. Tag dates. (2nd pass)
	FIND ``(\d{1,2})(th|[[:punct:]]|\s)(?=<month>)``
	REPLACE ``<date>\1</date>``
	FIND ``(\d{1,2})(th|[[:punct:]]|)\s(?=<month>)``
	REPLACE ``<date>\1</date>``
	FIND ``(\d{1,2})(th|[[:punct:]]|)\s(?=<year>)``
	REPLACE ``<date>\1</date>``
	FIND ``([JFMASOND][a-z]+)([[:punct:]]|\s)(?=<date>)``
	REPLACE ``<month>\1</month>``
	FIND ``([JFMASOND][a-z]+)([[:punct:]])\s(?=<year>)``
	REPLACE ``<month>\1</month>``
	FIND ``(<place-time>)(.+)([JFMASOND][a-z])([[:punct:]]|\s|[[:punct:]]\s)(<year>)``
	REPLACE ``\1\2<month>\3</month>\5``
	FIND ``(<place-time>.+)(\d\d)(st|nd|th)(\s|[[:punct:]]|[[:punct:]]\s)(<)``
	REPLACE ``\1<date>\2</date>\5``
	FIND ``(<place-time>.+)(\d)(st|nd|th)(\s|[[:punct:]]|[[:punct:]]\s)(<)``
	REPLACE ``\1<date>\2</date>\5``
	FIND ``(\d{1,2})(th|[[:punct:]]|\s)\s*(?=<month>)``
	REPLACE ``<date>\1</date>``
1. Tag locations.
	FIND ``^(<place-time>)([A-Z].+?)(<[dmy])``
	REPLACE ``\1<location>\2</location>\3``
1. Check for any dates captured within location tags.
	FIND ``(\d{1,2}).{1,3}(</location>)``
	REPLACE ``\2<date>\1</date>``
1. Remove trailing spaces in ``<location>`` tag.
	FIND ``\s(</location>)``
	REPLACE ``\1``

### Enclosures
1. Tag enclosures within the body of other documents.
	FIND ``(\[Enclosure.*)(</docbody>)``
	REPLACE ``<enclosure>\1</enclosure>\2``
1. Locate any misssed Enclosures
	FIND ``(?<!<enclosure>)\[Enclosure``
		*Identified 16 matches, instances of more than one Enclosure per document. Corrected by hand.*
1. Tag headers and dates within enclosures.
	FIND ``(<enclosure>)(\[E.*?\])(.*?)(\[[JFMASOND][a-z]*.*?\d{4}\])``
	REPLACE ``\1<encldata>\2</encldata><enclhead>\3</enclhead><encldate>\4</encldate><enclbody>``
1. Close ``<enclbody>`` tags.
	FIND ``(<enclbody>)(.*)(</enclosure>)``
	REPLACE ``\1\2</enclbody>\3``
1. Check for missed ``</enclbody>``
	FIND ``(?<!</enclbody>)(</enclosure>)``
	REPLACE ``</enclbody>\1``
1. Check for missed Enclosure heading tags. Date includes numeric date and year.
	FIND ``(<enclosure>)(\[E.*?\])(.*?)([JFMASOND][a-z]*\s\d+?.*?\d{4})``
	REPLACE ``\1<enclnote>\2</enclnote><enclhead>\3</enclhead><encldate>\4</encldate><enclbody>``
1. Check for missed Enclosure heading tags. Date does not include numeric date.
	FIND ``(<enclosure>)(\[E``
		*Identified 8 matches. Corrected by hand.*
1. Break enclosures onto new lines.
	FIND ``(<enclosure>)``
	REPLACE ``\n\1``
1. Break enclosure headings and enclosure bodies into separate lines.
	FIND ``(<enclnote>)``
	REPLACE ``\n\1``
	FIND ``(<enclhead>)``
	REPLACE ``\n\1``
	FIND ``(<enclbody>)``
	REPLACE ``\n\1``
