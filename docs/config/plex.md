# Plex Attributes

Configuring [Plex](https://www.plex.tv/) is required in order to connect to your libraries.

A `plex` mapping can be either in the root of the config file as global mapping for all libraries, or you can specify the `plex` mapping individually per library.

Below is a `plex` mapping example and the full set of attributes:
```yaml
plex:
  url: http://192.168.1.12:32400
  token: ####################
  timeout: 60
  clean_bundles: true
  empty_trash: true
  optimize: false
```

| Attribute       | Allowed Values                                                         | Default | Required |
|:----------------|:-----------------------------------------------------------------------|:-------:|:--------:|
| `url`           | Plex Server URL<br><strong>Example:</strong> http://192.168.1.12:32400 |   N/A   | &#9989;  |
| `token`         | Plex Server Authentication Token                                       |   N/A   | &#9989;  |
| `timeout`       | Plex Server Timeout                                                    |   60    | &#10060; |
| `clean_bundles` | Runs Clean Bundles on the Server after all Metadata Files are run      |  false  | &#10060; |
| `empty_trash`   | Runs Empty Trash on the Server after all Metadata Files are run        |  false  | &#10060; |
| `optimize`      | Runs Optimize on the Server after all Metadata Files are run           |  false  | &#10060; |

* This script can be run on a remote Plex server, but be sure that the `url` provided is publicly addressable, and it's recommended to use `HTTPS`.

* If you need help finding your Plex authentication token, please see Plex's [support article](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/).

# Secondary Plex Server

This fork can keep a second Plex server in step with the first one. The first server (`plex`) is the primary and the second server (`plex_secondary`) is the secondary. Both are expected to be clones of each other: the same libraries with the same names, looking at the same files on the same paths.

```yaml
plex:
  url: http://192.168.1.12:32400
  token: ####################
plex_secondary:
  url: http://192.168.1.12:32401
  token: ####################
  timeout: 60
```

| Attribute       | Allowed Values                                                         |      Default       | Required |
|:----------------|:-----------------------------------------------------------------------|:------------------:|:--------:|
| `url`           | Secondary Plex Server URL                                              |        N/A         | &#9989;  |
| `token`         | Secondary Plex Server Authentication Token                             |  primary `token`   | &#10060; |
| `timeout`       | Secondary Plex Server Timeout                                          | primary `timeout`  | &#10060; |
| `clean_bundles` | Runs Clean Bundles on the Secondary Server after the run               | primary value      | &#10060; |
| `empty_trash`   | Runs Empty Trash on the Secondary Server after the run                 | primary value      | &#10060; |
| `optimize`      | Runs Optimize on the Secondary Server after the run                    | primary value      | &#10060; |

How it runs:

* Every library is run against the primary server first, then the same library is run against the secondary server with the same metadata files, collections, overlays and library operations. Playlists are run on both servers too.
* If the primary server can't be reached, the secondary server still runs.
* Shared resources are only touched by the primary: Radarr and Sonarr adds and tags, `radarr_add_all_existing`, `sonarr_add_all_existing`, `*_remove_by_tag`, `metadata_backup` and the missing items file are skipped on the secondary so nothing happens twice.
* Rating keys are different on each server, so the secondary gets its own image cache, its own overlay poster backups (in `overlays/<name>/secondary/`) and its own log folder (`logs/<library>-secondary/`). Tautulli results are matched to the secondary server by title and year.
* A library can override the secondary connection with its own `plex_secondary` mapping, or skip the secondary server with `plex_secondary: false`.

```yaml
libraries:
  Movies:
    metadata_path:
      - file: config/Movies.yml
  4K Movies:
    plex_secondary: false             # only exists on the primary server
```

# Other examples:

Specifying a second Plex server for a specific library:

In this example we have two Plex servers [with the same libraries] and want to add the same collections to each.  The second Plex is less responsive, so it needs a higher timeout than the primary one.

```yaml
libraries:
  Movies:
    metadata_path:
      - file: config/Movies.yml
  Movies_on_Second_Plex:
    library_name: Movies
    metadata_path:
      - file: config/Movies.yml
    plex:
      url: http://plex.boing.bong
      token: SOME_TOKEN
      timeout: 360
...
plex:
  url: http://plex.bing.bang
  token: SOME_TOKEN
  timeout: 60
  clean_bundles: false
  empty_trash: false
  optimize: false
...
```
