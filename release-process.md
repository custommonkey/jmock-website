---
layout: page
---
The jMock Release Process
=========================

Releasing is pretty automated but there are still some manual steps.

1.  Tag a release using the `tag.sh` script. Tags have the format 1.2.3-RC4 where, in this case, 1 is the major version, 2 is the minor version and 3 is the patch number and 4 is the release candidate number. The trailing \_RC4 is optional.
2.  Run the release script: ./release.sh 1.2.3-RC4. Again the "-RC4" is optional. That will publish the zips and javadoc. Zips are published under `http://www.jmock.org/dist`, javadoc under `http://www.jmock.org/javadoc/`.
3.  Update the website. This is currently entirely manual and needs to be more automatic.
    1.  Check out the `website` module from SVN and cd into the checked out directory.
    2.  Run the `version.py` script to update the version numbers.
    3.  Update the `content/download.html` document to link to the released JARs
    4.  Update the `content/javadoc.html` document to link to the released Javadoc.
    5.  Update the `content/news-rss2.xml` file to announce the new release.
    6.  Run make to build a local copy. Check that it all looks ok by opening `skinned/index.html` and browsing the download and javadoc pages.
    7.  Run make published to publish the updated website.
    8.  Commit the website changes to SVN.
    9.  Add the new version to the list of jMock versions in JIRA (if it doesn't already exist) and mark it as released on the current date.
