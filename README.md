# Project Title

Personal website and blog hosted on GitHub pages built using Jekyll, check it out at [marcnev.in](https://marcnev.in)

## Getting Started

Posts / Pages are written in Markdown, Rakefile used to gererate templates,

Test locally before pushing to repo for posting, to get started get everything setup


### Prerequisites

Software needed for running locally:
* Ruby
* Jekyll

### Local Setup

To run locally we'll use bundle, the watch flag allows us to make changes

```
bundle exec jekyll serve --watch
```

### Rakefile
Used to generate the drafts / layouts:

```
rake draft[title]   -- create a new draft post with "rake draft['draft title']"
rake list           -- list tasks
rake post[title]    -- create a new post with "rake post['post title']"
rake preview        -- preview the site with drafts
rake undraft[file]  -- publish a draft post with "rake undraft['draft-file.md']"
```

## Deployment

Deployment handled by GitHub pages, merge any changes to Master will trigger the environment build

## License

[MIT License](./LICENSE.txt)

## Acknowledgments

* Jekyll Template from [Mixyll](https://github.com/saikiransripada/mixyll)

