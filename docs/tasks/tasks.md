---
title: Tasks
description: Deferred work in fig-css — one file each, every one with a done state
created: 2026-10-22
updated: 2026-10-26
contents:
  - "[A comment holding an opener cuts a value short](comment-holding-an-opener.md)"
  - "[Setting a repeated property edits the one that does not apply](repeated-property-set-at-the-first.md)"
---

# Tasks

Work this crate has committed to and has not done yet, one document each. A
bug is a task with a repro; anything else is a task with a done state written
down, so that finishing it is a fact rather than an opinion.

`contents` above lists every task, open and closed. Closing a task is an
edit, not a delete: its `status` becomes `done` or `dropped`, and it names
the commit or release that resolved it.

`status` takes `open`, `in-progress`, `done`, or `dropped`, and nothing else,
so that a tool can read it across every repository in the org.
