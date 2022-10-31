# `ytf`: YouTube Follow

This is a tool for reporting new videos on YouTube channels/playlists you're
interested in.

Benefits:
* no need to sign into YouTube and report to Google which channels you're following
* no need to consume your time by manually checking YouTube, and risking getting distracted by other videos

## Installation

1. Install [Python](https://www.python.org/) (&ge; 3.8) and [`yt-dlp`](https://github.com/yt-dlp/yt-dlp).
2. Clone this repo.
3. If you intend to run the script by explicitly specifying the path to it,
   or just using the repo directory as the current working directory, then you
   can stop here.
4. To make the script available in your `PATH`, follow whatever makes sense for
   your OS.  Note that on POSIX systems, all you'll need is the `ytf` script.
   On Windows, you'll need **both** `ytf` and `ytf.bat` (unless you know what
   you're doing and know how to run a Python script on Windows), and they need
   to remain in the same directory.

## Configuration

[_tl;dr?_](#tldr)

`ytf` will expect a `ytf-config.json` file to exist in the current working
directory from which you run it.

The file should contain information about which channels/playlists to follow
and some additional metadata:
* name of channel
  * used only for display purposes
  * if missing it will be populated automatically
* videos URL
  * URL of one of the following:
    * channel/user's videos page
      * URL should look like one of these:
        * `https://www.youtube.com/c/<...>/videos`
        * `https://www.youtube.com/<...>/videos`
        * `https://www.youtube.com/user/<...>/videos`
        * `https://www.youtube.com/channel/<...>/videos`
    * playlist ID
      * URL should look like this:
        * `https://www.youtube.com/playlist?list=<...>`
      * A playlist ID URL only makes sense if its maintainer is adding new
        videos to the start of the list, not the end of the list.  Otherwise,
        `ytf`'s known-video logic will not work correctly.
      * When using a playlist ID URL, last known upload date (see below) won't
        work so don't bother setting it.
  * the only **required** field
* last known video ID, last known upload date
  * if either or both specified, will be used to limit results in report
    * On its own, last known upload date becomes unreliable over time due to how `yt-dlp` approximates video upload dates (`ytf` may work around this in the future).
  * if both missing, report will contain entire list of videos
  * last known video ID will be updated automatically after each run
  * last known upload date will be updated automatically after each run, if URL was not a playlist ID

There is also an optional `verbose` property at the top-level, which if enabled
will cause `ytf` to run `yt-dlp` in verbose mode.  This is useful for debugging
or to demonstrate that it is actually doing something when it would have
otherwise appeared to be hanging (i.e. for channels/playlists with thousands of
videos or if you have a slow connection).

Example:

```json
{
    "verbose": false,
    "channels": [
        {
            "name": "NASA Jet Propulsion Laboratory",
            "videos_url": "https://www.youtube.com/c/NASAJPL/videos",
            "last_known_id": "UOdbwQE-0z4",
            "last_known_upload_date": "20221027"
        },
        {
            "name": "Computerphile",
            "videos_url": "https://www.youtube.com/user/Computerphile/videos",
            "last_known_id": "2iF9PRriA7w",
            "last_known_upload_date": "20221025"
        },
        {
            "name": "Numberphile",
            "videos_url": "https://www.youtube.com/user/numberphile/videos",
            "last_known_id": "nQR1NY03zIA",
            "last_known_upload_date": "20221026"
        }
    ]
}
```

### tl;dr

Create a `ytf-config.json` file in the directory you'll be working in.  At the
bare minimum, it should contain URLs to the channels/playlists' respective
pages listing their videos, for example:

```json
{
    "channels": [
        { "videos_url": "https://www.youtube.com/c/NASAJPL/videos" },
        { "videos_url": "https://www.youtube.com/user/Computerphile/videos" },
        { "videos_url": "https://www.youtube.com/user/numberphile/videos" }
    ]
}
```

The first time you run `ytf` it will generate a possibly huge report containing
all videos on these channels, but each successive run will generate a report of
only the videos released since the previous run.

## Running

`ytf` takes no arguments, it's controlled entirely by [configuration file](#configuration).

On each run, `ytf` will get the list of videos from each channel/playlist URL
you've specified in the configuration file, skip over known videos (if the
configuration file includes the necessary information), then generate an HTML
report of the rest of the videos (i.e. the "new" ones).  It will also update
the configuration file to record what it last saw for each channel/playlist, so
that on subsequent runs only newer videos will be reported.

The report file name will contain a timestamp (so that multiple runs will just
write new files without replacing old ones) and the number of items in the
report (so that you can see at a glance without having to open the report).

Open the report in a web browser to see the list of videos (if any) for each
channel/playlist, as embedded video frames.

### Running on Windows

Unless you know how to properly run a Python script on Windows, use the
`ytf.bat` convenience script instead of attempting to run `ytf` directly (but
note that `ytf` must be in the same directory as `ytf.bat`).

`ytf.bat` can be run from a Windows command prompt, or run by double-clicking
on it (which will open a command window).
