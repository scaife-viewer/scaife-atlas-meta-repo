# New ATLAS and Scaife Viewer Meta-Repository

This repository is for deploying the
[ATLAS](https://github.com/scaife-viewer/new-atlas) server and the [Scaife
Viewer](https://github.com/scaife-viewer/scaife-viewer) frontend.

It has three main goals:

1. Provide more control over the deployment process than the current
   sequentially-run GitHub actions
2. Create reproducible and testable builds via podman and `podman compose`
3. Make both services more infrastructure-agnostic

## Deployment

### Dependencies

- Submodules
  - `[scaife-viewer](https://github.com/scaife-viewer/scaife-viewer)`: requires
    Python < 3.9 and Node.js v12
  - `[new-atlas](https://github.com/scaife-viewer/new-atlas)`: Python <= 3.12
- [Docker](https://docker.com) or [Podman](https://podman-desktop.io/) (Tufts
  uses Podman)
- Docker images (these are already configured in `compose.yml`)
  - ElasticSearch
  - PostgreSQL

### Required Manual Interventions

1. Update submodules: `git submodule update --remote`
2. Upload [ATLAS data](#atlas-data) gzip to host and unzip to
   `./new-atlas/server/`: `tar xzvf atlas-data.tar.gz -C new-atlas/server/`

#### ATLAS data

Eventually, we should automate the creation of the ATLAS data gzip,
but we need to take a clearer inventory of where the data are coming
from and how they're being transformed.

For the time being, use existing ATLAS data by running

```sh
tar cvzf atlas-data.tar.gz data db
```

in the new-atlas repository.

### Starting the application

Make sure to complete the [required manual interventions](#required-manual-interventions)
before attempting to start the application.

Ideally, the application should spin up with `podman compose up` - or, if new
images need to be built due to underlying changes, `podman compose up --build`.
However, several issues have arisen that can lead to a non-functional application.

#### Loading ATLAS data

The Scaife Viewer database needs to ingest information about the various
corpora into its ATLAS database. This database is managed by the `atlas`
dependency in the `[backend](https://github.com/scaife-viewer/backend)`
repository. We need to ensure that its migrations are applied
_before_ attempting to ingest the data.

To apply the `atlas` migrations:

```sh
python manage.py migrate --database=atlas
```

To load and prepare the text repositories from
`scaife-viewer/data/content-manifests/production.yaml`:

```sh
python manage.py load_text_repos
python manage.py slim_text_repos
```

Finally, to prepare the `atlas` database:

```sh
./bin/copy_corpus_repo_metadata
python manage.py prepare_atlas_db --force
```

#### Building the search index

After the ATLAS database has been prepared (following the steps above), the
ElasticSearch index needs to be built. Be very careful here: this process has
crashed servers.

To run the ElasticSearch indexer:

```sh
pythong manage.py indexer --max-workers=1 --limit=1000
```

As of 2026-04-16: Although this command seemingly runs without issue as part of
`scaife-viewer/deploy/entrypoint.sh`, it seems to be necessary to run it
manually after the Podman containers are running. If that happens, you
will also need to prepare the ATLAS DB manually. To do so, use

```sh
podman compose exec scaife-viewer python manage.py prepare_atlas_db --force
podman compose exec scaife-viewer python manage.py indexer --max-workers=1 --limit=1000
```

DON'T GET GREEDY WITH WORKERS. This will take a while, but that's better than
crashing the server.

## LICENSE

The MIT License

Copyright (c) 2017-2026 Perseus Digital Library

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
