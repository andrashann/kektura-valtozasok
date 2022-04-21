# kektura-valtozasok

## What is this?

This is the source code of [https://kektura-valtozasok.onrender.com/](https://kektura-valtozasok.onrender.com/), a website that displays the route of the [National Blue Trail](https://en.wikipedia.org/wiki/National_Blue_Trail), a long-distance hiking trail in Hungary at various points in time. The route changes due to changes in ownership, forestry work, and other various reasons, but the main idea remains the same: walk one and a half million steps across Hungary to experience the country to the fullest.

This repository contains wthe website code (written in Vue/Nuxt); the data was gathered with the (not production quality) [iPython notebook found in this gist](https://gist.github.com/andrashann/7ee46cd169632a9fabe7b6197c0c830b), then manually uploaded to a Google Storage Bucket to be accessed from the website. This workflow could be improved and automatized.

## Build Setup

```bash
# install dependencies
$ npm install

# serve with hot reload at localhost:3000
$ npm run dev

# build for production and launch server
$ npm run build
$ npm run start

# generate static project
$ npm run generate
```
