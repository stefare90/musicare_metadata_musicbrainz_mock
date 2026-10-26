# MusicBrainz (mock) — MusicAre metadata plugin

A **mock build** of the official MusicAre metadata plugin, published separately so the
MusicAre app can install **two plugins of type `metadata`** and exercise **plugin
switching**. It is not a fork: the behaviour is the official provider's, only the identity
differs. It is not meant for end users.

| | |
| :--- | :--- |
| id | `org.musicare.metadata.musicbrainz.mock` |
| version | `0.1.0` |
| pluginSdkVersion | `6.0.0` |
| type | `metadata` |

The artifact is **`plugin.zip` in the [v0.1.0 release](../../releases/tag/v0.1.0)**
(175,938 bytes, 40 files,
`sha256 0773d4415ea0424d7c9cca8859944f4879b6aef9cd7fa1ad90ee001f3db25d48`).

## Why this repository has no source

The app never reads the repository. Plugin discovery
(`music_are/lib/features/plugin/discovery/data/services/plugin_discovery_service.dart`)
searches GitHub for `topic:musicare-plugin`, takes each repository's **latest** release,
downloads the **first asset whose name ends in `.zip`** and reads **only `plugin.json`
from inside it**. A single `plugin.zip` asset per release is the whole contract, so this
repository only documents the mock; the code lives upstream.

## Identity diff

The entire difference from the official plugin, pinned at tag
[`v3.0.3`](https://github.com/stefare90/musicare_metadata_musicbrainz_plugin/tree/v3.0.3)
(commit `b397d72`). Save it as `identity.patch`: it applies cleanly with `git apply`.

```diff
--- a/plugin.json
+++ b/plugin.json
@@ -1,11 +1,11 @@
 {
-    "id": "org.musicare.metadata.musicbrainz",
-    "packageId": "musicare_metadata_musicbrainz_plugin",
+    "id": "org.musicare.metadata.musicbrainz.mock",
+    "packageId": "musicare_metadata_musicbrainz_mock",
     "type": "metadata",
-    "name": "MusicBrainz & ListenBrainz",
-    "version": "3.0.3",
+    "name": "MusicBrainz (mock)",
+    "version": "0.1.0",
     "pluginSdkVersion": "6.0.0",
     "author": "MusicAre Team",
-    "description": "Metadata, recommendations and user library provided by MusicBrainz, ListenBrainz, Wikidata and Wikimedia.",
-    "repository": "https://github.com/stefare90/musicare_metadata_musicbrainz_plugin"
+    "description": "Mock metadata provider used to test plugin switching in the MusicAre app.",
+    "repository": "https://github.com/stefare90/musicare_metadata_musicbrainz_mock"
 }
--- a/src/plugin.py
+++ b/src/plugin.py
@@ -32,9 +32,9 @@
 from .segments.track import MusicBrainzTrack
 from .segments.user import MusicBrainzUser
 
-PLUGIN_ID = "org.musicare.metadata.musicbrainz"
-PLUGIN_NAME = "MusicBrainz & ListenBrainz"
-PLUGIN_VERSION = "3.0.3"
+PLUGIN_ID = "org.musicare.metadata.musicbrainz.mock"
+PLUGIN_NAME = "MusicBrainz (mock)"
+PLUGIN_VERSION = "0.1.0"
 USER_AGENT = user_agent_for(PLUGIN_VERSION)
 
 
--- a/test/test_plugin_loader.py
+++ b/test/test_plugin_loader.py
@@ -15,8 +15,8 @@
     plugin = get_plugin()
 
     assert isinstance(plugin, BaseMetadataPlugin)
-    assert plugin.id == "org.musicare.metadata.musicbrainz"
-    assert plugin.name == "MusicBrainz & ListenBrainz"
+    assert plugin.id == "org.musicare.metadata.musicbrainz.mock"
+    assert plugin.name == "MusicBrainz (mock)"
     # The runtime reports ``plugin.version``, and the build validates ``plugin.json``:
     # they are two separate sources that must never drift (a release bumped only the
     # manifest and the runtime kept reporting the old version).
```

## Rebuild

```bash
git clone --depth 1 --branch v3.0.3 \
  https://github.com/stefare90/musicare_metadata_musicbrainz_plugin.git mock
cd mock
git apply ../identity.patch          # the diff above

python3 -m venv .venv
.venv/bin/pip install -r requirements-dev.txt
.venv/bin/python -m pytest test/ -q   # 201 passed, 12 deselected (live) — v0.1.0 build
.venv/bin/musicare-build              # plugin.zip: 175,938 bytes, 40 files

gh release upload v0.1.0 plugin.zip --clobber
```

`plugin.version` is reported by `src/plugin.py` while `plugin.json` is validated by the
builder: both must carry the same version (the upstream `test_plugin_loader.py` enforces
it), so bump them together.

## Notes

- **Why a separate id.** The app identifies an install by `manifest.id`, and the metadata
  runtime keys its per-plugin data directory — `auth.json` included — by `plugin.id`
  (`metadata_engine.py`). Keeping the official id would make the mock share the real
  plugin's login and report the wrong identity at runtime.
- **Behaviour is frozen on purpose.** A test fixture should not change under the tests;
  rebuild only when the real plugin's behaviour has to be re-verified, and bump the
  version if the identity patch changes.
