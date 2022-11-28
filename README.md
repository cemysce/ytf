# `ytf`: YouTube Follow

This is a tool for "following" YouTube playlists[^pl] without actually creating
an account, signing in, reporting your interests to Google, or opening YouTube
and risking getting distracted by other videos.

Every time you run it, it will generate a report of videos newer than the ones
it knew about the previous time it ran (if previous report still exists, it
will replace it with an updated one that includes the same videos plus any new
ones.)

## Installation

1. Install [Python](https://www.python.org/) (&ge; 3.8) and
   [other requirements](requirements.txt).
2. Clone this repo.
3. If you intend to run the script by explicitly specifying the path to it,
   or just using the repo directory as the current working directory, then you
   can stop here.
4. To make the script available in your `PATH`, follow whatever makes sense for
   your OS.  Note that on POSIX systems, all you'll need is the `ytf` script.
   On Windows, unless you know what you're doing and know how to run a Python
   script on Windows, you'll need `ytf` **and** the `.bat` files
   (`ytf-generate.bat`, `ytf-view.bat`) and they'll need to remain in the same
   directory.

## Configuration

[**_tl;dr?_**](#tldr)

`ytf` will expect a `ytf-config.json` file to exist in the current working
directory from which you run it.

The file should contain information about which playlists[^pl] to follow:
* _(optional)_ name
  * used only for display purposes
  * if missing it will be populated automatically
* **(required)** URL of one of the following:
  * a channel/user's page listing their videos, live streams, or shorts
    * URL should look like one of these:
      * `https://www.youtube.com/<...>/<page>`
      * `https://www.youtube.com/c/<...>/<page>`
      * `https://www.youtube.com/channel/<...>/<page>`
      * `https://www.youtube.com/user/<...>/<page>`
    * where `<page>` is one of:
      * `videos`
      * `streams`
      * `shorts`
  * a playlist ID
    * URL should look like this:
      * `https://www.youtube.com/playlist?list=<...>`
    * A playlist ID URL only makes sense if its maintainer is adding new
      videos to the start of the list, not the end of the list.  Otherwise,
      `ytf`'s known-video logic will not work correctly.

_Example:_

```json
{
    "playlist_infos": [
        {
            "name": "NASA Jet Propulsion Laboratory",
            "url": "https://www.youtube.com/c/NASAJPL/videos"
        },
        {
            "name": "Computerphile",
            "url": "https://www.youtube.com/user/Computerphile/videos"
        },
        {
            "name": "Numberphile",
            "url": "https://www.youtube.com/user/numberphile/videos"
        }
    ]
}
```

### State

`ytf` maintains the state of known video IDs in a `ytf-state.json` file.

The file should contain information about each playlist's[^pl] known videos:
* playlist[^pl] URL
* known video ID(s)
  * comma-separated list
  * if specified, will be used to limit results in report
  * if missing, report will contain entire list of videos
  * will be populated/updated automatically after each run
  * **NOTE:** `yt-dlp` supports limiting query to only videos uploaded after a
    specified date, but this feature has a few critical flaws preventing it
    from being used here.  See comments in code for details.

The file keeps this information in 2 lists, one for the state before any
existing report file was generated, and one for the state after it was
generated.  The **before** state is used when a report file exists and
therefore must be regenerated as if it had never been created, and the
**after** state is used when there is no report file (under the assumption that
the user has deleted it because they've already viewed it).

If you're preparing for an initial run and don't want the report to contain the
entire history of videos, it should be sufficient to create a state file
containing only the most recent videos' IDs:

_Example:_

```json
{
    "pre-report": [],
    "post-report": [
        {
            "playlist_url": "https://www.youtube.com/c/NASAJPL/videos",
            "known_ids": "UOdbwQE-0z4"
        },
        {
            "playlist_url": "https://www.youtube.com/user/Computerphile/videos",
            "known_ids": "2iF9PRriA7w"
        },
        {
            "playlist_url": "https://www.youtube.com/user/numberphile/videos",
            "known_ids": "nQR1NY03zIA"
        }
    ]
}
```

### tl;dr

Create a `ytf-config.json` file in the directory you'll be working in.  At the
bare minimum, it should contain playlists'[^pl] URLs.

_Example:_

```json
{
    "playlist_infos": [
        { "url": "https://www.youtube.com/c/NASAJPL/videos" },
        { "url": "https://www.youtube.com/user/Computerphile/videos" },
        { "url": "https://www.youtube.com/user/numberphile/videos" }
    ]
}
```

The first time you run `ytf` it will generate a possibly huge report containing
all videos in these playlists[^pl], but each successive run will generate a
report of only the videos added since the previous run.

## Running

Run `ytf` without any arguments to get its usage information.  It supports
multiple modes of operation.  Multiple concurrent runs in the same directory,
in any mode, are not supported and are blocked.

### `generate` Mode

On each `generate` run, `ytf` will get the list of videos from each
playlist[^pl] URL you've specified in the configuration file, skip over known
videos (if the configuration file includes the necessary information), then
generate an HTML report of the rest of the videos (i.e. the "new" ones).  It
will also update the state file to record what it saw for each playlist[^pl] so
that on subsequent runs only newer videos will be reported, and if the
configuration file was missing a name for the playlist[^pl] it will update that
file too.

Multiple subsequent runs in the same directory will either generate a new
report file if none exists, or will generate an updated file (containing
everything in the existing report plus any new videos) to replace the existing
file.

### `view` Mode

Use the `view` mode to open the report in your default web browser.  New videos
(if any) for each playlist[^pl] will be listed as embedded video frames.  You
will be prompted to delete the report file after it is opened.

### Running on Windows

Unless you know how to properly run a Python script on Windows, use the `.bat`
convenience scripts instead of attempting to run `ytf` directly (but note that
`ytf` must be in the same directory as the `.bat` scripts).

The `.bat` scritps can be run from a Windows command prompt, or by
double-clicking on them (which will open a command window).

[^pl]: The term "playlist" is used throughout this project to refer to a
**generalized** concept of playlists, which includes various kinds of YouTube
pages listing videos, not just explicit playlist URLs like
`https://..../playlist?list=...`.
