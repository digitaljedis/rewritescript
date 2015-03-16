# rewritescript
Rewrite Script for Wordpress Permalinks when using Namesco hosting make sure to save as rewrite.script and place in root directory.
RULE_0_START:
# get the document root
map path into SCRATCH:DOCROOT from /
# initialize our variables
set SCRATCH:ORIG_URL = %{URL}
set SCRATCH:REQUEST_URI = %{URL}

# see if theres any queries in our URL
match URL into $ with ^(.*)\?(.*)$
if matched then
  set SCRATCH:REQUEST_URI = $1
  set SCRATCH:QUERY_STRING = $2
endif
RULE_0_END:

RULE_1_START:
# prepare to search for file, rewrite if its not found
set SCRATCH:REQUEST_FILENAME = %{SCRATCH:DOCROOT}
set SCRATCH:REQUEST_FILENAME . %{SCRATCH:REQUEST_URI}

# check to see if the file requested is an actual file or
# a directory with possibly an index.  don't rewrite if so
look for file at %{SCRATCH:REQUEST_FILENAME}
if not exists then
  look for dir at %{SCRATCH:REQUEST_FILENAME}
  if not exists then
    set URL = /index.php?q=%{SCRATCH:REQUEST_URI}
    goto QSA_RULE_START
  endif
endif

# if we made it here then its a file or dir and no rewrite
goto END
RULE_1_END:

QSA_RULE_START:
# append the query string if there was one originally
# the same as [QSA,L] for apache
match SCRATCH:ORIG_URL into % with \?(.*)$
if matched then
  set URL = %{URL}&%{SCRATCH:QUERY_STRING}
endif
goto END
QSA_RULE_END:
