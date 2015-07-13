# eVA - Procurements for State of Virginia, FY2001-2015
## what does VA spend its money on?
## who spends how much on what?

Cleaned up from 15 messy tables.  
Was able to salvage >14 million records out of <18 million.  
Problem with guessing count of records is tricky embedded newlines and occaisonal improper quoting.

Source tables from eVA [here](https://eva.virginia.gov/pages/eva-opendatasets.htm).  
NB: FY2015 _shouldn't_ be partial as data were collected 2015-07-12, and VA's fiscal calendar ends June 30 but I have not confirmed.


### method

None of these excellent tools could quit get the parsing job right, possibly the fault of the records themselves:

* [csvquote](https://github.com/dbro/csvquote)
* [csvclean](https://github.com/onyxfish/csvkit)
* Gnumeric's ssconvert
* Libreoffice --headless
  * I got desperate!

Method ultimately employed to identify acceptable records was to
 
1. check for right number of columns
1. of that, get a list of unique IDs
1. for the list of IDs, pull out any that look suspicious
  1. ie, not just capital letters, numbers, and special characters
1. do a little manual curation of this list
  1. just a few hundred entries - we got lucky!
1. from the suspicious ID field entries (eg "HP Laserjet . . ."), 
  1. get the record number they occur at with "grep -n"
  1. pull out those records by number with sed
  1. match the inserve of the unwanted records and write back to file
    1. this is a low memory approach that works well when scanning plain text a multiple the size of your RAM


Looks like this

```bash
# get just records with 19 fields
gunzip -c *.gz | tawk '{if(NF == 19)print $0}' > foo
# get unique IDs that are all jacked up
cat foo | cut -f1 | sort | uniq | grep -vE "^[A-Z0-9 \"_/:-]+$" > rm_if_ID.txt
# get line numbers of records with those IDs
grep -nFf rm_if_ID.txt <(cat foo | cut -f1 ) | grep -oE "^[^:]*" > records_to_rmLine.txt
# get the actual record text
cat records_to_rmLine.txt | xargs -I '{}' sed -n "{}p" foo > records_to_rm.txt
# remove that record text
grep -vFf records_to_rm.txt foo > healthyRecords.tsv
```
