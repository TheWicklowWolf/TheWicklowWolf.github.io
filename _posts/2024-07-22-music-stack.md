---
layout: post
title: Full Music Stack
---

![full-music-stack]({{ site.baseurl }}/images/full-music-stack.png "Music Stack")


## Services:

- **Lidarr** - used for structure, file names and metadata.
- **LidaTube** - used for retrieving albums.
- **Lidify** - used for discovering new artists.
- **Syncify** - used for retrieving singles from playlists.
	


## Media Server and companion apps
- **Plex and Plexamp**
- **Jellyfin and Finamp**



## Folder Structure

`/data/media/music/managed` -> Used for Lidarr root folder.  
`/data/media/music/disorganized` -> Used for Syncify to store singles from playlists.


## Lidarr Setup

In Settings -> Media Management -> Track Naming -> Standard Track Format ->

`{Album Title} ({Release Year})/{Artist Name} - {Album Title} - {track:00} - {Track Title}`

In Settings -> Media Management -> Track Naming -> Artist Folder Naming ->

`{Artist Name}`

In this section Rename Tracks is ticked and Replace Illegal Characters is ticked.


## LidaTube

The downloads folder can be mapped directly to the Lidarr root folder:  
`/data/media/music/managed:/lidatube/downloads`

Lidatube will attempt to create the correct folder structure (e.g., /artist/album) and will download albums into their respective folders.

Note:
> Lidarr does not automatically rename files after import, and matching may not be perfect.  
> Some tracks/albums will not be imported automatically as there are differences in track length, metadata etc.

> If you want an easy way to improve import matching, you have the option to disable fingerprinting. However, this approach may _potentially_ lead to incorrect imports.  
> - Lidarr Settings -> Media Management -> Allow Fingerprinting -> Never  
    (with advanced options shown)  

> It's recommended to review for unmapped files and use manual import where necessary.  
> After importing, you can rename tracks using Lidarr's file renaming function, if needed.  


## Lidify


Default Settings.


## Syncify

> Syncify is a tool for synchronising and fetching content from a Spotify playlist via yt-dlp.

The downloads folder can be mapped to a location that is not managed by Lidarr:  
`/data/media/music/disorganized:/syncify/downloads`

Each playlist added will have its own folder containing the individual songs. As this runs on a schedule, whenever you add a track to your Spotify playlist, it is retrieved by Syncify.



## Media Servers

Add `/data/media/music/managed` as a standard music library.  
Add `/data/media/music/disorganized` as a separate music library but set it to prefer local metadata.  

This way, singles and albums remain separated while still accessible.


## Conclusion
This setup illustrates a music stack that effectively manages both albums and individual singles.


## Example YAML Configuration

```yaml
services:
  lidarr:
    image: lscr.io/linuxserver/lidarr:latest
    container_name: lidarr
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - /path/to/config:/config
      - /data/media/music/managed:/data/media/music/managed
    ports:
      - xxxx:8787
    restart: unless-stopped

  lidatube:
    image: thewicklowwolf/lidatube:latest
    container_name: lidatube
    volumes:
      - /path/to/config:/lidatube/config
      - /data/media/music/managed:/lidatube/downloads
      - /etc/localtime:/etc/localtime:ro
    ports:
      - xxxx:5000
    restart: unless-stopped

  lidify:
    image: thewicklowwolf/lidify:latest
    container_name: lidify
    volumes:
      - /path/to/config:/lidify/config
      - /etc/localtime:/etc/localtime:ro
    ports:
      - xxxx:5000
    restart: unless-stopped
  
  syncify:
    image: thewicklowwolf/syncify:latest
    container_name: syncify
    volumes:
      - /path/to/config:/syncify/config
      - /data/media/music/disorganized:/syncify/downloads
      - /etc/localtime:/etc/localtime:ro
    ports:
      - xxxx:5000
    restart: unless-stopped

```


---
