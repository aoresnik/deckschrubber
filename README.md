# Deckschrubber
> *n. person responsible for scrubbing a ship's deck.*

[![Go Report Card](https://goreportcard.com/badge/github.com/fraunhoferfokus/deckschrubber)](https://goreportcard.com/report/github.com/fraunhoferfokus/deckschrubber)
[![License](https://img.shields.io/github/license/fraunhoferfokus/sesame.svg)](https://github.com/fraunhoferfokus/sesame/blob/master/LICENSE)

Deckschrubber inspects images of a [Docker Registry](https://docs.docker.com/registry/) and removes those older than a given age.

## Quick Start

```bash
go get github.com/fraunhoferfokus/deckschrubber
$GOPATH/bin/deckschrubber
```

## Why this?
We run our own private registry on a server with limited storage and it was only a question of time, until it was full! Although there are similar approaches to do Docker registry house keeping (in Python, Ruby, etc), a native module (using Docker's own packages) was missing. This module has the following advantages:

* **Is binary!**: No runtime, Python, Ruby, etc. is required
* **Uses Docker API**: same packages and modules used to relaze Docker registry are used here

## Arguments
```
-day int
      max age in days
-debug
      run in debug mode      
-dry
      does not actually deletes
-latest int
      number of the latest matching images of an repository that won't be deleted (default 1)      
-month int
      max age in months
-registry string
      URL of registry (default "http://localhost:5000")
-repo string
      matching repositories (allows regexp) (default ".*")      
-repos int
      number of repositories to garbage collect (default 5)
-tag string
      matching tags (allows regexp) (default ".*")      
-ntag string
      match everything but this tag (allows regexp) (default empty)
-v    shows version and quits
-year int
      max age in days


```

## Registry preparation
Deckschrubber uses the Docker Registry API.
Its delete endpoint is disabled by default, you have to enable it with the following entry in the registry configuration file:

```
delete:
  enabled: true
```

See [the documentation](https://github.com/docker/distribution/blob/master/docs/configuration.md#delete) for details.

## Examples

* **Remove all images older than 2 months and 2 days**

```
$GOPATH/bin/deckschrubber -month 2 -day 2
```

* **Remove all images older than 1 year from `http://myrepo:5000`**

```
$GOPATH/bin/deckschrubber -year 1 -registry http://myrepo:5000
```

* **Analyize (but do not remove) images of 30 repositories**

```
$GOPATH/bin/deckschrubber -repos 30 -dry
```

* **Remove all images of each repository except the 3 latest ones**

```
$GOPATH/bin/deckschrubber -latest 3
```

* **Remove all images with tags that ends with '-SNAPSHOT'**

```
$GOPATH/bin/deckschrubber -tag ^.*-SNAPSHOT$
```

* **Usage in a scenario where we want to keep CI images for 1 week, but keep at
least 10 released images for at least 3 months**

CI tags consist of Git commit hash (40 hex digits). All other tags are
considered release tags. The same image can have both CI tag and release tag
(but can also have only one of both for some projects).

The problem is that the [docker-distribution](https://github.com/docker/distribution)
registry server doesn't implement tag deletion, only image deletion. In order to
support it, deckschrubber must only use image deletion. It should only delete
the images where all the tags match deletion criteria. By default, deckschrubber
does that. But in this case we have to specify to delete anyway, because
otherwise the CI tags would prevent deletion.

```
# Scrub obsolete CI images (don't delete images that have also release tag)
$GOPATH/bin/deckschrubber -day 7 -tag ^.*-SNAPSHOT$
# Scrub obsolete release images (delete images that also have CI tag)
$GOPATH/bin/deckschrubber -no-other-tags-check -month 3 -latest 10 -ntag ^.*-SNAPSHOT$
```
