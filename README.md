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

_(**TL;DR?** Click [here](#quick-basic-setup) for quick basic setup
instructions.)_

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

`ytf` maintains the state of acknowledged video IDs (i.e. IDs of videos that
the user's actions indicate were either viewed or consciously ignored), as
as well as the raw data that was used to generate the most recent report file,
in a `ytf-state.json` file.

Beyond the (**optional!**) initial setup described below, this file is not
intended to be user-edited, therefore its format will not be documented here.
Modifying this file can have unintended consequences, so refrain from doing so
unless you know what you're doing and have made a backup.

If you're preparing for an initial run and don't want the report to contain the
entirety of each playlist's[^pl] videos, you may create an initial state file
containing only the most recent videos' IDs:

_Example:_

```json
{
    "pre-report": [
        {
            "playlist_url": "https://www.youtube.com/c/NASAJPL/videos",
            "acked_video_ids": "UOdbwQE-0z4"
        },
        {
            "playlist_url": "https://www.youtube.com/user/Computerphile/videos",
            "acked_video_ids": "2iF9PRriA7w"
        },
        {
            "playlist_url": "https://www.youtube.com/user/numberphile/videos",
            "acked_video_ids": "nQR1NY03zIA"
        }
    ]
}
```

### Quick Basic Setup

_(Ignore this section if you already followed the full instructions above.)_

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

The first time you run `ytf` it will generate a possibly massive report
containing all videos in these playlists[^pl], and opening such a large report
(which contains embeds of each video) in your browser could very well bring
your system to its knees.  However, each successive run will generate a report
of only the videos added since the previous run.

Therefore, you may choose to ignore the first generated report.  In other
words, run `ytf` the first time to generate that massive report file, wait for
it to complete, then delete that report file.  Reports generated from this
point forward should be much more manageable in size.

## Running

Run `ytf` without any arguments to get its usage information.  It supports
multiple modes of operation.  Multiple concurrent runs in the same directory,
in any mode, are not supported and are blocked.

### `generate` Mode

On each `generate` run, `ytf` will get the list of videos from each
playlist[^pl] URL you've specified in the configuration file, skip over
already-acknowledged videos (if the configuration file includes the necessary
information), then generate an HTML report of the rest of the videos (i.e. the
"new" ones).  It will also update the state file to record what it saw for each
playlist[^pl] so that on subsequent runs only newer videos will be reported,
and if the configuration file was missing a name for the playlist[^pl] it will
update that file too.

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

The `.bat` scripts can be run from a Windows command prompt, or by
double-clicking on them (which will open a command window).

## Lessons Learned

This section is really only relevant to developers, particularly Python ones.
Here's a list of a few techniques I learned while writing this program.  If any
of them interest you, you may want to take a look at the source code.

* safely (i.e. atomically/transactionally) handling files in Python (reading,
  writing, renaming, etc.) to tolerate unexpected failures and concurrency, and
  the relevant system calls for common OSes
* using Python context managers for some of the above and for other things
* using `ctypes` module to directly call OS-specific system calls that Python
  does not otherwise provide wrappers for via its standard library
* handling signals/events for Windows console programs that Python does not
  handle (like closing console window or typing Ctrl+Break while program is
  still running)
* keypress detection on POSIX and Windows
* `asyncio` module to start a child process and read its stdout and stderr in
  "parallel" reliably (i.e. without possibility of deadlock)
* using `codecs` module to parse text from bytes on a "best effort" basis (to
  gracefully handle cases where raw string is read in chunks and a multibyte
  character spans multiple chunks)
* using `json` module to parse **multiple** JSON objects in a single string,
  and also operate on a "best effort" basis (to gracefully handle cases where
  raw string is read in chunks and a JSON object spans multiple chunks)
* numerous idiosyncrasies in Python, Windows, and Python's behavior on Windows,
  some of which are actual bugs or just unresolved mysteries, with links in the
  code
* various `yt-dlp` options needed for running it consistently and reliably,
  especially across multiple different OSes
* [`subprocess` module has sloppy and documentation (repeats itself, could be
  organized better), and is misleading](https://github.com/python/cpython/issues/99864):
  * may take Python file objects as arguments, but no transcoding is done
    between child process output and writing to file; in other words Python
    directly passes files' descriptors to child processes (as opposed to
    creating and passing intermediate pipes, which are then read from and
    transcoded according to type and attributes of file object); so if parent
    process uses one type of line ending or encoding, and child process uses
    another, you'll end up with a corrupted or mixed-encoding file

[^pl]: The term "playlist" is used throughout this project to refer to a
**generalized** concept of playlists, which includes various kinds of YouTube
pages listing videos, not just explicit playlist URLs like
`https://..../playlist?list=...`.
