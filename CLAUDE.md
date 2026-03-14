# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Chinese translation of MIT's "The Missing Semester of Your CS Education" course. It's a Jekyll static site hosted on GitHub Pages at https://missing-semester-cn.github.io

## Development Commands

```bash
# Install dependencies
bundle install

# Build and serve locally with live reload
bundle exec jekyll serve -w

# Build for production
bundle exec jekyll build
```

## Architecture

### Jekyll Collections
Content is organized by year as Jekyll collections:
- `_2019/` - 2019 course lectures
- `_2020/` - 2020 course lectures
- `_2026/` - 2026 course lectures (new, being translated)

### Lecture File Format
Each lecture markdown file has frontmatter:
```yaml
---
layout: lecture
title: "课程标题"
date: YYYY-MM-DD
ready: true
sync: true          # whether synced with official English version
syncdate: YYYY-MM-DD
video:
  aspect: 56.25
  id: YouTube_ID
solution:
  ready: true
  url: solution-slug
---
```

### Layouts
- `default.html` - Base HTML template
- `lecture.html` - Lecture pages (extends default, includes video embed)
- `page.html` - Static pages (extends default)

### Includes
- `head.html` - HTML head section
- `nav.html` - Navigation bar
- `video.html`, `scaled_video.html`, `scaled_image.html` - Media embeds

## Translation Workflow

1. Check README.md for translation status and assignments
2. Create an issue using the translation template to claim a lecture
3. Translate the English content from https://missing.csail.mit.edu/
4. Submit a PR with the translated file
5. Update the status table in README.md

## Key Files

- `README.md` - Translation status tracking and contributor credits
- `_config.yml` - Jekyll configuration (collections, permalink structure)
- `.github/ISSUE_TEMPLATE/translation.md` - Issue template for claiming translations