# New ATLAS and Scaife Viewer Meta-Repository

This repository is for deploying the [ATLAS](https://github.com/scaife-viewer/new-atlas) server and the [Scaife Viewer](https://github.com/scaife-viewer/scaife-viewer) frontend.

It has three main goals:

1. Provide more control over the deployment process than the current sequentially-run GitHub actions
2. Create reproducible and testable builds via Podman and `podman compose`
3. Make both services more infrastructure-agnostic

## Required Manual Interventions

1. Update submodules: `git submodule update --remote`
2. Upload ATLAS data gzip to host and unzip to `./new-atlas/server/`: `tar xzvf atlas-data.tar.gz -C new-atlas/server/`


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
