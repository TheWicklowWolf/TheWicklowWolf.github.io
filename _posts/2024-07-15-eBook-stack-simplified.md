---
layout: post
title: Simplified eBook Stack
---

![eBook-Stack-Simplified]({{ site.baseurl }}/images/ebook-stack-simplified.png "eBook Stack")


## Services:

- **Readarr** - used for structure, file names and metadata.
- **eBookBuddy** - used for discovering new books.
- **BookBounty** - used for book retrieval.
- **AudioBookShelf (ABS)** - used as the book server.
	
## Folder Structure

`/data/media/ebooks` -> Used for Readarr root folder and ABS book library.


## Readarr Setup

In Settings -> Media Management -> Book Naming -> Standard book format ->

`{Book Series}/{Book SeriesPosition - }{Book Title} {(Release Year)}/{Book SeriesPosition - }{Book Series - }{Author Name} - {Book Title}{ (Release Year)}{ (PartNumber)}`

In this section Rename Books is ticked and Replace Illegal Characters is ticked.


## eBookBuddy

Default Settings.


## BookBounty

The downloads folder can be mapped directly to the Readarr root folder:  
`/data/media/ebooks:/bookbounty/downloads`

With the environment variable `- selected_path_type=folder`, it will attempt to create the correct folder structure (e.g., /author/book) and will download books into their respective folders.

Note:
> Readarr does not automatically rename files after import, and matching may not be perfect.  
> It's recommended to review for unmapped files and use manual import where necessary.  
> After importing, you can rename books using Readarr's file rename function if needed.  

## AudioBookShelf (ABS)

Use `/data/media/ebooks` as its books folder.

### Setup the Send Ebook to Device function (in Settings -> Email):

**1. Set up the email to send from**
> The first step is to configure the e-mail that ABS will use to send the e-mail. The following steps are for a Gmail. If you use a different provider or self-hosted option, follow the steps for that specific email setup.

> Enter the Host (for example, smtp.gmail.com)  
> Set the port (common ports include: 587, 465, 25)  
> - If using 587 or 25, you will need to disable the "Secure" toggle
>
> Set the "Username" to login  
> Set the password  
> If using Gmail, you will need to create an app password [https://support.google.com/mail/answer/185833](https://support.google.com/mail/answer/185833) and use the app password in this field  
> Enter the From Address. e.g. example@example.com

**2. Add Kindle**
> Select Add Device in Ereader Devices section.  
> E-reader devices can only be added to ABS by an admin.  
> Each e-reader device has a Name and an Email, where the Name is how the device will be displayed in ABS and the Email is the destination email address of the device.

**3. Ensure the From address is on the Approved Personal Document E-mail List**
> Before sending, make sure the account you'll use is on your Approved Personal Document E-mail List in your Personal Document Settings.  
> See [https://www.amazon.com/sendtokindle/email](https://www.amazon.com/sendtokindle/email)

## Conclusion
With ABS's "Send Ebook to Device" feature, you don't need multiple formats for each eBook. 
Once your Kindle is properly set up, the book will automatically be delivered in the correct format.

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
  
```


---
