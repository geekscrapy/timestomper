# Timestomper

This app converts timestamps in text files, line-by-line. It was made to combat the non-standardised date/time fields tools output. Pull requests welcome!

Give it an input file (\-\-infile) and the format type of dates and/or times in that file (\-\-search), then an output format (\-\-replace).

If it is a **completely** freeform input file, it is not guaranteed every line will contain a date/time value. If these lines want to still be retained, use the \-\-include argument. If they do not want to be included in the output use \-\-ignore.

If there is the possibility of the wrong timestamps being converted (or multiple timestamps on a line), the \-\-cut or \-\-index options may be used.

\-\-cut - allows the matching engine to be applied to the section of the line between the start and end values (first character starts at 0).
\-\-index - If there are multiple matches, the index can be set to only replace that one particular timestamp. If \-\-index is not given all matched timestamps on the line are replaced. N.B. Matches are ordered based on their indices of the regex match - not usually a problem unless the input can have overlapping timestamps (and bad regex matches...).


## \-\-help
```
  usage: timestomp.py [-h] [--formats] [-i file.txt [file.txt ...]]
                      [-o file.txt]
                      [-s us-hyphen, osx-ls, us-slash-no_secs OR "%Y-%m-%d %H:%M"]
                      [-r "%Y-%m-%d %H:%M"] [-y 1997] [-c #-#] [--index #]
                      [--include] [--ignore] [--highlight] [-v]

  optional arguments:
    -h, --help            show this help message and exit
    --formats             Print the preloaded search formats
    -i file.txt [file.txt ...], --infile file.txt [file.txt ...]
                          File to parse. - is stdin
    -o file.txt, --outfile file.txt
                          Output changed lines to this file. Without or -,
                          results are printed to stdout
    -s us-hyphen, osx-ls, us-slash-no_secs OR "%Y-%m-%d %H:%M", --search us-hyphen, osx-ls, us-slash-no_secs OR "%Y-%m-%d %H:%M"
                          Type of date/time strftime format that will be found
                          in the file - you can get a list of available searches
                          using: timestomp.py --formats
    -r "%Y-%m-%d %H:%M", --replace "%Y-%m-%d %H:%M"
                          Translate the found date/time to this strptime format
                          - you can get a list of available formats using:
                          timestomp.py formats --search
    -y 1997, --year 1997  Some dates dont contain the year. Set those dates with
                          this flag
    -c #-#, --cut #-#     Start and end position in lines to look for timestamps
                          - same as Linux cut syntax (10-15, -10, 10- etc.) cut
                          operation is performed before index evaluation
    --index #             Preferred timestamp to convert should there be more
                          than one match. If there is more than one match and
                          index is not specified, all matches on a line are
                          replaced
    --include             Include non-matching lines with output - helps with
                          free-form text files. If used with --ignore, --ignore
                          is, ignored :)
    --ignore              Ignore non-critical errors. If --include is not
                          specified, lines which would normal generate an error
                          are ommited from output
    --highlight           Highlight the text being changed
    -v, --verbose         Print debug information to stderr
```

## formats\.py
This file contains a list of common dates and times and their regex.

#### searches

If we consider the following entry:

    'win-dir-uk': [
      {
        'regex': r'(\d{2}\/\d{2}\/\d{4}\s\s\d{2}\:\d{2})',
        'strftime': '%d/%m/%Y  %H:%M'
      }
    ],

This entry allows us to dissect the Windows dir command output in UK formatted time. The regex field allows the script to recognise a date in the line, and the strftime parser is then run over that regex capture to translate to a python datetime object.
If it is possible to have two types of date/time formats in a file then additional formats need to be used:

    'osx-ls': [
      {
          'regex': r'(\d\d?\s(?:Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec)\s+\d\d\:\d\d)',
          'strftime': '%d %b %H:%M'
      },
      {
          'regex': r'(\d\d?\s(?:Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec)\s+\d{4})',
          'strftime': '%d %b %Y'
      }
    ],
The output of OSX `ls` command likes to exclude the year data if that file/directory has been modified in the present year. For that reason, two parsers need to be used. The parsers will be matched in the order they are given, but matches are ordered by the match startpos value.

In the case of a missing year, the flag --year must be used to define the year to be output. Otherwise, --ignore can be used to set the year as the present.

## example_import\.py

This file gives examples to how to utilise the functions in another script. Explanations embedded in the file

## Aliases

Add the following to your.bashrc (etc.) to limit typing!

    alias us2dfir='timestomp.py generic --infile - --outfile - -s us-slash --ignore --include'
