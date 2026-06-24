---
'@backstage/plugin-techdocs-react': patch
'@backstage/plugin-techdocs': patch
---

Fixed an issue where TechDocs would repeatedly request missing `techdocs_metadata.json` files, causing page flickering and excessive backend load. When metadata cannot be loaded, a warning is now shown instead of retrying indefinitely.
