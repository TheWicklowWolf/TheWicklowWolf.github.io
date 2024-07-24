---
layout: post
title: Readarr and AudioBookShelf Integration (with multiple formats for each eBook)
---

> This approach is ideal if you prefer not to use Calibre or Calibre Web but still want to manage multiple versions of each eBook.

## Readarr 

Setup Readarr to use `/data/media/ebooks` as it's root folder.


## AudioBookShelf (ABS)

Setup ABS to use `/data/media/abs_books` as it's books folder.


# Linking Readarr and AudioBookShelf
To seamlessly convert and link your ebooks from Readarr to ABS, you can use a tool called ConvertBooks. This tool can convert your ebooks into multiple formats such as MOBI, EPUB, and AZW3.
See [https://github.com/TheWicklowWolf/ConvertBooks](https://github.com/TheWicklowWolf/ConvertBooks).

    environment:
      - desired_output_formats=.mobi,.epub,.azw3
    volumes:      
      - /data/media/ebooks:/convertbooks/source:ro
      - /data/media/abs_books:/convertbooks/destination


# Conclusion
By setting up Readarr and AudioBookShelf as described, you ensure your ebook collection is well-organized and accessible in multiple formats without needing Calibre.

