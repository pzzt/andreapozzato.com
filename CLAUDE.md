# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal blog/zettelkasten built with Hugo static site generator using the PaperMod theme. The site serves as a collection of technical notes and articles primarily about system administration, Kubernetes, Linux, and Microsoft 365.

## Common Development Commands

### Building and Development
```bash
# Build the site in development mode (serves on http://localhost:1313)
hugo server

# Build the site in production mode (outputs to public/ directory)
hugo --minify

# Build including drafts
hugo --minify --buildDrafts

# Clean build directory
rm -rf public/
```

### Content Management
```bash
# Create new blog post using archetype
hugo new content/blog/post-title/index.md

# Create new content with default archetype
hugo new content/new-page.md

# Check site structure
hugo list all
```

### Theme Management
```bash
# Update PaperMod theme submodule
git submodule update --remote themes/PaperMod

# Initialize theme submodule if needed
git submodule update --init --recursive
```

## Architecture and Structure

### Content Organization
- **Blog posts**: `content/blog/` - Main articles, often as page bundles with images
- **Special pages**: `content/archive.md`, `content/search.md` - Custom page templates
- **Static assets**: `static/` - Images, favicon, and other static files
- **Theme customization**: `layouts/` - Override theme templates if needed

### Key Configuration Details
- **Base URL**: `https://www.andreapozzato.com/`
- **Theme**: PaperMod (Git submodule in `themes/PaperMod`)
- **Content structure**: Uses Hugo page bundles for posts with associated images
- **Features**: Search (Fuse.js), dark/light theme, RSS, breadcrumbs, social sharing

### Site Features
- **Home Info Mode**: Displays greeting and description on homepage
- **Search**: Integrated search using Fuse.js configuration in `hugo.yaml`
- **Analytics**: Google Analytics (G-SV1EDQJ9QK) enabled in production
- **Menu structure**: Tags, Archives, Search, and external link to brainb.it

### Content Front Matter Pattern
Posts use extensive front matter with PaperMod-specific options:
- Standard Hugo fields (title, date, tags)
- PaperMod features (showToc, ShowReadingTime, ShowWordCount, etc.)
- SEO fields (canonicalURL, description)
- Social sharing configuration

### Deployment Notes
- Site generates static HTML in `public/` directory
- Contains `.htaccess` for Apache configuration
- Uses environment detection (`env: production`) for proper analytics rendering
- Fingerprinting disabled for assets to simplify deployment