# Moment Duration Format

**Format plugin for the Moment Duration object.**

This is a plugin to the Moment.js JavaScript date library to add comprehensive formatting to Moment Durations.

Format template grammar is patterned on the existing Moment Date format template grammar, with a few modifications because durations are fundamentally different from dates.

This plugin does not have any dependencies beyond Moment.js itself, and may be used in the browser and in Node.js.

---

## Installation

**Node.js**

`npm install moment-duration-format`

**Bower**

`bower install moment-duration-format`

**Browser**

`<script src="path/to/moment-duration-format.js"></script>`

When using this plugin in the browser, be sure to include moment.js on your page first.

---

## Usage

### Module

To use this plugin as a module, use the `require` function:
```
require("moment-duration-format");
```

The plugin returns the init function so that duration format can be initialized on other moment instances.

The plugin depends on moment.js, which is not specified as a package dependency in the currently published version.

### Basics

The duration format method can format any moment duration. If no template or other arguments are provided, the default template function will generate a template string based on the duration's value.

```
moment.duration(123, "minutes").format();
// "2:03:00"

moment.duration(123, "months").format();
// "10y 3m"
```

The duration format method may be called with three optional arguments:
```
moment.duration.format([template] [, precision] [, settings])
```

### Template

`template` (string|function) is the string used to create the formatted output, or a function that returns the string to be used as the format template.

```
moment.duration(123, "minutes").format("h:mm");
// "2:03"
```

The template string is parsed for moment token characters, which are replaced with the duration's value for each unit type. The moment tokens are:
```
years:   Y or y
months:  M
weeks:   W or w
days:    D or d
hours:   H or h
minutes: m
seconds: s
ms:      S
```

Escape token characters within the template string using square brackets.
```
moment.duration(123, "minutes").format("h [hrs], m [min]");
// "2 hrs, 3 min"
```

#### Token Length

For some time duration formats, a zero-padded value is required. Use multiple token  characters together to create the correct amount of padding.

```
moment.duration(3661, "seconds").format("h:mm:ss");
// "1:01:01"

moment.duration(15, "seconds").format("ssss [s]");
// "0015 s"
```

When the format template is trimmed, token length on the largest-magnitude rendered token can be trimmed as well.

```
moment.duration(123, "seconds").format("h:mm:ss");
// "2:03"
```

See sections *trim* and *forceLength* below for more details.

### Precision

`precision` (number) defines the number of decimal fraction or integer digits to display for the final value.

The default precison value is `0`.
```
moment.duration(123, "minutes").format("h [hrs]");
// "2 hrs"
```

Positive precision defines the number of decimal fraction digits to display.
```
moment.duration(123, "minutes").format("h [hrs]", 2);
// "2.04 hrs"
```

Negative precision defines the number of integer digits to truncate to zero.
```
moment.duration(223, "minutes").format("m [min]", -2);
// "200 min"
```

### Settings

`settings` is an object that can override any of the default moment duration format options.

Both the `template` and `precision` arguments may be specified as properties of a single `settings` object argument, or they may be passed separately along with an optional settings object.

```
moment.duration(123, "minutes").format({ template: "h [hrs]", precision: 2 });
// "2.04 hrs"
```

#### trim

The default `trim` value is `"largest"`.

Largest-magnitude tokens are automatically trimmed when they have no value.
```
moment.duration(123, "minutes").format("d[d] h:mm:ss");
// "2:03:00"
```

Trimming also functions when the format string is oriented with token magnitude increasing from left to right.
```
moment.duration(123, "minutes").format("s [seconds], m [minutes], h [hours], d [days]");
// "0 seconds, 3 minutes, 2 hours"
```

To stop trimming altogether, set `{ trim: false }`.
```
moment.duration(123, "minutes").format("d[d] h:mm:ss", { trim: false });
// "0d 2:03:00"
```

