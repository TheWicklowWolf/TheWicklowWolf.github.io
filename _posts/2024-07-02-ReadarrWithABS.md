---
layout: post
title: Readarr and AudioBookShelf (without Calibre)
---


## Readarr 

Setup Readarr to use `/data/media/books` as it's root folder


## AudioBookShelf (ABS)

Setup ABS to use `/data/media/abs` as it's books folder


# Link the two 
Use https://github.com/TheWicklowWolf/ConvertBooks to create MOBI, EPUB and AZW3 versions of your books.

        environment:
          - desired_output_formats=.mobi,.epub,.azw3
        volumes:      
          - /data/media/books:/convertbooks/source:ro
          - /data/media/abs:/convertbooks/destination

