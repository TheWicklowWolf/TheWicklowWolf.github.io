---
layout: post
title: Full eBook Stack
---

![eBook-Stack]({{ site.baseurl }}/images/ebook-stack.png "eBook Stack")


## Services:

- **Readarr** - used for structure, file names, and metadata.
- **eBookBuddy** - used for discovering new books.
- **BookBounty** - used for book retrieval.
- **ConvertBooks** - used for book conversions.
- **AudioBookShelf (ABS)** - used as book server.
	
## Folder Structure

`/data/media/abs_books` -> Used for ABS book library.

`/data/media/ebooks` -> Used for Readarr root folder.


## Readarr Setup

In Settings -> Media Management -> Book Naming -> Standard book format ->

`{Book Series}/{Book SeriesPosition - }{Book Title} {(Release Year)}/{Book SeriesPosition - }{Book Series - }{Author Name} - {Book Title}{ (Release Year)}{ (PartNumber)}`

In this section Rename Books is ticked and Replace Illegal Characters is ticked.


## eBookBuddy

Default Settings.


## BookBounty

The downloads folder can be mapped directly to the Readarr root folder:  
`/data/media/ebooks:/bookbounty/downloads`

This will attempt to create the correct folder structure (/author/book etc.) and will download books into their respective folders.

Note:
> Readarr does not automatically rename files after import, and matching may not be perfect.  
> It's recommended to review for unmapped files and use manual import where necessary.  
> After importing, you can rename books using Readarr's file rename function if needed.  

## ConvertBooks

This tool can convert your ebooks into multiple formats such as MOBI, EPUB, and AZW3.
See [https://github.com/TheWicklowWolf/ConvertBooks](https://github.com/TheWicklowWolf/ConvertBooks).

```yaml
    environment:
      - desired_output_formats=.mobi,.epub,.azw3
    volumes:      
      - /data/media/ebooks:/convertbooks/source:ro
      - /data/media/abs_books:/convertbooks/destination
```

## AudioBookShelf (ABS)

Use `/data/media/abs_books` as its books folder.


## Conclusion
Did you notice that there are no Calibre or Calibre-web containers used in this setup?  
Instead, ABS is utilized, offering each book in multiple formats and including a convenient "send to Kindle" feature.   
By following this setup, your ebook collection will be well organized and easily accessible in various formats, eliminating the need for the full Calibre application.  

## Example YAML Configuration

```yaml
services:
  readarr:
    image: lscr.io/linuxserver/readarr:develop
    container_name: readarr
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - /path_to_config/readarr:/config
      - /data/media/ebooks:/data/media/ebooks
    ports:
      - xxxx:8787
    restart: unless-stopped

 audiobookshelf:
    image: ghcr.io/advplyr/audiobookshelf:latest
    container_name: audiobookshelf
    volumes:
      - /data/media/abs_books:/books
      - /data/media/audiobooks:/audiobooks
      - /data/media/podcasts:/podcasts
      - /path_to_config/audiobooks:/config
      - /path_to_config/audiobooks/metadata:/metadata
    ports:
      - xxxx:80
    restart: unless-stopped
  
  bookbounty:
    image: thewicklowwolf/bookbounty:latest
    container_name: bookbounty
    environment:
      - selected_path_type=folder
      - library_scan_on_completion=True
    volumes:
      - /data/media/ebooks:/bookbounty/downloads
      - /path_to_config/bookbounty:/bookbounty/config
      - /etc/localtime:/etc/localtime:ro
    ports:
      - xxxx:5000
    restart: unless-stopped
    
  ebookbuddy:
    image: thewicklowwolf/ebookbuddy:latest
    container_name: ebookbuddy
    environment:
      - auto_start=True
      - search_for_missing_book=True
    volumes:
      - /path_to_config/ebookbuddy:/ebookbuddy/config
      - /etc/localtime:/etc/localtime:ro
    ports:
      - xxxx:5000
    restart: unless-stopped
  
  convertbooks:
    image: thewicklowwolf/convertbooks:latest
    container_name: convertbooks
    environment:
      - desired_output_formats=.mobi,.epub,.azw3
      - schedule=1 # Run at 1 AM
      - run_at_startup=False
      - thread_limit=1
    volumes:
      - /data/media/ebooks:/convertbooks/source
      - /data/media/abs_books:/convertbooks/destination
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
```


---