`trim` can be a string, a delimited list of strings, an array of strings, or a boolean. Accepted values are as follows:
- `"large"` Trim largest-magnitude zero-value tokens until finding a token with a value, a token identified as `stopTrim`, or the final token of the format string.
- `"small"` Trim smallest-magnitude zero-value tokens until finding a token with a value, a token identified as `stopTrim`, or the final token of the format string.
- `"both"` Execute `"large"` trim then `"small"` trim.
- `"mid"` Trim any zero-value tokens that are not the first or last tokens. Usually used in conjunction with `"large"` or `"both"`. e.g. `"large mid"` or `"both mid"`.
- `"final"` Trim the final token if it is zero-value. Use this option with `"large"` or `"both"` to output an empty string when formatting a zero-value duration. e.g. `"large final"` or `"both final"`.
- `"all"` Trim all zero-value tokens. Shorthand for `"both mid final"`.
- `"left"` Maps to `"large"` to support this plugin's version 1 API.
- `"right"` Maps to `"large"` to support this plugin's version 1 API.
- `true` Maps to `"large"`.
- `false` Disables trimming.

#### stopTrim

Trimming will stop when a token listed in this option is reached.

Option value may be a moment token string, a delimited set of moment token strings, or an array of moment token strings. Alternatively, set `stopTrim` on tokens in the format template string directly using a "*" character before the moment token.

```
moment.duration(23, "minutes").format("d[d] h:mm:ss", { stopTrim: "h" });
// "0:03:00"

moment.duration(23, "minutes").format("d[d] *h:mm:ss");
// "0:03:00"
```

This option affects all trimming modes: `"large"`, `"small"`, `"mid"`, and `"final"`.

```
moment.duration(2, "hours").format("y [years], d [days], h [hours], m [minutes], s [seconds]", { trim: "both", stopTrim: "d m" });
// "0 days, 2 hours, 0 minutes"

moment.duration(2, "hours").format("y [years], *d [days], h [hours], *m [minutes], s [seconds]", { trim: "both" });
// "0 days, 2 hours, 0 minutes"
```

#### largest

Set `largest` to a positive integer to output only the `n` largest-magnitude moment tokens that have a value. All lesser-magnitude or zero-value moment tokens will be ignored. This option effectively runs `trim: "all"` before selecting the largest-magnitude tokens, and takes effect even when `trim: false` is used.

```
moment.duration(7322, "seconds").format("d [days], h [hours], m [minutes], s [seconds]", { largest: 2 });
// "2 hours, 2 minutes"

moment.duration(1216922, "seconds").format("y [years], w [weeks], d [days], h [hours], m [minutes], s [seconds]", { largest: 2 });
// "2 weeks, 2 hours"
```

#### trunc

Default behavior rounds the final token value.

```
moment.duration(179, "seconds").format("m [minutes]");
// "3 minutes"

moment.duration(3780, "seconds").format("h [hours]", 1);
// "1.1 hours"
```

Set `trunc` to `true` to truncate final token value. This was the default behavior in version 1 of this plugin.

```
moment.duration(179, "seconds").format("m [minutes]", { trunc: true });
// "2 minutes"

moment.duration(3780, "seconds").format("h [hours]", 1, { trunc: true });
// "1.0 hours"
```

#### forceLength

Force the first moment token with a value to render at full length, even when the template is trimmed and the first moment token has a length of 1. Sounds more complicated than it is.

```
moment.duration(123, "seconds").format("h:mm:ss");
// "2:03"
```

If you want minutes to always be rendered with two digits, you can use a first token with a length greater than 1 (this stops the automatic token length trimming for the first token that has a value).

```
moment.duration(123, "seconds").format("hh:mm:ss");
// "02:03"
```

Or you can use `{ forceLength: true }`.

```
moment.duration(123, "seconds").format("h:mm:ss", { forceLength: true });
// "02:03"
```

#### useSignificantDigits

When `useSignificantDigits` is set to `true`, the `precision` option determines the maximum significant digits to be rendered. Precision must be a positive integer. Significant digits extend across unit types, e.g. `"6 hours 37.5 minutes"` represents `4` significant digits. Enabling this option causes token length to be ignored. See  https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toLocaleString.

### Localization

Formatted numerical output is rendered using `toLocaleString`. See https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toLocaleString.

Unit names are detected using the locale set in moment.js, which can be different from the locale of user's environment. See https://momentjs.com/docs/#/i18n/.

