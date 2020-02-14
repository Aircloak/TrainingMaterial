# Training material

Training material for use when training customer analysts and administrators.
This material is meant to be generic enough to work across customers.
The first version will undoubtedly contain Telefonica specific parts.
When the course has been held and we know what works and what not we
can replace these parts with more generic placeholders.

The goal is to have this repo contain versions which are not customer
specific and can therefore be used across customers.

For the time being the material outline is kept in [Analyst workshop](Analyst%20workshop.md)
and [Admin workshop](Admin%20workshop.md) respectively.

# Slides

## Dependencies

To make the slides you need node installed. The required version has been
specified in the [.tool-versions](.tool-versions) file and is managed
through the [asdf version manager](https://github.com/asdf-vm/asdf).
Please install `asdf` along with the [nodejs](https://github.com/asdf-vm/asdf-nodejs) and [yarn](https://github.com/twuni/asdf-yarn) plugins.
Then run `asdf install` at the root of the directory to get the necessary setup.

## Show slides

In the `slides-server` folder run:

- `yarn install` to install the dependencies
- `yarn start` to render the slides and start a webserver to show them
