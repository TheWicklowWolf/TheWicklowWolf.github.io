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
`/data/media/music/singles` -> Used for Syncify to store singles from playlists.


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
`/data/media/music/singles:/syncify/downloads`

Each playlist added will have its own folder containing the individual songs. As this runs on a schedule, whenever you add a track to your Spotify playlist, it is retrieved by Syncify.



## Media Servers

Add `/data/media/music/managed` as a standard music library.  
Add `/data/media/music/singles` as a separate music library but set it to prefer local metadata.  

Both singles and albums are kept separate but they remain easily accessible.


## Conclusion
This approach allows you to download singles without needing to download entire albums and helps keep the album collection organized. As a result, you have a well-maintained library for full albums and a separate library for individual singles. 

This setup demonstrates an effective way to manage both albums and singles within your music stack.


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
      - /data/media/music/singles:/syncify/downloads
      - /etc/localtime:/etc/localtime:ro
    ports:
      - xxxx:5000
    restart: unless-stopped

```


---



## Bonus #1 - PlaylistDir  
### Automatically Generate Playlists from Subdirectories  

This service generates playlists for each subdirectory containing MP3 files within a specified directory. This allows you to transform your files downloaded with Syncify into playlists within Jellyfin and/or Plex.


```yaml
services:
  playlistdir:
    image: thewicklowwolf/playlistdir:latest
    container_name: playlistdir
    environment:
      - media_server_addresses=Plex:http://192.168.1.2:32400, Jellyfin:http://192.168.1.2:8096
      - media_server_tokens=Plex:x-token, Jellyfin:api-token
      - plex_library_section_id=0
      - path_to_parent=/data/media/music/singles
      - path_to_playlists=/data/media/music/singles/playlists
    ports:
      - xxxx:5000
    volumes:
      - /data/media/music/singles:/playlistdir/parent:ro
      - /data/media/music/singles/playlists:/playlistdir/playlists
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
```

## Bonus #2 - SpotTube  
### Download Tracks from Spotify Playlists or Albums  

This service allows you to download specific albums or playlists from Spotify using yt-dlp. It creates a folder named after the playlist or album and saves the files there.  
You can also use this in conjunction with PlaylistDir to create playlists in Jellyfin and/or Plex.  

```yaml
services:
  spottube:
    image: thewicklowwolf/spottube:latest
    container_name: spottube
    environment:
      - spotify_client_id=abc
      - spotify_client_secret=123
      - thread_limit=1
    volumes:
      - /path/to/config:/spottube/config
      - /data/media/music/singles:/spottube/downloads
      - /etc/localtime:/etc/localtime:ro
    ports:
      - xxxx:5000
    restart: unless-stopped
```


---