The options below clearly do not yet address all i18n requirements for duration formatting (such as languages with [multiple forms of plural](https://developer.mozilla.org/en-US/docs/Mozilla/Localization/Localization_and_Plurals)), but they are a significant step in the right direction.

#### userLocale

Numerical output is rendered using the locale of the user's environment. Set the `userLocale` option to render numerical output using a different locale.

```
moment.duration(1234567, "seconds").format("m [minutes]", 3);
// "20,576.117 minutes"

moment.duration(1234567, "seconds").format("m [minutes]", 3, { userLocale: "de-DE" });
// "20.576,117 minutes"
```

#### Extending Moment's `locale` object

This plugin now extends moment.js's `locale` object with `durations` and `durationsShort` values. The `en` locale is included with this plugin. Other locales may be easily defined to provide auto-singularized and auto-localized unit labels in different languages. If the plugin cannot find the duration locale extensions for the active moment locale, those plugin features will not be active.

Below is the `en` locale extension.

```
context.updateLocale('en', {
    durations: {
        S: 'millisecond',
        SS: 'milliseconds',
        s: 'second',
        ss: 'seconds',
        m: 'minute',
        mm: 'minutes',
        h: 'hour',
        hh: 'hours',
        d: 'day',
        dd: 'days',
        w: 'week',
        ww: 'weeks',
        M: 'month',
        MM: 'months',
        y: 'year',
        yy: 'years'
    },
    durationsShort: {
        S: 'msec',
        SS: 'msecs',
        s: 'sec',
        ss: 'secs',
        m: 'min',
        mm: 'mins',
        h: 'hr',
        hh: 'hrs',
        d: 'dy',
        dd: 'dys',
        w: 'wk',
        ww: 'wks',
        M: 'mo',
        MM: 'mos',
        y: 'yr',
        yy: 'yrs'
    }
});
```

#### Auto-localized Unit Labels

Once Moment's `locale` object is extended with `durations` and `durationsShort` values, the `_` character can be used to generate auto-localized unit labels in the formatted output.

A single underscore `_` will be replaced with the short duration unit label for its associated moment token.

A double underscore `__` will be replaced with the duration unit label for its associated moment token.

Auto-localized unit labels are singularized just as if they had appeared directly in the format template string.

```
moment.duration(1, "minutes").format("m _");
// "1 min"

moment.duration(1, "minutes").format("m __");
// "1 minute"
```

#### useSingular

The default behaviour automatically singularizes unit labels when they appear in the text associated with each moment token. The plural form of the unit name must appear in the format template. Long and short unit labels are singularized, based on the locale defined in moment.js.

```
moment.duration(1, "minutes").format("m [minutes]");
// "1 minute"

moment.duration(1, "minutes").format("m [mins]");
// "1 min"
```

Set `useSingular` to `false` to disable auto-singularizing.

```
moment.duration(1, "minutes").format("m [minutes]", { useSingular: false });
// "1 minutes"

moment.duration(1, "minutes").format("m [mins]", { useSingular: false });
// "1 mins"
```

Singularization is not used when a value is rendered with decimal precision.

```
moment.duration(1, "minutes").format("m [minutes]", 2);
// "1.00 minutes"
```

#### useLeftUnits

The text to the right of each moment token in a format string is treated as that token's units for the purposes of trimming, singularizing, and auto-localizing. To properly singularize or localize a format string where the token/unit association is reversed, set `useLeftUnits` to `true`.

```
moment.duration(7322, "seconds").format("_ h, _ m, _ s", { useLeftUnits: true });
// "hrs 2, mins 2, secs 2"
```

#### useGrouping

Formatted numerical output is rendered using `toLocaleString` with the option `useGrouping` enabled. Set `useGrouping` to `false` to disable digit grouping.

```
moment.duration(1234, "seconds").format("s [seconds]");
// "1,234 seconds"

moment.duration(1234, "seconds").format("s [seconds]", { useGrouping: false });
// "1234 seconds"
```

#### Decimal Separator

Previous versions of the plugin used a `decimalSeparator` option. That option is no longer used and will have no effect. Decimal separators are rendered using `toLocalString`.

```
moment.duration(1234567, "seconds").format("m [minutes]", 3, { userLocale: "de-DE" });
// "20.576,117 minutes"
```
