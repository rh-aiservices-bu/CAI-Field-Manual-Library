---
title: CAI Field Manual
description: A working skeleton for the field-manual example site.
# Every branch of the tree renders expanded on every page: both sidebar tree
# walkers honor sidebar_expanded as a per-section force-expand, and Home's
# cascade sets it for the whole site. Without it the walkers render each
# page's tree with only the active path open, so navigating to Chat folds
# Field Manual. Manual chevron collapse still works within a visit (fold
# state is in-memory by theme design and resets on navigation).
#
# Second cascade entry: every top-level section (kind: section, path: /*,
# i.e. one path segment below Home) becomes its own sidebar root. This is
# what segregates each manual's sidebar from the others ("book" behavior) --
# scoped here, from the aggregator's own Home page, specifically so manual
# repos never need `sidebar_root_for: self` added to their own content.
# _target's path glob is one level deep on purpose: a manual with its own
# nested sections (deeper _index.md files) must not have those fragment
# into separate roots too.
cascade:
  - sidebar_expanded: true
  - target:
      kind: section
      path: /*
    sidebar_root_for: self
---
