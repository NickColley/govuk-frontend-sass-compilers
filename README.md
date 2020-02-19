# Comparison of building GOV.UK Frontend with different Sass compilers


- [meld](https://meldmerge.org/) to check the different output.
- `cleancss` to remove formatting differences which makes diffing hard.

## Overall summary

GOV.UK Frontend requires a minimum of:

- Ruby sass version 3.4.x
- Node sass version 3.13.x (Libsass 3.3.x)
- Dart sass version 1.x

TODO talk about wrappers here.

## Ruby sass

Before each test setup with the following commands:
- `gem uninstall sass`
- `gem install sass -v <VERSION-NUMBER-HERE>`

Check it worked with:
- `sass --version`

Then build using:
- `sass ../govuk-frontend/package/govuk/all.scss > ruby-sass-<VERSION-NUMBER-HERE>.css`
- `node_modules/.bin/cleancss --format beautify ruby-sass-<VERSION-NUMBER-HERE>.css > ruby-sass-<VERSION-NUMBER-HERE>.min.css`

### v3.7.4 (Latest)

Verdict: succeeds

diffeerence between this and node-sass 3.7.4 (libsass), rounding of some values
- reording of seelectors (seen below too)

difference between this and dartsass

- dart sass adds `@charset "UTF-8";` at the top
- changes `\2014 ` to `—`

expands media queries differently:

dart-sass
```scss

@media (min-width:40.0625em) {
  .js-enabled .govuk-tabs__tab::after {
    content: "";
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0
  }
}
@media (min-width:40.0625em) {
  .js-enabled .govuk-tabs__panel {
    margin-bottom: 0;
    padding: 30px 20px;
    border: 1px solid #b1b4b6;
    border-top: 0
  }
}
@media (min-width:40.0625em) and (min-width:40.0625em) {
  .js-enabled .govuk-tabs__panel {
    margin-bottom: 0
  }
}
@media (min-width:40.0625em) {
  .js-enabled .govuk-tabs__panel > :last-child {
    margin-bottom: 0
  }
}
@media (min-width:40.0625em) {
  .js-enabled .govuk-tabs__panel--hidden {
    display: none
  }
}
```

ruby sass:
```scss

@media (min-width:40.0625em) {
  .js-enabled .govuk-tabs__tab::after {
    content: "";
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0
  }
  .js-enabled .govuk-tabs__panel {
    margin-bottom: 0;
    padding: 30px 20px;
    border: 1px solid #b1b4b6;
    border-top: 0
  }
}
@media (min-width:40.0625em) and (min-width:40.0625em) {
  .js-enabled .govuk-tabs__panel {
    margin-bottom: 0
  }
}
@media (min-width:40.0625em) {
  .js-enabled .govuk-tabs__panel > :last-child {
    margin-bottom: 0
  }
  .js-enabled .govuk-tabs__panel--hidden {
    display: none
  }
}
```

### 3.5.0
verdict: succeeds

identical to 3.7.4

### 3.4.0
verdict: succeeds

rounding differences in line-height values
e.g.     line-height: 1.3157894737 (3.7.4) vs line-height: 1.31579 ( 3.4.0)

colour changes
e.g.  .govuk-button:hover {
  background-color: #005a30
} (3.7.4)

vs   background-color: #005930 (3.4.0)

likely a difference in how the 'mix' function works.

### 3.3.0
verdict: fails

```log
Syntax error: Functions can only contain variable declarations and control directives.
        on line 42 of ../govuk-frontend/package/govuk/helpers/_colour.scss
        from line 5 of ../govuk-frontend/package/govuk/settings/_colours-applied.scss
        from line 14 of ../govuk-frontend/package/govuk/settings/_all.scss
        from line 1 of ../govuk-frontend/package/govuk/all.scss
```

### 3.2.0
verdict: fails

```log
Syntax error: Invalid CSS after "...rontend_toolkit": expected ")", was ": $govuk-compat..."
        on line 72 of ../govuk-frontend/package/govuk/settings/_compatibility.scss
        from line 6 of ../govuk-frontend/package/govuk/settings/_all.scss
        from line 1 of ../govuk-frontend/package/govuk/all.scss
```

## Wrappers
### Node sass

Unless specified I ran these with Node.js v12.13.1 (npm v6.12.1). (`nvm use v12.13.1`).

Before each test setup with the following commands:
- `npm uninstall node-sass`
- `rm -rf node_modules`
-  `npm install node-sass@<VERSION-NUMBER-HERE> --save-dev`

Check it worked with:
- `node_modules/.bin/node-sass --version`

Then build using:
- `node_modules/.bin/node-sass ../govuk-frontend/package/govuk/all.scss > node-sass/<VERSION-NUMBER-HERE>.css`
- `node_modules/.bin/cleancss --format beautify node-sass/<VERSION-NUMBER-HERE>.css > node-sass/<VERSION-NUMBER-HERE>.min.css`

#### 4.13.1 - latest
LibSass: 3.5.4

verdict: succeeds

### 4.3.0
LibSass: 3.4.3

Requires Node.js version 8:
`nvm install v8.17.0`
`node -v`

verdict: succeeds

same apart from this:
```scss
@media print {
  .govuk-link[href^="/"]::after,
  .govuk-link[href^="http://"]::after,
  .govuk-link[href^="https://"]::after {
    content: " (" attr(href) ")";
    font-size: 90%;
    word-wrap: break-word
  }
}
```

4.3.0
```scss

@media print {
  [href^="/"].govuk-link::after,
  [href^="http://"].govuk-link::after,
  [href^="https://"].govuk-link::after {
    content: " (" attr(href) ")";
    font-size: 90%;
    word-wrap: break-word
  }
}
```

### 3.13.1
LibSass: 3.3.6

Requires Node.js version 8:
`nvm install v8.17.0`
`node -v`

verdict: same as 4.3.0

### 3.3.3
LibSass: 3.2.5

Requires Node.js version 5:
`nvm install v4.0.0`
`node -v`

verdict: fails

```log
{
  "message": "Unknown colour `blue`",
  "column": 5,
  "line": 42,
  "file": "/Users/nickcolley/Development/govuk-frontend/package/govuk/helpers/_colour.scss",
  "status": 1
}
```

## Dart sass 1.25.0 - latest

Requires Node.js version 5:
`nvm install v12.13.1`
`node -v`

Build with:
- `node_modules/.bin/sass ../govuk-frontend/package/govuk/all.scss > dart-sass/1-25-0.css`
- `node_modules/.bin/cleancss --format beautify dart-sass/1-25-0.css > dart-sass/1-25-0.min.css`
