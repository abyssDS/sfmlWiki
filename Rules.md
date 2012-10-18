# General guidelines for editing the wiki

## Main rules

This wiki is maintained by the SFML community, and as a consequence you are allowed (and encouraged) to edit or add any relevant contribution. However, it is recommended to follow the general organisation and page layout, in order to keep a consistent and pleasant wiki for all. In other words, don't put your source code under the tutorials section.

There's no restriction on the whole wiki for creating or editing pages; you can even create new categories for subjects that don't exist yet, if you think it's worth sharing.

You are also free to edit pages created by other users, if you have relevant additions or corrections to make.

## Contents

You are encouraged to share anything related to SFML with no restriction: tutorials, source codes, links, tips and tricks, personal projects, … as long as you find it relevant.

English is the main language of this wiki. However, you can provide translations of certain pages if you want. But keep in mind that the english version has to be the default/main one, if other languages are provided they must only appear as translations of the corresponding english page. Never make wiki links directly point to non-english pages.

When you edit content, please leave a comment in the dedicated text field at the bottom of the page (above the Save and Preview buttons) when you edit it, to let others know what you've done.

## Major modifications

If you have requests, suggestions or questions about this wiki, feel free to create a new discussion on the [Wiki section](http://en.sfml-dev.org/forums/index.php?board=11.0) of the forum. You should also use this forum to report any bug encountered while using the wiki.

## Pages names and local links

With the change from GitHub on how titles of wiki pages are being handled it's no longer possible to keep the address of a page short and handy, but we still want to have a nicely organized wiki, so read the new rules below.

When you create a link to a new page, you must separate the label of the link (for example *A class for animated sprites*) and the address of the page (for example *Tutorial:-Animated-Sprite*). This allows to use descriptive link labels while keeping the name of pages simple enough. Keep in mind that the address will get automatically generated and you don't have to add the dashes for the page name, but instead use spaces.

The example above would translate into the following wiki syntax:

```[A class for animated sprites](Tutorial:-Animated-Sprite)```

Since the page name is now automatically set as the title, you can't avoid adding spaces which then get translated into dashes. Please try to keep the characters count low and try to avoid special characters like slashes, underscores, etc.

This wiki is "flat": you can't create sections/directories, all pages have to be under the "wiki/" root. To keep them organized, please follow this pattern to name them: "wiki/Category:-Page-Name". Categories are the main categories listed on the home page: Tutorial, Source, Project (FAQ should have no child page).

Examples:

* ```wiki/Source:-Light-Manager```
* ```wiki/Tutorial:-Collision-Detection```

## Media (images)

There's no option to upload images or other types of files directly to the wiki, you must instead find an external storage for them. If you do so, please make sure that the file will always be available, and not be automatically deleted from the website where you host it after a certain amount of days.

Well, in fact there's a way to upload files directly to the wiki, but it's kind of unusable: you must clone the wiki's repository, commit your file and push it. That's far from convenient.

If you upload files directly here, please limit yourself to images of acceptable sizes.