Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

There were two different wikidata items for this position, so I had to
merge those first, and then manually switched the P39s from the
now-redirected item to the primary.

Step 2: Tracking page
=====================

PositionHolderHistory already exists — starting version is
https://www.wikidata.org/w/index.php?title=Talk:Q16323054&oldid=1256322079
with 9 dated memberships, 0 undated; and 6 warnigs.

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit [add_P39.js script](add_P39.js)
to configure the Item ID and source URL.

Step 4: Get local copy of Wikidata information
==============================================

```sh
wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
  xargs wd sparql existing-p39s.js | tee wikidata.json
```

Step 5: Scrape
==============

```sh
wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
  xargs bundle exec ruby scraper.rb | tee wikipedia.csv
```

Step 6: Create missing P39s
===========================

```sh
bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
  wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"
```

15 additions made as https://tools.wmflabs.org/editgroups/b/wikibase-cli/ec9a5834134fa

Step 7: Add missing qualifiers
==============================

```sh
bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
  wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"
```

Nothing to do

Step 8: Refresh the Tracking Page
==================================

New version at
https://www.wikidata.org/w/index.php?title=Talk:Q16323054&oldid=1257581598
