# Jetpack CRM API Docs

## Development

Documentation and other files can be found on `source/`.

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
