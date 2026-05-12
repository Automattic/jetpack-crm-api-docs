# Jetpack CRM API Docs

## Development

The scaffolding for this project is forked from the now-abandonded Slate generator (https://github.com/slatedocs/slate).

Documentation can be found in `source/index.html.md`.

To get started, run the following:
```shell
bundle install
bundle exec middleman server
```

Docs will be visible here: `http://localhost:4567`

## Deployment

Run the following to build the documentation:
```shell
bundle exec middleman build
```

This will generate content in `build/`. You can then deploy to `gh-pages` with `./deploy.sh`.
